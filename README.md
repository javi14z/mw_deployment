# Deployment of setup for Test Case 3.5 (Traffic Generation)

## Description

Using **Ansible** we have automated the deployment of the different Docker images (described in test case 3.5 [https://github.com/javi14z/mw_cgclient]) inside the **Kne** host server (192.168.159.53) of the laboratory. Ansible is already installed inside the server, although the installation method is:

```
$ sudo apt install ansible
```

Throughout the document, we will describe the way of using the different playbooks created. This will allow us to deploy the scenario (__mw-deployment.yaml__),configure the pods(__mw-config.yaml__), delete them (__mw-undeployment.yaml__) and execute tasks (__mw-tasks.yaml__). With this, we can easily **scale** the number of pods (__clients_number.yaml__) that we are going to deploy for different experiments. We can also decide the programs and scripts that we want to run as well as the execution time and other options.

## Scenario

![scenario](https://github.com/javi14z/mw_k8s/blob/main/images/Traffic_Generation.png)

## Usage

### - clients_number.yaml
The first thing to do is adjust in this file the number of clients that we want to deploy for traffic generation (**ddosclient** and **cgclient**). This file will be used by the different playbooks to display the exact number of clients:

We can deploy two types of clients depending on the traffic we want to generate in the experiment:
- Simulation of Elephant and Cheetah Network Flows (**cg_count**)
- Simulation of a Variety of DDoS Attacks (**ddos_count**)

    ![image1](https://github.com/javi14z/mw_k8s/blob/main/images/image1.png)

**Note** : Currently the topology configuration has support for 10 clients of each type.

### - mw-deployment.yaml
```
$ ansible-playbook mw-deployment.yaml
```

Inside the playbook we run the configuration file that Kne uses to create the topology through which the traffic will pass. This brings up all the routers following the topology built in EVE-NG.

**Note** : In this file you must configure the path where the kne topology file is (kne create <path_to_kne_file>). Currently the server is configured with the following path: ~/kne/examples/cisco/conversion/Topologias/ddos/TopologiaDdos.yaml

### - mw-config.yaml
```
$ ansible-playbook mw-config.yaml
```

With this playbook we configure the different pods according to the number requested in the __clients_number.yaml__ file.

Inside the file also we configure the different routes of the **topology** through which all the traffic generated will pass as well as the different environment variables for the operation of the programs. We also start a capture with tcpdump in each pod, in which all the traffic will be saved.

**Note** : When the deployment playbook is finished, all the scenario is configured and ready to play tasks. If we want to collect .pcaps inside the pods, we can activate them in the last section of the playbook called tcpdump.

For more debug options in the deploymets with Ansible add: "<set_command> 2>&1 | tee /dev/tty"


### - mw-undeploy.yaml
```
$ ansible-playbook mw-undeploy.yaml
```

When we run this playbook, we will store all the **logs** generated by the different programs within our host machine. Once the information is stored, the entire deployment will be completely deleted.

**Note** : The captures previously started with tcpdump of each pod will be saved the path ~/NDT. This path is a mount point that is connected to an NFS (Network File System) server with the IP address 192.168.159.137 and the remote path /mnt/nfs/NDT. We use the flag -s with tcpdump to limit the snapshot length.

### - mw-tasks.yaml
```
$ ansible-playbook mw-tasks.yaml
```

In this file we have to declare the differents programms that we are going to execute in the pods generated. Inside the file you can comment with **#** the tasks that you don't want to be executed. You can also configure the execution time of each task and other options:

![image2](https://github.com/javi14z/mw_k8s/blob/main/images/image2.png)

#### The programs and scripts that have been implemented are the following:
##### **Cgserver:**
- ACROSSserver.py:
    ```
    kubectl exec -n ddos cgserver -- python3 ACROSSserver.py 
    ```
    We use the sockets library in Python, which offers low-level access to network interfaces. Using this library, we have developed a straightforward server that listens for bursts of traffic from clients.

##### **Ddosserver:**
- ngnix:
    ```
    kubectl exec -n ddos ddosserver -- nginx -g 'daemon off;' 
    ```
    We have set up a standard HTTP/2 server using NGINX, with the aim of serving a basic 'Hello World' web page at the root URI. This URI will be used by clients to carry out the HTTP flood attack

- bind9:
    ```
    kubectl exec -n ddos ddosserver -- service bind9 start
    ```
    We have setup a dns server using bind9. The server is waiting to receive new queries through port 53.
    - Features: EDNS,DNSSEC
    - Next features: DOH support

##### **Cgclient:**
- ACROSSfile_transfer.py:
    ```
    kubectl exec -n ddos {{ item.pod }} -- env https_proxy="http://{{ internet_ip }}:3128" https_proxy="http://{{ internet_ip }}:3128" python3 ACROSSfile_transfer.py <large_file_url> 
    ```
    To simulate elephant flows, we have used WGET to generate file transfers. With this, we begin to download a large video file from a website.
    - Parameters:
        
        -<large_file_url>: set the url to dowload the large file

- ACROSSconsuming_video.py:
    ```
    kubectl exec -n ddos {{ item.pod }} -- env https_proxy="http://{{ internet_ip }}:3128" https_proxy="http://{{ internet_ip }}:3128" python3 ACROSSconsuming_video.py <YT_video_URL> <viewing_time>
    ```
    This Python script automates the process of opening a YouTube video and simulates the user watching the video. It is designed to simulate elephant flows having a large number of users consuming a viral video on YouTube. To automate web browsing we use the Selenium library.
    - Parameters:

        -<YT_video_URL>: set the url to a Youtube Video

        -<viewing_time>: set the viewing time video
        
- ACROSSshorts.py:
    ```
    kubectl exec -n ddos {{ item.pod }} -- env https_proxy="http://{{ internet_ip }}:3128" https_proxy="http://{{ internet_ip }}:3128" python3 ACROSSshorts.py <viewing_time>
    ```
    This Python script automates the process of opening YouTube Shorts and simulating the user scrolling through the Shorts feed at a random pace. It is designed to simulate cheetah flow, where a user scrolls through YouTube Shorts and videos load lazily as they show up in the browser viewport. We also use  the Selenium libraryto automate the procces.
    - Parameters:

        -<viewing_time>: set the viewing time of Youtube shorts

- ACROSSclient.py:
    ```
    kubectl exec -n ddos {{ item.pod }} -- env cgserver={{ cgserver_ip }} python3 ACROSSclient.py <set_multiplier>
    ```
    This Python script generates short-lived bursts of high traffic, creating 'cheetah flows'. To simulate this behavior, random spikes of data are sent to the server during communication. This can be achieved by sending a sudden surge of packets over a short duration.
    - Parameters:

        -<set_multiplier>: adjust the multiplier as needed.

- CGTest19.sh:
    ```
    kubectl exec -n ddos {{ item.pod }} -- env https_proxy="http://{{ internet_ip }}:3128" https_proxy="http://{{ internet_ip }}:3128" ./CGTest19.sh web_links.txt
    ```
    This script generates traffic visiting websites with curl from a text file with links.


##### **Ddosclient:**
- hping3.sh:
    ```
    kubectl exec -n ddos {{ item.pod }} -- env ddosserver={{ ddosserver_ip }} sudo hping3 <select_mode> -c <number_of_packets> -d <packet_data_size> -S <target_ip> -w <window size> -p <select_port> --flood
    ```
    This script initiates a DDoS attack on the server using hping3 with specified parameters, simulating a IP/TCP/UDP flood of traffic.
    - Parameters:
        
        -<select_mode>: TCP -> leave blank, IP -> -0, ICMP -> -1, UDP -> -2 

        -<number_of_packets>: set the packet number to send
        
        -<packet_data_size>: set the packet data size
        
        -<target_ip>: target ip to flood
        
        -<window_size>: MTU size
        
        -<select_port>: set the port to flood

- vegeta.sh:
    ```
    kubectl exec -n ddos {{ item.pod }} -- env ddosserver={{ ddosserver_ip }} ./vegeta.sh <set_time>
    ```
    This script initiates a DDoS attack on the server using vegeta (https://github.com/tsenart/vegeta), simulating a HTTP flood of traffic.
    - Parameters:
        
        -<set_time>: set the time of the flood.

- Quick_scapy (scapy-demo.py):
    This script initiates a DDoS attackk on the server using the scapy library, simulating source IP Spoofing 

    **Note**: Is not yet implemented

- dns_scapy.py:
    ```
    kubectl exec -n ddos {{ item.pod }} -- env https_proxy="http://{{ internet_ip }}:3128" https_proxy="http://{{ internet_ip }}:3128" ddosserver={{ ddosserver_ip }} python3 dns_scapy.py <dns_query_packets>
    ```
    This script initiates a DNS Amplification attack on the local Dns server using the scapy library.
    - Parameters:
        
        -<dns_query_packets>: set the number of querys.

- dns_proxy (for doh):
    This Python script initiates a DDoS attack on the server using the scapy library, simulating DNS Amplification.

    **Note**: The DNS server still doesn't support doh

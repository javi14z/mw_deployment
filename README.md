# Deployment of setup for Test Case 3.5 (Traffic Generation)

## Description

Using **Ansible** we have automated the deployment of the different Docker images (described in test case 3.5) inside the **Kne** host server (192.168.159.53) of the laboratory. Ansible is already installed inside the server, although the installation method is:

```
$ sudo apt install ansible
```

Throughout the document, we will describe the way of using the different playbooks created. This will allow us to deploy the scenario (__mw-deployment.yaml__), delete them (__mw-undeployment.yaml__) and execute tasks (__mw-tasks.yaml__). With this, we can easily **scale** the number of pods that we are going to deploy for different experiments. We can also decide the programs and scripts that we want to run as well as the execution time and other options.

## Scenario

![scenario](https://github.com/javi14z/mw_k8s/blob/main/images/Traffic_Generation.png)

## Usage

### - clients_number.yaml
The first thing to do is adjust in this file the number of clients that we want to deploy for traffic generation (**ddosclient** and **cgclient**). This file will be used by the different playbooks to display the exact number of clients:

We can deploy two types of clients depending on the traffic we want to generate in the experiment:
- Simulation of Elephant and Cheetah Network Flows (**cg_count**)
- Simulation of a Variety of DDoS Attacks (**ddos_count**)

    ![image1](https://github.com/javi14z/mw_k8s/blob/main/images/image1.png)

### - mw-deployment.yaml
```
$ ansible-playbook mw-deployment.yaml
```

With this playbook we configure and launch the different pods according to the number requested in the __clients_number.yaml__ file.

Inside the file also we configure the different routes of the **topology** through which all the traffic generated will pass as well as the different environment variables for the operation of the programs.

**Note** : Inside the playbook we run the following command:
```
$ kne create ~/kne/examples/cisco/conversion/Topologias/ddos/TopologiaDdos.yaml
```

This is the configuration file that Kne uses to create the topology through which the traffic will pass. Currently the topology configuration has support for 10 clients of each type.


### - mw-undeploy.yaml
```
$ ansible-playbook mw-undeploy.yaml
```

When we run this playbook, we will store all the **logs** generated by the different programs within our host machine. Once the information is stored, the entire deployment will be completely deleted.

**Note** : The collection of logs before their deletion is not yet implemented

### - mw-tasks.yaml
```
$ ansible-playbook mw-tasks.yaml
```

In this file we have to declare the differents programms that we are going to execute in the pods generated. Inside the file you can comment with **#** the tasks that you don't want to be executed. You can also configure the execution time of each task and other options:

![image2](https://github.com/javi14z/mw_k8s/blob/main/images/image2.png)

**Note** : The configuration of the topology routers takes some time to be operational. If the playbook runs too soon, the traffic of some tasks may not pass through the topology correctly.

#### The programs and scripts that have been implemented are the following:
##### **Cgserver:**
- ACROSSserver.py: 
    We use the sockets library in Python, which offers low-level access to network interfaces. Using this library, we have developed a straightforward server that listens for bursts of traffic from clients.

##### **Ddosserver:**
- ngnix:
    We have set up a standard HTTP/2 server using NGINX, with the aim of serving a basic 'Hello World' web page at the root URI. This URI will be used by clients to carry out the HTTP flood attack

- bind9:
    We have setup a dns server using bind9. The server is waiting to receive new queries through port 53.
    - Features implemented:
          EDNS
          DNSSEC
    - Next features:
          DOH support


##### **Cglient:**
- ACROSSfile_transfer.py:
    To simulate elephant flows, we have used WGET to generate file transfers. With this, we begin to download a large video file from a website.

- ACROSSconsuming_video.py:
    This Python script automates the process of opening a YouTube video and simulates the user watching the video. It is designed to simulate elephant flows having a large number of users consuming a viral video on YouTube. To automate web browsing we use the Selenium library.

- ACROSSshorts.py:
    This Python script automates the process of opening YouTube Shorts and simulating the user scrolling through the Shorts feed at a random pace. It is designed to simulate cheetah flow, where a user scrolls through YouTube Shorts and videos load lazily as they show up in the browser viewport. We also use  the Selenium libraryto automate the procces.

- ACROSSclient.py:
    This Python script generates short-lived bursts of high traffic, creating 'cheetah flows'. To simulate this behavior, random spikes of data are sent to the server during communication. This can be achieved by sending a sudden surge of packets over a short duration.

##### **Ddosclient:**
- hping3.sh:
    This script initiates a DDoS attack on the server using hping3 with specified parameters, simulating a IP/TCP/UDP flood of traffic.

- vegeta.sh:
    This script initiates a DDoS attack on the server using vegeta (https://github.com/tsenart/vegeta), simulating a HTTP flood of traffic.

- Quick_scapy:
    This script initiates a DDoS attackk on the server using the scapy library, simulating source IP Spoofing 
    **Note**: Is not yet implemented

- dns_proxy:
    This Python script initiates a DDoS attack on the server using the scapy library, simulating DNS Amplification.

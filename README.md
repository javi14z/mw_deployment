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
- Simulation of Elephant and Cheetah Network Flows (cg_count)
- Simulation of a Variety of DDoS Attacks (ddos_count)

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

When we run this playbook, we will save all the **logs** generated by the different programs within our host machine. Once the information is saved, the entire deployment will be completely deleted.

**Note** : The collection of logs before their deletion is not yet implemented

### - mw-tasks.yaml
```
$ ansible-playbook mw-tasks.yaml
```

In this file we have to declare the differents programms that we are going to execute in the pods generated. Inside the file you can comment with **#** the tasks that you don't want to be executed. You can also configure the execution time of each task and other options:

![image2](https://github.com/javi14z/mw_k8s/blob/main/images/image2.png)

**Note** : The configuration of the topology routers takes some time to be operational. If the playbook runs too soon, the traffic of some tasks may not pass through the topology correctly.

#### The programs and scripts that have been implemented are the following:
- ACROSSfile_transfer.py:
- ACROSSconsuming_video.py:











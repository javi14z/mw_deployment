# Deployment of setup for Test Case 3.5 (Traffic Generation)

## **ANSIBLE and KNE**

By using Ansible we have automated the deployment of the different Docker images within the Kne host server (192.168.159.53). Inside the server Ansible is 
already installed, although the installation method is:

```
$ sudo apt install ansible
```

This allows us to decide the number of pods that we are going to deploy as well as the programs and scripts that we want to be launched.

## **Usage**

Create the pods with the topology configuration:
```
$ ansible-playbook mw-deployment.yaml
```
Execute the differents tasks in the pods:
```
$ ansible-playbook mw-tasks.yaml
```
Delete the topology and save the logs:
```
$ ansible-playbook mw-undeploy.yaml
```

### clients_number.yaml

In this file we decide the number of clients for traffic generation (ddosclient and cgclient).

-Simulation of Elephant and Cheetah Network Flows (cglient)
-Simulation of a Variety of DDoS Attacks (ddosclient)

![image1](https://github.com/javi14z/mw_k8s/blob/main/images/image1.png)

### mw-deployment.yaml

In this file we configure and launch the different pods according to the number requested in the clients_number.yaml file.

Inside we configure the different routes of the topology through which all the traffic generated will pass as well as the different environment variables for 
the operation of the programs.

### mw-tasks.yaml

In this file we have to declare the differents programms that we are going to execute in the pods generated. Inside the file you can comment with # the tasks 
that you don't want to be executed. You can also configure the execution time of each task and other options:

![image2](https://github.com/javi14z/mw_k8s/blob/main/images/image2.png)

### mw-undeploy.yaml

When we run this playbook, first of all we are going to save all the logs generated by the different programs. Once done, the entire deployment will be deleted 
completely.
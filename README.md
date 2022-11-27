# Pi Kube Cluster Project
- Build a Kubernetes cluster using 5 Raspberry Pi's as worker nodes and 3 Ubuntu's VMs as master nodes (controllers).
- Use wireless connection via private router.
- Build and deploy an application on the cluster which contain 5 different containers providing different services. Every containers are dependent on each other which require container networking.
- Use K3s, a lightweight Kubernetes distribution created by Rancher Labs.

# Preparation
- 5 x Raspberry Pi 3 Model B+ with microSD cards
- 3 x Ubuntu VMs (using Virtual box)
- Ethernet/WiFi Router (using TP-Link TL WR841N)

# How to set up Kubernetes Cluster
## Setting up each VMs as K3s Master nodes
- Set network adapter to bridged networking to make VMs accessible from the network and attain IP address of the VMs.
1. Update system
```
sudo apt update
sudo apt upgrade -y
```
2. Allow ports on firewall that will be used to communicate between the master and the worker nodes (6443 - https port).
```
sudo ufw allow 6443/tcp
```
3. Install K3s from https://get.k3s.io which contain shell script for installing K3s
```
curl -sfL https://get.k3s.io | INSTALL_K3S_SKIP_ENABLE=true sh -
```
4. Launch a server node with the cluster-init flag to enable clustering and a token that will be used as a shared secret to join nodes to the cluster
```
sudo k3s server --cluster-init --token=SECRET
```
5. Join the second and third servers to the cluster using the shared secret token

```
sudo k3s server --server https://<master_IP>:6443 --token=SECRET
```
6. Marks the master nodes as unschedulable using Kubernetes Cordon.
```
sudo kubectl cordon <node_name>
```

## Setting up each Pis as K3s Worker nodes
### Writing microSD card
- Download and install [Raspberry Pi Imager](https://www.raspberrypi.com/software/) to a computer with an microSD card reader. Put the microSD card you'll use with your Raspberry Pi into the reader and run Raspberry Pi Imager.

![image](https://github.com/xLazaki/sds-project-final/blob/main/images/rpi.png)

> Raspberry Pi Imager is the quick and easy way to install Raspberry Pi OS and other operating systems to a microSD card, ready to use with your Raspberry Pi.

- Write microSD card with the following configurations:
    - Choose Raspberry Pi OS Lite (64-bit) as an operating system
    - Set hostname
    - Enable SSH using password authentication
    - Set username and password
    - Configure wireless LAN
- After writing is done, ssh into each pis using their local hostname or IP address to remote control into each Pi. For an IP address, open the dashboard of your wifi router and looking for connected devices. 
```
ssh <username>@<hostname>.local
ssh <username>@<IP>
```

### Static IP Configurations
1. Go to DHCP config file in Pi
```
sudo nano /etc/dhcpcd.conf
```
2. Find the settings of interface wlan0 (scroll down to the bottom) and change the following lines:
```
interface wlan0
static routers = <router_IP>
static domain_name_servers = <router_DNS_IP>
static ip_address = <pi_assigned_IP>
```
![image](https://github.com/xLazaki/sds-project-final/blob/main/images/dhcpconf.png)
3. Reboot Pi to make the change
```
sudo reboot
```
> Additional commands for debugging: <br>
> - hostname -I
> - ifconfig

### Installing K3s
1. Before installing K3s, enable c-groups by modifying the configuration file /boot/firmware/cmdline.txt for the kubelet:
```
sudo nano /boot/firmware/cmdline.txt
```
And append the following options:
```
cgroup_enable=memory cgroup_memory=1
```
Save the file in your editor and reboot:
```
sudo reboot
```
2. Download & Run Worker nodes 
```
curl -sfL https://get.k3s.io/ | K3S_URL=https://{IP’s server}:6443/ K3S_TOKEN=SECRET sh -
```
3. To use kubectl on worker nodes (or any client nodes), we need to config the kubctl config file using kubeconfig file from master nodes.
-  On master nodes, get k3s.yaml and copy the content
```
cat /etc/rancher/k3s/k3s.yaml
```
- On worker nodes, paste the content of k3s.yaml which replacing the localhost IP 127.0.0.1 of the server with the actual IP address of the master node in your network to **~/.kube/config**.
```
mkdir .kube
cd .kube
nano config
```
- You can also easily sending the copy of the file from a master node to other nodes via ssh using the following commands:
```
scp /etc/rancher/k3s/k3s.yaml <username>@<IP>:~/.kube/config
```
> **Now, the setting up is finished !**<br>
> ✺◟(＾∇＾)◞✺ . . . . . ✺◟(＾∇＾)◞✺
# How to deploy an application
- On Master nodes (VMs) run the following commands to create the deployments and services objects:
```
git clone https://github.com/xLazaki/sds-project-final.git
cd sds-project-final
kubectl create namespace vote
kubectl create -f k3s-app/
```
- The resulting pods are as belowed:
![image](https://github.com/xLazaki/sds-project-final/blob/main/images/deploy_result.png)
# Application
## Architecture
![image](https://github.com/xLazaki/sds-project-final/blob/main/images/app_architecture.png)
- A front-end web app in Python which lets you vote between two options: Cats and Dogs 
    - Using docker image from: https://hub.docker.com/r/taechitph/example-voting-app_vote
- A Redis queue which collects new votes
    - Using docker image from: https://hub.docker.com/_/redis
- A .NET Core worker which consumes votes and stores them in Postgres database
    - Using docker image from: https://hub.docker.com/r/taechitph/example-voting-app_worker
- A Postgres database backed by a Docker volume 
    - Using docker image from: https://hub.docker.com/_/postgres
- A Node.js webapp which shows the results of the voting in real time
    - Using docker image from: https://hub.docker.com/r/boomnatchanon/example_voting_app-result
## Details
- The vote interface is available on port 31000
![image](https://github.com/xLazaki/sds-project-final/blob/main/images/vote-endpoint.png)
- The result interface is available on port 31001.
![image](https://github.com/xLazaki/sds-project-final/blob/main/images/result-endpoint.png)
- Modified Version of [Official Docker Samples' Example Voting App](https://github.com/dockersamples/example-voting-app)
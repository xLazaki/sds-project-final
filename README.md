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
sudo k3s server --server https://<master1_ip>:6443 --token=SECRET
```
6. Marks the master nodes as unschedulable using Kubernetes Cordon.
```
sudo kubectl cordon <node_name>
```

## Setting up each Pis as K3s Worker nodes
### Writing microSD card
- Raspberry Pi Imager
- ssh into each pis using their local hostname or IP address. To get an IP address, open the dashboard of your wifi router and looking for connected devices.  

### Config static IP
1. Go to DHCP config file in Pi
```
sudo nano /etc/dhcpcd.conf
```
2. Find the settings of interface wlan0 (scroll down to the bottom) and change the following lines:
```
static routers = <router_IP>
static domain_name_servers = <router_DNS_IP>
static ip_address = <pi_assigned_IP>
```
3. Reboot Pi
```
sudo reboot
```
> Additional commands for debugging: <br>
> - hostname -I
> - ifconfig

### Installing K3s
1. Download & Run Worker nodes 
```
curl -sfL https://get.k3s.io/ | K3S_URL=https://{IP’s server}:6443/ K3S_TOKEN=SECRET sh -
```
2. Fix CA shit- Copy CA to each worker
-  On master nodes, get k3s.yaml
```
cat /etc/rancher/k3s/k3s.yaml
```
- On worker nodes, create kube config file
```
mkdir .kube
cd .kube
nano config
```
- paste all content from k3s.yaml to config, also don't forgot to change ip address to match your server's ip then save

# How to deploy an application
- On Master nodes (VMs):
```
git clone https://github.com/xLazaki/sds-project-final.git
cd sds-project-final
```

```
kubectl create -f k3s-app/
```
```
kubectl get pods
```
# Application Details
Modified Version of [Official Docker Samples' Example Voting App](https://github.com/dockersamples/example-voting-app) 
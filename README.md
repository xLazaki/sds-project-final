# แกไม่รอดแน่
# Preparation
- 5 x Raspberry Pi 3 Model B+ with microSD cards
- 2 x Ubuntu VMs (using Virtual box)
- (Optional) Router
- K3s is a lightweight Kubernetes distribution created by Rancher Labs.
# How to set up Kubernetes Cluster
## Setting up each VMs as K3s Master nodes
- Bridged
1. Update system
```
sudo apt update
sudo apt upgrade -y
```
2. Allow tcp port 6443
```
sudo ufw allow 6443/tcp
```
3. Install k3s
```
curl -sfL https://get.k3s.io | INSTALL_K3S_SKIP_ENABLE=true sh -
```
4. Built cluster
```
sudo k3s server --cluster-init --token=SECRET
```
5. Join server

```
sudo k3s server --server https://<master1ip>:6443 --token=SECRET
```
6. Marks the master nodes as unschedulable using Kubernetes Cordon.
```
sudo kubectl cordon <node_name>
```


## Setting up each Pis as K3s Worker nodes
### Writing microSD card

- ssh into each pis using their local hostname or IP address. To get an IP address, open the dashboard of your wifi router and looking for connected devices.  

### Config static IP
1. Go to DHCP config file
```
sudo nano /etc/dhcpcd.conf
```
2. Find the settings of interface wlan0 (scroll down to the bottom) and change the following lines:
```
static routers = 192.168.0.1
static domain_name_servers = 192.168.0.1
static ip_address = 192.168.0.{200+เลขpi}/24
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
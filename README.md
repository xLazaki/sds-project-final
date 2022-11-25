# แกไม่รอดแน่
# Preparation
- 5 x Raspberry Pi 3 Model B+ with microSD cards
- 2 x Ubuntu VMs (using Virtual box)
- (Optional) Router
- K3s is a lightweight Kubernetes distribution created by Rancher Labs.
# How to set up Kubernetes Cluster
## Setting up 2 VMs as K3s Master nodes
1. 
```
sudo apt update
sudo apt upgrade -y
```
2. 
```
sudo ufw allow 6443/tcp
```
3. 
```
curl -sfL https://get.k3s.io | INSTALL_K3S_SKIP_ENABLE=true sh -
```
4. 
```
sudo k3s server --cluster-init --token=SECRET
```

```
sudo k3s server --server https://<master1ip>:6443 --token=SECRET
```
5. Fix CA shit
```
```
```
```

## Setting up each Pis as K3s Worker nodes
### Writing microSD card
- ssh into each pis using their local

### Config static IP
1. 
```
sudo nano /etc/dhcpcd.conf
```
2. Find the settings of interface wlan0 (scroll down to the bottom) and change the following lines:
```
static routers = 192.168.0.1
static domain_name_servers = 192.168.0.1
static ip_address = 192.168.0.{200+เลขpi}/24
```
3. 
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
2. Fix CA shit

# How to deploy an application
- On Master nodes (VMs):
```
git clone https://github.com/xLazaki/sds-project-final.git

cd sds-project-final

kubectl create namespace vote

kubectl create -f k3s-app/
```
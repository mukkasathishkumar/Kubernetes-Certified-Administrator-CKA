#Create 3 VMs on GCP 

Goto https://console.cloud.google.com/
Repeat following steps 3 times for (master, node1 and node2)
Click Navigation menu > compute engine > VM instances > create instance 

Step 1 - Specify name (you can choose master or node1 or node2)
Step 2 - Select region as per your own location
Step 3 - Select machine (I have used e2-medium 2vcpu and 4gb Memory)
Step 4 - Select Bootdisk/ OS Ubuntu 18.04 LTS (New 10 GB standard persistent disk)
Step 5 - Allow default access
Step 6 - Allow HTTP traffic
Step 7 - Allow HTTPS traffic

#Create 3 VMs on GCP END

# Install the latest versions, this document may have instructions for older versions.
#Install docker and other dependencies on master, node1 and node2 (https://kubernetes.io/docs/setup/production-environment/container-runtimes/)
# (Install Docker CE)
## Set up the repository:
### Install packages to allow apt to use a repository over HTTPS
sudo apt-get update && sudo apt-get install -y \
  apt-transport-https ca-certificates curl software-properties-common gnupg2
# Add Docker's official GPG key:
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# Add the Docker apt repository:
sudo add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"
# Install Docker CE
sudo apt-get update && sudo apt-get install -y \
  containerd.io=1.2.13-2 \
  docker-ce=5:19.03.11~3-0~ubuntu-$(lsb_release -cs) \
  docker-ce-cli=5:19.03.11~3-0~ubuntu-$(lsb_release -cs)
# Set up the Docker daemon
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
sudo mkdir -p /etc/systemd/system/docker.service.d
# Restart Docker
sudo systemctl daemon-reload
sudo systemctl restart docker

If you want the docker service to start on boot, run the following command:

sudo systemctl enable docker

sudo modprobe overlay
sudo modprobe br_netfilter

# Set up required sysctl params, these persist across reboots.
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system
#Install docker and other dependencies on master, node1 and node2 END


#Install kubeadm on master
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/


sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

kubeadm init

********************************Yoy will see content like this please copy your content like this to a notepad*****************************************

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.148.0.5:6443 --token ow7z0c.8d717z8snprfwnb3 \
    --discovery-token-ca-cert-hash sha256:7b6ea24b25a81201becb5950a7dad8585d05abc0662882e01648a8f0826d9ba0 

********************************************************************************************************************************************************



mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

curl https://docs.projectcalico.org/manifests/calico.yaml -O

kubectl apply -f calico.yaml

#Install kubeadm on master END

#Join any number of worker nodes by running the following on each as root:

kubeadm join 10.148.0.5:6443 --token ow7z0c.8d717z8snprfwnb3 \
    --discovery-token-ca-cert-hash sha256:7b6ea24b25a81201becb5950a7dad8585d05abc0662882e01648a8f0826d9ba0 

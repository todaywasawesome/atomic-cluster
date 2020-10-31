This repo is basically a scratch pad for my home cluster. 

# Install K3s
## Server
wget https://get.k3s.io -O k3s-install.sh
chmod +x k3s-install.sh
systemctl status k3s-agent
./k3s-install.sh server --no-deploy traefik --no-deploy servicelb

## Client
curl -sfL https://get.k3s.io | K3S_URL=https://10.0.0.76:6443 K3S_TOKEN=K102e9aa86d859e5d9e1a2ebf7507d0aea9eaec754b960123992ae686aa93c7c93d::server:albe-apple-buster-brown sh -

## Automated upgrades
https://rancher.com/docs/k3s/latest/en/upgrades/automated/

##Install Codefresh
https://codefresh.io/features/codefresh-runner/
npm install -g codefresh
codefresh runner init

## Intel Quicksync Video
This allows access to GPU hardware and acceleration. 
https://wiki.ubuntu.com/IntelQuickSyncVideo
sudo apt install ubuntu-restricted-addons
https://elatov.github.io/2018/04/esxi-65-passthrough-video-card-to-plex-vm/

Testing with ffmpeg
apt install gstreamer1.0-vaapi

ffmpeg -hwaccel vaapi -hwaccel_device /dev/dri/renderD128 -hwaccel_output_format vaapi -i rolleroaster.mp4 -vf 'fps=30,scale_vaapi=w=640:h=-2:format=nv12' -c:v h264_vaapi -profile 578 -level 30 -bf 0 -b:v 1M -maxrate 1M rollercoaster-test.mp4
usermod -aG video kube

## Install Argo CD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Storage
## Setup PVC based on NFS
https://github.com/kubernetes/examples/tree/master/staging/volumes/nfs

Test write speed: https://www.cyberciti.biz/faq/howto-linux-unix-test-disk-performance-with-dd-command/
dd if=/dev/zero of=/data/tests.deleteme bs=1G count=1 oflag=dsync

Speed on rpi2 12MB/s
Speed on atomic pi 52MB/s

# Networking

## Deploy metalLB
Default k3s deploys wtih traefik which will not work with metalLB and cause the cluster to lockup and consume all cpu on the target machien. THIS WILL CONFLICT 

https://kauri.io/install-and-configure-a-kubernetes-cluster-with-k3s-to-self-host-applications/418b3bc1e0544fbc955a4bbba6fff8a9/a

helm install metallb stable/metallb --namespace kube-system \
  --set configInline.address-pools[0].name=default \
  --set configInline.address-pools[0].protocol=layer2 \
  --set configInline.address-pools[0].addresses[0]=10.0.0.20-10.0.0.60

## Deploy Ingress

helm upgrade --install nginx-ingress ingress-nginx/ingress-nginx --namespace kube-system \
    --set controller.service.loadBalancerIP=10.0.0.22 \
    --set defaultBackend.enabled=false

 Fails to start because permission denied on fake ssl cert creation. what's this run as 33 user? 

# Plex
## Plex

helm upgrade plex ./kube-plex/charts/kube-plex \
	--values media.plex.values.yml

## url in cluster 
my-svc.my-namespace.svc.cluster-domain.example
http://sabnzbd-service.default.svc.cluster.local:8080

# Minecraft server
helm install --upgrade minecraft stable/minecraft --values minecraft.values.yaml

# PXE Boot
- Using LTSP https://kubernetes.io/blog/2018/10/02/building-a-network-bootable-server-farm-for-kubernetes-with-ltsp/

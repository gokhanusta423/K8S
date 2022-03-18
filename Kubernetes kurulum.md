# KUBERNETES KURULUMU
NOT: kubernets i production ortamı için aşağıdaki anlatıldığı gibi kurmalıyız.
https://kubernetes.io/docs/setup/production-environment/container-runtimes/

- Download imaj.

mkdir ubuntubaseimage
cd ubuntubaseimage
wget http://cdimage.ubuntu.com/ubuntu-base/releases/20.04/release/ubuntu-base-20.04.4-base-amd64.tar.gz
tar -xfv tar -xvf ubuntu-base-20.04.4-base-amd64.tar.gz 
- Vmware kurulum yap. 
- ssh ile bağlanılır yap
    - **sudo apt update**
    - **sudo apt-get install openssh-server**
    - **sudo systemctl enable ssh**
    - **sudo systemctl start ssh**
- Master ve worker lara IP ver.
- **sudo nano /etc/netplan/01-netcfg.yaml**
- **sudo netplan apply**

- **nmcli d** komuutu ile mevcut eth arayüzlerini gör
-**sudo nano /etc/netplan/01-netcfg.yaml**  komutu ile yaml dosyasını ekle. 
---
    - Master  =192.168.191.10
    - Worker1 =192.168.191.11
    - Worker2 =192.168.191.12
    - Worker3 =192.168.191.13
---
Kubeadm kullanılarak kurulum aşağıdaki şekilde
- **sudo apt-get update**

1- Tüm node lara docker kur
- **sudo apt install docker.io  curl**
- **docker --version**
- **sudo systemctl enable docker**
- **sudo apt-get install gnupg2** gpg için ilgili paketi ekle.

2- Google gpg key inin download ed bu key i apt-key add diyerek apt nin key lerine ekle. Kubernetes reposunu repoların içine ekle. 
- **sudo apt install software-properties-common -y** add-apt-repository gibi ek komutlar için paket.
- **curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add**
- **sudo add-apt-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"**
 
3- swap alanını sıfırla
 - **sudo swapoff –a**
- fstab dosyasından swap kaydını # ile disable ediyoruz. 

4-	Tüm nodelara kubelet, kubeadm,kubectl kur
- **sudo apt-get update**
- **sudo apt-get install -y kubelet kubeadm kubectl**
- **sudo apt-mark hold kubelet kubeadm kubectl**  apt-mark paketlerle ilgili bazı ayarlar yapmamızı sağlar. örn: yukarıda paketler, hold işareti kaldırılmadan kurulamaz, kaldırılamaz, temizlenemez veya yükseltilemez. Bu şekilde paketler "apt update" gibi komutlar dan korunur. 

5-	MASTER NODE a Kubeadm init diyip network vererek kurulumu yap 
Sadece MASTER üzerine aşağıdaki komut çalıştır. 
 - **sudo kubeadm init --pod-network-cidr=172.168.10.0/24**
 master üzerinde kurulum sırasında problem çıkarsa 
 önce **sudo systemctl stop kubelet.service** diyerek kubelet i durdur.
 - **sudo kubeadm reset**  komutu ile kurulan dosyaları silip tekrardan **kubeadm init ...** diyebilirsin.
--------
 NOT: Burada **kubeadm init** dediğimizde kubelet in ayağa kalkamadığını gördük. çözümü 

https://stackoverflow.com/questions/54728254/kubernetes-kubeadm-init-fails-due-to-dial-tcp-127-0-0-110248-connect-connecti

sitesinde bulduk

-   sudo mkdir /etc/docker
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
    sudo systemctl enable docker
    sudo systemctl daemon-reload
    sudo systemctl restart docker
    sudo kubeadm reset
    sudo kubeadm init
---------------------


NOT: kubectl önce kullanıcının home klasörünün altındaki .kube/config dosyasına bakıyor eğer onu bulamazsa KUBECONFIG çevresel deg. belirtilen yoldaki config dosyasını arıyor. 
eğer standart kullanıcı isek $KUBECONFIG in boş olduğundan emin olmalıyız.
cp /etc/kubernetes/admin.conf ~/.kube/config şeklinde kendi home klasörümüze almalıyız.
eğer root  kullanıcısı ise export KUBECONFIG=/etc/kubernetes/admin.conf komutunu çalıştırmalıyız. 

- **kubectl config current-context**
- **kubectl config get-contexts**
- **kubectl config view**
- **kubectl config use-context kontextismi**
---------


6- Aşağıdaki uyarıda denilenleri yap.

our Kubernetes control-plane has initialized successfully!
 
To start using your cluster, you need to run the following as a regular user:
 
- **mkdir -p $HOME/.kube**
- **sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config**
- **sudo chown $(id -u):$(id -g) $HOME/.kube/config**

Alternatively, if you are the root user, you can run:
 
  - **export KUBECONFIG=/etc/kubernetes/admin.conf**
 
7-	You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/
 
Then you can join any number of worker nodes by running the following on each as root:
 
kubeadm join 192.168.184.101:6443 --token 6sc4hj.k3kpdfr1fk7oy52j --discovery-token-ca-cert-hash sha256:f7406eead8570a6b31d11c05b3f7fe6040544679b22efade8c96946e9858c90c 
 
--------------------------------------------------------------------------------------
Burada ikinci adımda belirtilen network kurulumunda flannel kurmak için 
 
kubectl apply -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml

Diyoruz ama bu komut hata verirse
RBAC modülünüde kurmak gerek ozaman 
kubectl create -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel-rbac.yml
 
 
#### KURULUMU TEST ETMEK İÇİN
 
- **Kubectl get nodes**
- **kubectl get pods --all-namespaces**
- **Docker container ls**


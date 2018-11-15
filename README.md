### How to install Kubernetes Cluster:

1) Architecture:

    Master Node: 192.168.135.111 k8s-master
    Worker Node: 192.168.135.121 worker-node-1
    Worker Node: 192.168.135.122 worker-node-2


2) Master Node:


Add Kubernetes repo:

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
> [kubernetes]
> name=Kubernetes
> baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
> enabled=1
> gpgcheck=1
> repo_gpgcheck=1
> gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
>         https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
> EOF


Install kubernetes and Docker :



    . yum install kubeadm docker -y

    . systemctl restart docker && systemctl enable docker

    . systemctl  restart kubelet && systemctl enable kubelet
    
Turn swapp Off

    . swapoff -a && sed -i '/swap/d' /etc/fstab

Init Kubernetes admin with advertise IP of Master Node, and POD network CIDR 10.1.0.0/16, so you can initialice the network module kube-router:

    . kubeadm init --apiserver-advertise-address 192.168.135.111 --pod-network-cidr 10.1.0.0/16

Initialice router plugin:

    . kubectl apply -f https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter.yaml

Verify everything is OK

    . kubectl get nodes

    . kubectl  get pods  --all-namespaces

3) Worker Node:

    Install kubernetes and Docker :


    . yum install kubeadm docker -y

    . systemctl restart docker && systemctl enable docker

    . systemctl  restart kubelet && systemctl enable kubelet

    Turn swapp Off

    . swapoff -a && sed -i '/swap/d' /etc/fstab

    Join cluster with token and hash derived from init on master. example: ( kubeadm join 192.168.135.111:6443 --token hrnutk.5c8fg801igp7sesb --discovery-token-ca-cert-hash sha256:0814ba20f7a1cf85d45fc5cbb179bd2d86f29bfc9f924e5c877f1e12002b9b3e )


    Recomended commands to run on Worker Nodes :

    . sudo yum install ipvsadm -y
    . sudo modprobe -- ip_vs_rr
    . sudo modprobe -- ip_vs_wrr
    . sudo modprobe -- ip_vs_sh
    . sudo yum install net-tools -y


4)

    Install Dashboard: 

    . kubectl apply -f https://gist.githubusercontent.com/initcron/32ff89394c881414ea7ef7f4d3a1d499/raw/4863613585d05f9360321c7141cc32b8aa305605/kube-dashboard.yaml
 


  watch -n 1 kubectl get nodes,pods,svc,deployment --all-namespaces -o wide


  kubectl create namespace sock-shop

  kubectl apply -n sock-shop -f config/socks.yaml
# Setup cluster kubernetes 3 node with kubeadm using ansible
## My environment

| hostname | ip  | os  | ansible version | virtualization | kubernetes version | containerd version |
| --- | --- | --- | --- | --- | --- | --- |
| ops-master | 10.10.10.60 | 20.04.6 LTS (Focal Fossa) | 2.12.10 | kvm | v1.29.15 | containerd://1.7.24 |
| ops-worker-1 | 10.10.10.61 | 20.04.6 LTS (Focal Fossa) | \-  | kvm | v1.29.15 | containerd://1.7.24 |
| ops-worker-2 | 10.10.10.62 | 20.04.6 LTS (Focal Fossa) | \-  | kvm | v1.29.15 | containerd://1.7.24 |
## Setup ssh

Setup ssh for all node for ssh passwordless to user

### execute from master-1
create pubkey

```
ssh-keygen -t rsa
```

copy /.ssh/id.rsa.pub to all node

```
cat ~/.ssh/id_rsa.pub
```

![](/assets/cat-ssh.png)

insert id\_rsa.pub master to all worker node

```
nano ~/.ssh/authorized_keys
```
![](/assets/insert-pubkey.png)  

after insert pubkey , test ssh to all worker node to user root , make sure the ssh passwordless

## Install ansible

Install ansible on master node

```
sudo apt install software-properties-common -y
```

```
sudo add-apt-repository --yes --update ppa:ansible/ansible
```

```
sudo apt install ansible -y
```

makesure ansible has been installed

```
ansible --version
```

![](/assets/ansible-version.png)

## Setup ansible
If you want custom version kubernetes, enpoint_kube, portkube and cni . Setup at vars.yml

![](/assets/vars.png)

to check available version using this command

```
apt-cache policy kubelet
apt-cache policy kubeadm
apt-cache policy kubectl
```

## Run ansible
Run ansible at master node for boostrap cluster
```
ansible-playbook -i inventory.yml playbooks.yml
```

If ansible done , make bash completion for kubernetes

```
cat <<EOF | tee -a ~/.profile
source <(kubectl completion bash)
alias k=kubectl
complete -F __start_kubectl k
EOF
```

```
source ~/.profile
```

## Check cluster
make sure node all ready , and pod at namespace kube-system all running well
```
k get nodes
```

![](/assets/node.png)

>Make sure node all running

```
k get pods -n kube-system
```

![](/assets/pod-kube-system.png)

>Make sure pod all runing

## Deploy simple deployment for test 

Deploy simple deployment

```
k apply -f deployment.yaml
```

Check deployment

```
k get pods
```

![](/assets/pod-kube-system.png)

>Make sure pod all runing

## Sanity for deployment

Check ip services 

```
k get svc
```
![](/assets/svc.png)

Curl ip services

```
curl 10.103.131.59
```
Make sure the output its oke

![](/assets/tescurlsvc.png)


## Destroy cluster
Run ansible destroy.yml to destroy cluster
```
ansible-playbook -i inventory.yml destroy.yml
```

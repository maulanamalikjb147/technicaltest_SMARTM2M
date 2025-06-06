# Setup cluster kubernetes 3 node with kubeadm using ansible

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

## Run ansible
Run ansible for boostrap cluster
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

> _Make sure node all running

```
k get pods -n kube-system
```

![](/assets/pod-kube-system.png)

> _Make sure pod all runing

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

> _Make sure pod all runing

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

# Setup nginx with client server using ansible
## My environment

| hostname | ip  | os  | nginx version | ansible version | virtualization | note |
| --- | --- | --- | --- | --- | --- | --- |
| ops-master | 10.10.10.60 | 20.04.6 LTS (Focal Fossa) | nginx version: nginx/1.18.0 (Ubuntu) | 2.12.10 | kvm | Loadbalancer & client servers |
| ops-worker-1 | 10.10.10.61 | 20.04.6 LTS (Focal Fossa) | nginx version: nginx/1.18.0 (Ubuntu) | \- | kvm | client servers |
| ops-worker-2 | 10.10.10.62 | 20.04.6 LTS (Focal Fossa) | nginx version: nginx/1.18.0 (Ubuntu) | \- | kvm | client servers |

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

## Setup ansible
If you want custom version nginx, location ssl ,  and host client server. Setup at vars.yml

![](/assets/nginx_vars.png)

to check available version using this command

```
apt-cache show nginx
```

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
Run ansible at master node for boostrap cluster
```
ansible-playbook -i inventory.yml playbooks.yml
```

If ansible done , check nginx status

```
systemctl status nginx
```
![](/assets/nginxstatus.png)

> Make sure all nginx running

## Check nginx
make sure on all client hosts, there is an index.html file in /var/www/html/index.html, paint it and make sure the contents have an ip according to the host.

```
cat /var/www/html/index.html
```

![](/assets/catindexhtml.png)

make sure in loadbalancer there is nginx_selfsigned file in /etc/nginx/sites-available /etc/nginx/sites-enabled

```
ls /etc/nginx/sites-available | grep nginx_selfsigned
```
```
ls /etc/nginx/sites-enabled | grep nginx_selfsigned
```

![](/assets/checkfilenginx.png)

make sure in loadbalancer , cert sucessfully generated at /etc/nginx/ssl

```
ls /etc/nginx/ssl
```

![](/assets/checkssl.png)


## Sanity nginx

hit ip loadbalancer

```
curl -k https://10.10.10.60
```

> Do hit 6 times , or 3 times

or you can try curl with looping

```
for i in {1..6}; do curl -k https://10.10.10.60; echo ""; done
```

![](/assets/curlnginx.png)

> Make sure the output :
Welcome this from 10.10.10.60
Welcome this from 10.10.10.61
Welcome this from 10.10.10.62

because :
Welcome this from 10.10.10.60 > from 10.10.10.60

Welcome this from 10.10.10.61 > from 10.10.10.61

Welcome this from 10.10.10.62 > from 10.10.10.62

is it rule from loadbalancer


## Destroy cluster
Run ansible destroy.yml to destroy cluster
```
ansible-playbook -i inventory.yml destroy.yml
```

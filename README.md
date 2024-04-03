## Automation Build PostgreSQL VM image for NDB

Berikut adalah cara deploy PostgreSQL standalone / High Available menggunaka tools Ansible Playbook, Ada beberapa tools yang kita gunakan diantaranya:

- PostgreSQL Community
- HA-Proxy
- etcd
- patroni

Seperti pada diagram berikut:

![diagram-architecture]()

Pertama kita install package ansible di laptop / Virtual Machine, Ansible ini akan digunakan untuk provision tools/software tersebut.

Untuk menggunakan ansible kita bisa pasang di Linux, MacOS dan Windows (WSL2)

```bash
# install for mac
brew install ansible

# install for ubuntu
apt-get install -y ansible

# install for centos
dnf/yum install -y ansible
```

Ref:
- [how to install ansible](https://docs.ansible.com/ansible/2.9/installation_guide/intro_installation.html)

## Preparation and Requirement

Ada beberapa hal yang perlu kita siapkan untuk meng-install tools DevSecOps tersebut diantaranya Virtual Machine dengan minimum specifikasi seperti berikut:

```yaml
cpus: 4 core
ram: 4 GB
os: OracleLinux 8.7
network:
    type: dhcp
    ip: 10.12.12.1 # contoh, silahkan sesuaikan dengan kondisi infra
    gw: 10.12.12.254
storage: 
    partisions:
        "/": 50 GB
```

Dan menggunakan **credential yang sama** untuk memudahkan provision by ansible, setelah ter-deploy/provision kita bisa **ganti password yang lebih secure!!**

```yaml
username: admin
password: password
```

## Using this Ansible Playbook

Pertama kita clone dulu repository ini, dengan perintah berikut:

```bash
git clone https://github.com/dimMaryanto93/ansible-playbook-ndb-postgres.git --depth 1 && \
cd ansible-playbook-ndb-postgres ## masuk ke folder ansible
```

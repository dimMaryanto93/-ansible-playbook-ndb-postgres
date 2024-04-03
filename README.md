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

Setelah itu kita perlu install dependency dengan menggunakan perintah berikut:

```bash
ansible-galaxy role install -r requirements.yaml --force && \
ansible-galaxy collection install -r requirements.yaml --force
```

Untuk provision perlu step-by-step flownya seperti berikut

- Create VM, sesuai dengan specifikasi
- Export VM sebagai ova dan image disk pada nutanix prism central
- Update file `inventory.ini`
- generate [ssh private/public key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent?platform=linux) if not exists
- add ssh key to server, become passwordless
- Jalankan ansible script ini dengan `ansible-playbook`

```bash
ssh-copy-id <username>@<ip-address-server>
```

Kemudian jalankan perintah berikut:

```bash
ansible-playbook -i inventory.ini --extra-vars='@vars.yaml' site.yaml --ask-become-pass
```

Jika dijalankan hasilnya seperti berikut:

```bash
💻 ~/D/p/n/g/ansible ➡ ansible-playbook -i inventory.ini --extra-vars='@vars.yaml' site.yaml --ask-become-pass
BECOME password: ## enter admin password for sudo access

playbook: site.yaml

  play #1 (all): Provision PostgreSQL for NDB   TAGS: []
    tasks:
      dimmaryanto93.keepalived : Install dependencies for keepalived    TAGS: []
      dimmaryanto93.keepalived : Ensure keepalived is installed.        TAGS: []
      dimmaryanto93.keepalived : Get keepalived version.        TAGS: []
      dimmaryanto93.keepalived : Debug version of keepalived    TAGS: []
      dimmaryanto93.ha_proxy : Install dependencies for HA-Proxy        TAGS: []
      dimmaryanto93.ha_proxy : Ensure HA-Proxy is installed.    TAGS: []
      dimmaryanto93.ha_proxy : Get HAProxy version.     TAGS: []
      dimmaryanto93.ha_proxy : Debug version of HAProxy TAGS: []
      ansible.posix.sysctl      TAGS: []
      ansible.posix.sysctl      TAGS: []
      dimmaryanto93.ha_proxy : Ensure HAProxy is started and enabled on boot.   TAGS: []
      dimmaryanto93.postgres_community : Load a variable file based on the OS type      TAGS: []
      dimmaryanto93.postgres_community : Install basic dependencies     TAGS: []
      dimmaryanto93.postgres_community : Install PostgreSQL server for RedHat   TAGS: []
      dimmaryanto93.postgres_community : Install PostgreSQL server for RedHat   TAGS: []
      dimmaryanto93.postgres_community : Install Python package manager TAGS: []
      dimmaryanto93.postgres_community : Make sure a service unit is running    TAGS: []
      dimmaryanto93.postgres_community : Configure PostgreSQL server    TAGS: []
      dimmaryanto93.postgres_community : Install patroni binary TAGS: []
      dimmaryanto93.postgres_community : Install etcd plugin pip        TAGS: []
      dimmaryanto93.postgres_community : Install etcd database  TAGS: []
      dimmaryanto93.nutanix_db-patch-os : create group of user era      TAGS: []
      dimmaryanto93.nutanix_db-patch-os : create user for era user      TAGS: []
      dimmaryanto93.nutanix_db-patch-os : Make users passwordless for sudo in user era  TAGS: []
      dimmaryanto93.nutanix_db-patch-os : Install python binnary        TAGS: []
      dimmaryanto93.nutanix_db-patch-os : Install epel-release for RedHat       TAGS: []
      dimmaryanto93.nutanix_db-patch-os : Install python package manager        TAGS: []
      dimmaryanto93.nutanix_db-patch-os : Update pip engine to at least TAGS: []
      dimmaryanto93.nutanix_db-patch-os : Check pip version     TAGS: []
      dimmaryanto93.nutanix_db-patch-os : debug pip version     TAGS: []
      dimmaryanto93.nutanix_db-patch-os : Install NFS package   TAGS: []
      dimmaryanto93.nutanix_db-patch-os : Create a directory ==> /opt/ndb if it does not exist  TAGS: []
      dimmaryanto93.nutanix_db-patch-os : Copy file era_linux_prechecks.sh to server    TAGS: []
      dimmaryanto93.nutanix_db-patch-os : Copy file era_windows_prechecks.sh to server  TAGS: []

  play #2 (all): Post install config    TAGS: []
    tasks:
      Remove data etcd  TAGS: []
      Disable and Stop service etcd     TAGS: []

  play #3 (all): Running precheck postgresql    TAGS: []
    tasks:
      Check NDB precheck        TAGS: []
      debug     TAGS: []

TASK [Check NDB precheck] *************************************************************************************************************************
changed: [ndb-postgres15]

TASK [debug] **************************************************************************************************************************************
ok: [ndb-postgres15] => {
    "msg": [
        "",
        "",
        "--------------------------------------------------------------------",
        "|              Era Pre-requirements Validation Report              |",
        "--------------------------------------------------------------------",
        "",
        "     General Checks:",
        "     ---------------",
        "         1] Username           : root",
        "         2] Package manager    : yum",
        "         2] Database type      : postgres_database",
        "",
        "     Era Configuration Dependencies:",
        "     -------------------------------",
        "         1] User has sudo access                         : YES",
        "         2] User has sudo with NOPASS access             : YES",
        "         3] Crontab configured for user                  : YES",
        "         4] Secure paths configured in /etc/sudoers file : YES",
        "",
        "     Era Software Dependencies:",
        "     --------------------------",
        "          1] GCC                  : N/A",
        "          2] readline             : YES",
        "          3] libselinux-python    : YES",
        "          4] crontab              : YES",
        "          5] lvcreate             : YES",
        "          6] lvscan               : YES",
        "          7] lvdisplay            : YES",
        "          8] vgcreate             : YES",
        "          9] vgscan               : YES",
        "         10] vgdisplay            : YES",
        "         11] pvcreate             : YES",
        "         12] pvscan               : YES",
        "         13] pvdisplay            : YES",
        "         14] zip                  : YES",
        "         15] unzip                : YES",
        "         16] rsync                : YES",
        "         17] bc                   : N/A",
        "         18] sshpass              : N/A",
        "         19] ksh                  : N/A",
        "         20] lsof                 : YES",
        "         21] tar                  : YES",
        "         22] xfsprogs             : N/A",
        "         23] ifupdown             : N/A",
        "         24] net-tools            : N/A",
        "         25] nftables             : N/A",
        "",
        "     Summary:",
        "     --------",
        "         This machine satisfies dependencies required by Era, it can be onboarded.",
        "",
        "     **WARNING: Cluster API was not provided. Couldn't go ahead with the Prism API connectivity check.",
        "     Please ensure Prism APIs are callable from the host.",
        "=================================="
    ]
}

PLAY RECAP ****************************************************************************************************************************************
ndb-postgres15             : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
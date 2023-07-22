---
title: System Setup
---

# Setup

Describes how the server VM has been setup from the [map5topo repo](https://github.com/map5nl/map5topo).
This basically describes how you can setup your local system to start preparing data and creating maps.

Below all steps are documented. 

## 1. Ubuntu Server

Info:

* Hetzner Cloud Linux Ubuntu server Virtual Machine (VM), with Ubuntu 20.4 
* Specs: CX41 2 VCPU  8 GB  RAM  40 GB DISK LOCAL - 15.90 a month
* Extra Volume 300GB  - 12.00 a month
* IP address `65.108.253.148`

Prepare server steps:

* DNS: create A-record `topo.map5.nl` for IP address `65.108.253.148`
* local user with full sudo rights e.g. `sudo su -`
* Upgrade server to latest: `apt-get update && apt-get -y upgrade`
* provide local user direct SSH root access via `authorized_keys` (needed for Ansible)


### Extra Volume

300GB for tile caches and Docker storage.

``` {.bash linenums="1"}
sudo mkfs.ext4 /dev/disk/by-id/scsi-0HC_Volume_19990557
mkdir /var/map5
mount -o discard,defaults /dev/disk/by-id/scsi-0HC_Volume_19990557 /var/map5
echo "/dev/disk/by-id/scsi-0HC_Volume_19990557 /var/map5 ext4 discard,nofail,defaults 0 0" >> /etc/fstab

```

Later we will configure Docker daemon to use this storage.

600GB for AHN source and hillshade Tiffs. After preparation we'll store in long-term storage and remove this volume.
Output result is around 90GB Hillshade files.

``` {.bash linenums="1"}


sudo mkfs.ext4 /dev/disk/by-id/scsi-0HC_Volume_20257946
sudo mkdir /var/data
sudo mount -o discard,defaults /dev/disk/by-id/scsi-0HC_Volume_20257946 /var/data
echo "/dev/disk/by-id/scsi-0HC_Volume_20257946 /var/data ext4 discard,nofail,defaults 0 0" >> /etc/fstab
sudo mkdir /var/data/dem


```
## 2. Prepare Local System

On your local system you mainly need to have `Ansible` and `Git` (client) installed:

### Install Ansible:

* have Python 3 (3.7 or better) installed
* OPTIONAL (but recommended) create a Python Virtualenv (for Ansible)  
* install [Ansible](https://www.ansible.com/) with `pip install ansible` 2.9.* or higher
* test: `ansible --version` - shows *ansible 2.9.19 ...*
* test: `ansible-vault --version` shows *ansible-vault 2.9.19 ...*
 
More on Ansible below.

### Install `Git` client.
 
Depends on your system. Make sure you have a command line `git` (CLI).

## 3. Prepare New GitHub Repo

Clone the Git repo locally: `git clone https://github.com/map5nl/map5topo.git`

We will call the root dir of the cloned git repo on your system just `git/` from here.

## 4. Setup Ansible

Most of the configuration that is specific to your new server 
is stored under:

* `git/ansible/hosts` (Ansible inventories)
* `git/ansible/vars` (variables and SSH keys). 

Files under `git/ansible/vars` need to be always encrypted with `Ansible Vault`. You will need to 
create your own (encrypted) version of these encrypted files. 
For many files an example file is given. 

### Install Ansible Modules

Called "Roles" these are third-party Ansible components that help with specific tasks.
Install these as follows:

``` {.bash linenums="1"}
cd git/ansible
ansible-galaxy install --roles-path ./roles -r requirements.yml
```

### Ansible Hosts
The hostname is crucial to services functioning. Two steps:

* set content of `git/ansible/hosts/prod.yml` (Inventory) to

``` {.yaml linenums="1"}
map5topo:
  hosts:
    MAP5TOPO:
       ansible_port: 22
       ansible_host: topo.map5.nl
       ansible_user: root
       ansible_python_interpreter: /usr/bin/python3
```


* note: `MAP5TOPO` will also be the new hostname, and prompt name 
* set content of `git/env.sh` (common environment Docker-based services) to:

``` {.bash linenums="1"}
#!/bin/bash
# Sets global env vars based on host-name
# Needed for various host-dependent configs, especiallly Traefik SSL-certs.

# Export and Defaults

# You may (or not may) set/export these vars externally
# MAP_AREA
# "muiden" implies small dataset Muiden area,
# "utrecht" implies larger dataset about Utrecht province,
# "nl" is full NL dataset
export MAP_AREA=${MAP_AREA}

# DEPLOY_SERVER
# "local" implies localhost and http.
# "prod" implies a domain and https
export DEPLOY_SERVER=${DEPLOY_SERVER:-local}

# Assume local deployment
export TRAEFIK_DOMAIN="localhost"
export TRAEFIK_SSL_ENDPOINT=
export TRAEFIK_SSL_DOMAIN=
export TRAEFIK_SSL_CERT_RESOLVER=
export TRAEFIK_USE_TLS="false"
export MAP5_SITE_URL=

export HOST_UID=$(id -u)
export HOST_GID=$(id -g)
export HOST_UID_GID="${HOST_UID}:${HOST_GID}"

export VAR_DIR="/var/map5"
export GIT_DIR=
export DATA_DIR="${VAR_DIR}/data"

# Set host-dependent vars
case "${HOSTNAME}" in
    # map5topo development server
    "MAP5TOPO")
        DEPLOY_SERVER="prod"
        GIT_DIR="/home/madmin/git"
        MAP_AREA=${MAP_AREA:-nl}
        TRAEFIK_SSL_DOMAIN="topo.map5.nl"
        TRAEFIK_SSL_CERT_RESOLVER="le"
        ;;
    # map5.nl official test server
    "MAP5TEST")
        DEPLOY_SERVER="prod"
        GIT_DIR="/home/madmin/map5topo.git"
        MAP_AREA=${MAP_AREA:-nl}
        TRAEFIK_SSL_DOMAIN="test.map5.nl"
        TRAEFIK_SSL_CERT_RESOLVER="default"
        ;;
    # Just's Mac
    "nusa")
        DEPLOY_SERVER="local"
        GIT_DIR="/Users/just/project/map5/map5topo.git"
        MAP_AREA=${MAP_AREA:-muiden}
        ;;
    # Niene - no 1
    "niene-ThinkPad-P53s")
        DEPLOY_SERVER="local"
        GIT_DIR="/home/niene/Documents/git_projecten/map5topo"
        ;;
    # Niene - no 2
    "niene-desktop")
        DEPLOY_SERVER="local"
        GIT_DIR="/home/niene/Documents/git_projecten/map5topo"
        ;;
# PLACE HERE THE RESULT 'hostname' on your system, basically your GIT root dir
# for the map5topo GH repo.
#    "myhost")
#        DEPLOY_SERVER="local"
#        GIT_DIR="/your/map5topo/git root/dir/git"
#        ;;
    *)
        echo "Your host ${HOSTNAME} is unknown - PLEASE ADD YOUR HOSTNAME IN git/env.sh - exit"
        exit 1
esac

# default
MAP_AREA=${MAP_AREA:-utrecht}

if [[ ${DEPLOY_SERVER} = "local" ]]
then
    MAP5_SITE_URL="http://localhost:8000"
    DATA_DIR="${GIT_DIR}/data"
    VAR_DIR="${GIT_DIR}/services/mapproxy/var"

elif [[ ${DEPLOY_SERVER} = "prod" ]]
then
	source /etc/environment
  TRAEFIK_SSL_ENDPOINT="https"
  TRAEFIK_USE_TLS="true"
  TRAEFIK_DOMAIN="${TRAEFIK_SSL_DOMAIN}"
  MAP5_SITE_URL="https://${TRAEFIK_SSL_DOMAIN}"
fi

# Data PostGIS dump download files
export BAG_GZ_FILE_NAME="bagv2-pand-${MAP_AREA}.sql.gz"
export BGT_DUMP_FILE_NAME="bgt-lean-${MAP_AREA}.dump"
export BRK_GZ_FILE_NAME="kadgrens-${MAP_AREA}.sql.gz"
export BRT_TOP10_GZ_FILE_NAME="top10nl-${MAP_AREA}.sql.gz"
export BRT_TOP50_GZ_FILE_NAME="top50nl-${MAP_AREA}.sql.gz"
export BRT_TOP100_GZ_FILE_NAME="top100nl-${MAP_AREA}.sql.gz"
export GRID_RD_1KM_FILE_NAME="grid-rd-1km.sql"
export NWB_GZ_FILE_NAME="nwb-wegen-${MAP_AREA}.sql.gz"
export OSM_SEA_GZ_FILE_NAME="sea-polygons-28992-nl.sql.gz"
export OSM_SEA_SIMP_GZ_FILE_NAME="sea-polygons-simplified-28992-nl.sql.gz"

# Not used (yet)
export BRT_TOP250_GZ_FILE_NAME="top250nl-${MAP_AREA}.sql.gz"
export BRT_TOP500_GZ_FILE_NAME="top500nl-${MAP_AREA}.sql.gz"

# Tools locations
export TOOLS_DIR="${GIT_DIR}/tools"
export MAPNIK_DIR="${TOOLS_DIR}/mapnik"
export DEM_DIR="${TOOLS_DIR}/dem"
export ETL_DIR="${TOOLS_DIR}/etl"

# Local data folders
export BAG_DATA_DIR="${DATA_DIR}/bag"
export BGT_DATA_DIR="${DATA_DIR}/bgt"
export BRK_DATA_DIR="${DATA_DIR}/brk"
export BRT_DATA_DIR="${DATA_DIR}/brt"
export DEM_DATA_DIR="${DATA_DIR}/dem/${MAP_AREA}"
export GRID_DATA_DIR="${DATA_DIR}/grid"
export NWB_DATA_DIR="${DATA_DIR}/nwb"
export OSM_DATA_DIR="${DATA_DIR}/osm"

echo "env.sh: DEPLOY_SERVER=${DEPLOY_SERVER} - MAP_AREA=${MAP_AREA} - MAP5_SITE_URL=${MAP5_SITE_URL}"
export LOG_DIR="${VAR_DIR}/log"


```
 
So `MAP_AREA=prod` here is to discern with a deployment on `localhost` (`MAP_AREA=test`, where .e.g. no https/SSL is used).

### Create SSH Keys

These are used to invoke actions on the server both from GitHub Actions (via GitHub Sercrets) 
and from your local Ansible setup. Plus a set of authorized_keys for the admin SSH user.

* cd `git/ansible/vars`
* create new SSH keypair (no password): `ssh-keygen -t rsa -q -N "" -f gh-key.rsa`

### Create authorized_keys

Create new `git/ansible/vars/authorized_keys` with your public key and for others you want to give access to the admin SSH account,
plus `gh-key.rsa.pub` .

``` {.bash linenums="1"}
cat gh-key.rsa.pub > authorized_keys
cat ~/.ssh/id_rsa.pub >> authorized_keys
cat id.rsa.pub.of.joe >> authorized_keys   # etc

```
    
Set these for the `root` and `<admin user>` in their `.ssh/authorized_keys`.
 
See `MAP5TOPO_GH`  GitHub Deploy key below.

``` {.bash linenums="1"}
scp vars/gh-key.rsa root@topo.map5.nl:.ssh/id_rsa

```
### Set GitHub Deploy Key and Secrets

Go to GH repo Settings/keys

* set DEPLOY_KEY from `git/ansible/vars/gh-key.rsa.pub`

Go to GH repo Settings|Secrets and create these secrets for Actions:

* ANSIBLE_SSH_PRIVATE_KEY - with value from `git/ansible/vars/gh-key.rsa`
* ANSIBLE_VAULT_PASSWORD - value from `~/.ssh/ansible-vault/map5topo.txt` 

### Adapt vars.yml

Create new `git/ansible/vars/vars.yml` from example `vars.example.yml` in that dir.

The first part of `vars.yml` contains generic, less-secret, values. 
Use variables where possible. Format is Python-Jinja2 template-like:


``` {.yaml linenums="1"}
my_ssh_pubkey_file: ~/.ssh/id_rsa.pub
my_email: my@email.nl
my_admin_user: the_admin_username
my_admin_home: "/home/{{ my_admin_user }}"
my_git_home: "{{ my_admin_home }}/git"
my_github_repo: git@github.com:map5nl/map5topo.git
var_dir: /var/ogcapi
logs_dir: "{{ var_dir }}/log"
services_home: "{{ my_git_home }}/services"
pip_install_packages:
  - name: docker
timezone: Europe/Amsterdam
ufw_open_ports: ['22', '80', '443']
```
  
Note the GitHub repo is SSH-based for deploy-key!

The second part deals with more secret values, like usernames and passwords for services.
These will be copied into the VM's `/etc/environment` file.

``` {.yaml linenums="1"}
etc_environment:
  PG_DB: the_db  # PostGIS service
  PG_USER: the_user  # PostGIS service
  PG_PASSWORD: the_pw  # PostGIS service
  PGADMIN_EMAIL: the_user@the_user.nl # PGadmin service
  PGADMIN_PASSWORD: the_pw  # PGadmin service

```

### Create Ansible Vault Password

* create strong password  
* store in `~/.ssh/ansible-vault/map5topo.txt` for convenience



### Encrypt Ansible Files

VERY IMPORTANT. UNENCRYPTED FILES SHOULD NEVER BE CHECKED IN!!!

Using `ansible-vault` with password encrypt these:

``` {.bash linenums="1"}
ansible-vault encrypt --vault-password-file ~/.ssh/ansible-vault/map5topo.txt vars/vars.yml
ansible-vault encrypt --vault-password-file ~/.ssh/ansible-vault/map5topo.txt vars/gh-key.rsa
ansible-vault encrypt --vault-password-file ~/.ssh/ansible-vault/map5topo.txt vars/gh-key.rsa.pub 
ansible-vault encrypt --vault-password-file ~/.ssh/ansible-vault/map5topo.txt vars/authorized_keys 

```

 
### Disable GitHub Workflows

We do not want that workflows take effect immediately. 
So disable them temporary by renaming the dir.

``` {.bash linenums="1"}
cd git/.github/workflows
git mv workflows workflows.not
git add .
git commit -m "disable workflows"
git push

```


## 5 Bootstrap/provision Server

Moment of truth! Bootstrap (provision the server) in single playbook. Save the logfile for analysis.

* `ansible-playbook -v --vault-password-file ~/.ssh/ansible-vault/map5topo.txt bootstrap.yml -i hosts/prod.yml > bootstrap.log 2>&1`

If all goes well, this output should be shown at end:

```
PLAY RECAP ***********************************************************************************************************
apisandbox                 : ok=58   changed=22   unreachable=0    failed=0    skipped=8    rescued=0    ignored=0   

```

Observe output for errors (better is to save output in file via `.. > bootstrap.log 2>&1`).

In cases of errors and after fixes, simply rerun the above Playbook.

Site should be running at: [https://topo.map5.nl](https://topo.map5.nl)
Check with portainer [https://topo.map5.nl/portainer/](https://topo.map5.nl/portainer/).

## 6 Docker storage in attached Volume

See https://www.guguweb.com/2019/02/07/how-to-move-docker-data-directory-to-another-location-on-ubuntu/

``` {.bash linenums="1"}
service docker stop
mkdir -p /var/map5/docker

# Add this file
vi /etc/docker/daemon.json
# add this content
{
  "data-root": "/var/map5/docker"
}

# copy current content
rsync -aP /var/lib/docker/ /var/map5/docker

# move old dir just in case
mv /var/lib/docker /var/lib/docker.old

service docker start

# remove the old
rm -rf /var/lib/docker.old

```

## 6a Disk Resizing

``` {.bash linenums="1"}
cd git/services
./stop.sh

# as root sudo -
service docker stop
umount /var/map5

# enlarge Volume size in Hetzner Console

# figure out which disk
fdisk -l
resize2fs /dev/sdc

mount /var/map5
service docker stop
cd git/services
./start.sh


```

## 7 Resolve Issues

These are typical issues found and resolved:

* make sure the `gh-key.rsa.pub` is present in both `/root` and `/home/<admin user>` `.ssh/authorized_keys`
* `postgis + pgadmin`: needed to manually `stop.sh`, remove all volumes and `start.sh` otherwise could not login on pgadmin nor postgis from there  

## 8. Enable GitHub Workflows

Enable by renaming:

``` {.bash linenums="1"}
git mv workflows.not workflows 
git add .
git commit -m "enable workflows"
git push

```

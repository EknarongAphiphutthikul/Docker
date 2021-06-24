# Docker

## Prepare System
- Multiple VM Ubuntu version 20.04 on Hyper-V   [set up hyper-v on windows10](https://github.com/EknarongAphiphutthikul/Hyper-V)
- DNS Server  [set up](https://github.com/EknarongAphiphutthikul/Dns-bind9)
- Update Package On Ubuntu 20.04
  ```sh
  sudo apt-get update
  ```
- Show hostname
  ```sh
  hostnamectl
  ```
- Set hostname
  ```sh
  sudo hostnamectl set-hostname docker.ake.com
  ```
- Show ip
  ```sh
  ifconfig
  ```
- Set ipv4 internal network (vEthernet-Internal-ME)
  - On cloud : you'll need to disable.
    ```sh
    sudo nano /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
    ```
    ```console
    network: {config: disabled}
    ```
  - Show file config in netplan
    ```sh
    ls /etc/netplan/
    ```
    ```console
    00-installer-config.yaml
    ```
  - Edit file config in netplan
    ```sh
    sudo nano /etc/netplan/00-installer-config.yaml
    ```
    ```console
    network:
      ethernets:
        eth0:
          dhcp4: false
          addresses:
            -  169.254.19.102/16
          nameservers:
            search: [ ake.com ]
            addresses:
              - 169.254.19.105
        eth1:
          dhcp4: true
      version: 2
    ```
  - Apply Config
    ```sh
    sudo netplan apply
    ```

- Set DNS (change default 127.0.0.53 to 169.254.19.105)  
  > **Important** : Workaround for  [Bug #1624320](https://bugs.launchpad.net/ubuntu/+source/systemd/+bug/1624320)
  ```sh
  sudo rm -f /etc/resolv.conf
  sudo ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf
  sudo reboot
  ```
----

<br/>

## Install Docker

- Install the required dependencies
  ```sh
  sudo apt-get install apt-transport-https ca-certificates curl software-properties-common curl -y
  ```
- Import the Docker GPG key
  ```sh
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
  ```
- Add the Docker CE official repository to the APT source file
  ```sh
  sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
  ```
- Update the repository cache
  ```sh
  sudo apt-get update -y
  ```
- Install
  ```sh
  sudo apt-get install docker-ce -y
  ```
- Verify the installed version of Docker CE
  ```sh
  docker --version
  ```
- Add Group user
  ```sh
  sudo groupadd docker

  sudo usermod -aG docker $USER

  newgrp docker
  ```
----

<br/>

## How to Connect to Private Docker Registry   
> **Note** How to install [private docker registry](https://github.com/EknarongAphiphutthikul/Docker-Registry)   

- If need certificate
  ```sh
  copy cert to /usr/local/share/ca-certificates/<domain-name>.crt

  sudo update-ca-certificates

  sudo service docker restart
  ```
  ```console
  akeadm@docker:~$ sudo cp ca-chain.cert.pem /usr/local/share/ca-certificates/registry.ake.com.crt
  [sudo] password for akeadm:
  akeadm@docker:~$ ls /usr/local/share/ca-certificates/
  registry.ake.com.crt
  akeadm@docker:~$ sudo update-ca-certificates
  Updating certificates in /etc/ssl/certs...
  rehash: warning: skipping registry.ake.com.pem,it does not contain exactly one c ertificate or CRL
  1 added, 0 removed; done.
  Running hooks in /etc/ca-certificates/update.d...
  done.
  akeadm@docker:~$ sudo service docker restart
  ```

- If need authentication (username, password)
  ```sh
  docker login <domain-name>:<port>
  ```
  ```console
  akeadm@docker:~$ docker login registry.ake.com:443
  Username: registryusr
  Password:

  Login Succeeded
  ```


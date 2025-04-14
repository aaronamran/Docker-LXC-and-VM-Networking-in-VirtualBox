# Docker, LXC and VM Networking in VirtualBox

This write-up documents a practical project that uses VirtualBox alongside Linux container technologies (Docker and LXC/LXD) to simulate and explore the core concepts provided by platforms like Proxmox. A lightweight Lubuntu VM is used for this setup. Note that the initial setup of the Linux VM in VirtualBox is not included, as it was completed beforehand.

1. [Setting Up Docker for Containerised Services]()
2. [Exploring LXC/LXD for Lightweight Virtualisation]()
3. [Container Snapshots for LXD]()


## [Setting Up Docker for Containerised Services]

### Installing Docker
- The first thing to do is to update the package index
  ```
  sudo apt update
  ```
- Then install necessary packages
  ```
  sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release -y
  ```
- Add Docker's official GPG key and repository
  ```
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
  
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  ```
- Install Docker Engine
  ```
  sudo apt update
  sudo apt install docker-ce docker-ce-cli containerd.io -y
  ```
- An optional step is to add your user to the Docker group to run commands without sudo. Log out and back in or reboot to apply the changes
  ```
  sudo usermod -aG docker $USER
  ```

### Running Containerised Services
- Pull a Sample Image and run a container
  ```
  docker pull hello-world
  docker run hello-world
  ```
- Example - Run a Web Server Container
  ```
  docker pull nginx
  docker run -d -p 8080:80 --name webserver nginx
  ```
- To verify the operation, Open a web browser on your host system and navigate to `http://<VM_IP>:8080` if using bridged networking or use port forwarding for NAT

## Exploring LXC/LXD for Lightweight Virtualisation
### Install LXD
- Install LXD (on Ubuntu/Debian)
  ```
  sudo apt install lxd -y
  ```
  If the command above gives a warning about Snap, then use
  ```
  sudo snap install lxd
  ```
  Check if LXD is installed as a snap with
  ```
  which lxd
  ```
  If it says `/snap/bin/lxd`, then all is good
  
- Initialise LXD
  ```
  sudo lxd init
  ```
- An optional step is to run LXC commands without sudo
  ```
  sudo usermod -aG lxd $USER
  ```
  Then log out and back in or reboot to apply the group change

### Launching and Managing Containers
- To launch a container, use
  ```
  lxc launch ubuntu:20.04 mycontainer
  ```

- List running containers
  ```
  lxc list
  ```

- To access container shell, use
  ```
  lxc exec mycontainer -- /bin/bash
  ```

- To stop and delete containers when not needed, use
  ```
  lxc stop mycontainer
  lxc delete mycontainer
  ```

## Container Snapshots for LXD
- Create container snapshots with
  ```
  lxc snapshot mycontainer snap1
  ```

- Restore snapshots if you run into issues
  ```
  lxc restore mycontainer snap1
  ```






# Docker, LXC and VM Networking in VirtualBox

This write-up documents a practical project that uses VirtualBox alongside Linux container technologies (Docker and LXC/LXD) to simulate and explore the core concepts provided by platforms like Proxmox. A lightweight Lubuntu VM is used for this setup. This project uses LXD, a system container manager built on top of LXC, to demonstrate lightweight virtualization. Note that the initial setup of the Linux VM in VirtualBox is not included, as it was completed beforehand.

![docker_lxc_vm_networking_diagram](https://github.com/user-attachments/assets/cc0aabdb-06f1-482b-83b1-67ac211862b9)


1. [Setting Up Docker for Containerised Services](https://github.com/aaronamran/Docker-LXC-and-VM-Networking-in-VirtualBox/blob/main/README.md#setting-up-docker-for-containerised-services)
2. [Exploring LXC/LXD for Lightweight Virtualisation](https://github.com/aaronamran/Docker-LXC-and-VM-Networking-in-VirtualBox/blob/main/README.md#exploring-lxclxd-for-lightweight-virtualisation)
3. [VM Networking in VirtualBox](https://github.com/aaronamran/Docker-LXC-and-VM-Networking-in-VirtualBox/blob/main/README.md#vm-networking-in-virtualbox)


## Setting Up Docker for Containerised Services

### Installing Docker
- The first thing to do is to update the package index
  ```
  sudo apt update && sudo apt upgrade -y
  ```
- Then install necessary packages
  ```
  sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release -y
  ```
  ![image](https://github.com/user-attachments/assets/faf722bc-8755-4581-8d58-c94ca4d5016f)

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
  ![image](https://github.com/user-attachments/assets/7d9084ae-e4e5-4acc-9c32-385e500b6365)
  <br />
  If error messages appear, run the command below to check the Ubuntu codename
  ```
  lsb_release -cs
  ```
  At this point in time, we are currently on Ubuntu 24.04 "Noble Numbat", which was just released, and Docker hasn’t added official support for it yet in their APT repository. Hence the displayed error messages

- The solution is to use jammy (Ubuntu 22.04) repo instead — it's stable and fully supported by Docker. Remove the invalid (noble) Docker repo
  ```
  sudo rm /etc/apt/sources.list.d/docker.list
  ```
  Then re-add Docker's jammy repo manually
  ```
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
  https://download.docker.com/linux/ubuntu jammy stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  ```
  Update the package index again and install Docker
  ```
  sudo apt install docker-ce docker-ce-cli containerd.io -y
  ```
  When it is successful, it will install normally as shown below <br />
  ![image](https://github.com/user-attachments/assets/dbb021f4-4bbd-4715-bf28-5c548de22b96)


- An optional step is to add your user to the Docker group to run commands without sudo. Log out and back in or reboot to apply the changes
  ```
  sudo usermod -aG docker $USER
  ```

### Running Containerised Services
- Pull a sample image and run a container
  ```
  docker pull hello-world
  docker run hello-world
  ```
  ![image](https://github.com/user-attachments/assets/0b5658b4-9477-4360-9cd5-f94f3aea042e)

- For testing service accessibility, Nginx was deployed as a Docker container rather than installed natively on the VM. Here is the example to run a web server container
  ```
  docker pull nginx
  docker run -d -p 8080:80 --name webserver nginx
  ```
  ![image](https://github.com/user-attachments/assets/ee6ad309-0946-4eca-ba01-b1be112252b0)

- To verify the operation, open a web browser on your host system and navigate to `http://<VM_IP>:8080` if using bridged networking or use port forwarding for NAT <br />
  ![image](https://github.com/user-attachments/assets/7bfe27e5-6d1c-46f2-b1c5-fc4850c484e0)


## Exploring LXC/LXD for Lightweight Virtualisation
### Install LXD
- Install LXD (on Ubuntu/Debian)
  ```
  sudo apt install lxd -y
  ```
  ![image](https://github.com/user-attachments/assets/9e467684-1fb7-4b78-b6bf-c53c974d12a6) 
  <br />
  If the command above gives a warning about Snap, then use
  ```
  sudo snap install lxd
  ```
  Check if LXD is installed as a snap with
  ```
  which lxd
  ```
  ![image](https://github.com/user-attachments/assets/8ed5f8c3-5794-49dc-8b4e-f173efbac6f4)
  <br />
  If it says `/snap/bin/lxd`, then all is good
  
- Initialise LXD
  ```
  sudo lxd init --auto
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
  ![image](https://github.com/user-attachments/assets/f34b80d7-1fb9-4569-b3b1-093f062312b7)

- To access container shell, use
  ```
  lxc exec mycontainer -- /bin/bash
  ```
  ![image](https://github.com/user-attachments/assets/ac147f1e-6979-4f44-93e0-e796b52a2800)

- To stop and delete containers when not needed, use
  ```
  lxc stop mycontainer
  lxc delete mycontainer
  ```

### Container Snapshots for LXD
- Create container snapshots with
  ```
  lxc snapshot mycontainer snap1
  ```

- Restore snapshots if you run into issues
  ```
  lxc restore mycontainer snap1
  ```
  ![image](https://github.com/user-attachments/assets/db41f319-e94b-414f-aba8-b9b5ea30fd6c)


## VM Networking in VirtualBox
To simulate real-world infrastructure and container networking (like Proxmox setups), understanding how your Lubuntu VM is networked inside VirtualBox is key
<br />
1. Bridged Adapter
- Acts as if the VM is just another machine on your LAN
- Gets its own IP address from your network's DHCP server
- Accessible from other devices on the network
- Recommended for container scenarios where host and VM need to communicate seamlessly

2. NAT (Network Address Translation)
- VM shares the host’s IP and uses VirtualBox as a gateway
- Good for quick setups where internet access is needed but external access to the VM isn't
- Requires port forwarding to access container services from the host

3. Host-only Adapter
- Creates a virtual network between the host and VM only
- VM has no internet unless combined with another adapter
- Great for isolated labs and multi-VM setups where external access isn’t needed

For this project, the Bridged Adapter mode is used. It allows the Lubuntu VM to get its own IP on the network, making it easy to:
- Access containerized services (like Nginx) from the host without configuring port forwarding
- Simulate a more realistic server environment
- Test inbound connectivity as if the VM were a standalone physical machine

### Checking the VM's IP Address
- Run the following in Lubuntu VM
  ```
  ip a
  ```
- A more concise version is
  ```
  ip -4 addr show | grep inet
  ```

### Verifying Network Connectivity
- Once Docker or LXC containers are running, test the container's outbound network access
  ```
  lxc exec mycontainer -- ping -c 4 8.8.8.8
  ```
  ![image](https://github.com/user-attachments/assets/80526a43-ea4e-4770-8ede-4d9827cbe3fe)
  <br />
  If the container has no internet access and displays 100% packet loss, it usually points to an issue with the LXD network bridge (lxdbr0) not being configured properly or not NAT'ing traffic to the outside world
  <br />
  To check for the bridge settings, run
  ```
  lxc network list
  ```
  Look for lxdbr0 — its "MANAGED" column should say "YES", and "TYPE" should be "bridge" <br />
  ![image](https://github.com/user-attachments/assets/215ccddc-1f60-431c-a097-9a86438d0afb)
  
  


  An alternative is to use
  ```
  lxc exec mycontainer -- curl http://google.com
  ```
- Test if a service in the container is accessible from the host. We will need to run a simple HTTP server in the container
  ```
  lxc exec mycontainer -- bash
  ```
  Then install Python if not available and start the HTTP server
  ```
  apt update && apt install -y python3
  cd /tmp
  python3 -m http.server 8080
  ```
  This will serve files from /tmp on port 8080. From the host system, run
  ```
  curl http://<Container_IP_Address>:8080
  ```


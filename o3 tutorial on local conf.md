Below is a complete tutorial in Markdown format. You can copy–paste this file into GitHub (or any Markdown editor) and it will serve as a comprehensive guide for setting up DevStack on Ubuntu 22.04 in VirtualBox on Windows 11.

---

```markdown
# OpenStack DevStack Installation on Ubuntu 22.04 in VirtualBox (Windows 11)

This tutorial walks you through setting up a single-node OpenStack environment using DevStack on an Ubuntu 22.04 VM running in VirtualBox on a Windows 11 host. We cover VirtualBox network configuration (NAT + Host-Only), preparing Ubuntu, setting up SSH, configuring static IP via netplan, creating a dedicated DevStack user, and finally installing and configuring DevStack.

---

## 1. VirtualBox Network Configuration

To allow both internet connectivity and SSH/local access, configure two network adapters for your VM:

### Adapter 1: NAT
- **Purpose:** Provides the VM with internet access.
- **Settings:**  
  - Attached to: **NAT**  
  - Promiscuous Mode: **Deny** (default)  
  - Cable Connected: **Checked**

### Adapter 2: Host-Only
- **Purpose:** Provides a private network between your Windows host and the Ubuntu VM.
- **Settings:**  
  - Attached to: **Host-Only Adapter**  
  - Choose the host-only network (e.g., `vboxnet0` or "VirtualBox Host-Only Ethernet Adapter")  
  - Promiscuous Mode: **Deny**  
  - Cable Connected: **Checked**

**Create/Verify Host-Only Network:**
1. In VirtualBox Manager, go to **File > Host Network Manager**.
2. If none exists, click **Create** to add one.  
   - Default settings: Host IP `192.168.56.1/24` with DHCP enabled.
3. (Optional) You can disable DHCP if you prefer to configure a static IP on the VM.

---

## 2. Preparing the Ubuntu 22.04 VM

### A. Boot and Verify IP Addresses

1. Boot your Ubuntu VM.
2. Open a terminal and run:
   ```bash
   ip a
   ```
   **Expected Output:**
   - **NAT Interface** (e.g., `enp0s3`): IP like `10.0.2.15`
   - **Host-Only Interface** (e.g., `enp0s8`): IP like `192.168.56.x` (we will set this to a static IP)

### B. Configure a Static IP for Host-Only

We'll configure `enp0s8` to have a static IP (e.g., `192.168.56.10`).

1. **Edit Netplan Configuration:**
   On Ubuntu 22.04 Desktop, the file is usually `/etc/netplan/01-network-manager-all.yaml`:
   ```bash
   sudo nano /etc/netplan/01-network-manager-all.yaml
   ```
2. **Modify the file** so it contains the following (note: we're using `NetworkManager` as the renderer):
   ```yaml
   # Let NetworkManager manage all devices on this system
   network:
     version: 2
     renderer: NetworkManager
     ethernets:
       enp0s3:    # NAT adapter; keep DHCP for internet access
         dhcp4: true
       enp0s8:    # Host-only adapter; set a static IP
         addresses: [192.168.56.10/24]
         # Do not set a gateway here; let the NAT interface handle the default route
         nameservers:
           addresses: [8.8.8.8, 1.1.1.1]
   ```
3. **Fix file permissions** (if you see warnings about "too open"):
   ```bash
   sudo chmod 600 /etc/netplan/01-network-manager-all.yaml
   ```
4. **Apply the configuration:**
   ```bash
   sudo netplan apply
   ```
5. **Verify:**
   ```bash
   ip a
   ```
   You should see `enp0s8` with IP `192.168.56.10`.

### C. Test Connectivity from Windows

From your Windows host, open Command Prompt or PowerShell and run:
```powershell
ping 192.168.56.10
```
You should receive replies.

### D. Install and Enable SSH Server

1. **Update packages:**
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
2. **Install OpenSSH server:**
   ```bash
   sudo apt install -y openssh-server
   ```
3. **Check status:**
   ```bash
   sudo systemctl status ssh
   ```
4. **(Optional) Allow SSH through the firewall:**
   ```bash
   sudo ufw allow ssh
   ```
5. **Test SSH from Windows:**
   ```powershell
   ssh <your_username>@192.168.56.10
   ```
   Replace `<your_username>` with your Ubuntu username. You should be prompted to accept the host key and then enter your password.

---

## 3. Preparing for DevStack Installation

DevStack must not be run as root, so we’ll create a dedicated `stack` user.

### A. Create the `stack` User

Run these commands from your current Ubuntu user (you may need to use `sudo`):

```bash
sudo useradd -s /bin/bash -d /opt/stack -m stack
sudo passwd -d stack        # Remove password (optional; allows passwordless sudo)
sudo usermod -aG sudo stack
echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
```

### B. Switch to the `stack` User

```bash
sudo su - stack
```

*You should now see your prompt change (e.g., `stack@user:~$`).*

### C. Disable the Firewall (if not already done)

```bash
sudo ufw disable
```

### D. (Optional) Create a Swap File

If you have limited RAM, creating a swap file can help prevent out-of-memory issues. Check if swap is active:

```bash
sudo swapon --show
```

If you see `/swapfile` and it's active, you're set. If not, run:

```bash
sudo swapoff /swapfile   # In case it's in a bad state
sudo rm /swapfile
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

---

## 4. Configuring the `/etc/hosts` File

Your `/etc/hosts` file might look like this:

```
127.0.0.1       localhost
127.0.1.1       user

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

- **Note:** If your primary username is `user` and that’s acceptable, this file is fine. No additional changes are required for DevStack.

---

## 5. Installing DevStack

### A. Clone the DevStack Repository

Make sure you're logged in as the `stack` user, then run:

```bash
git clone https://opendev.org/openstack/devstack
cd devstack
```

### B. Create and Configure `local.conf`

Create a file named `local.conf` in the `devstack` directory:

```bash
nano local.conf
```

Paste the following configuration (tailored to your network):

```ini
[[local|localrc]]
# Host network settings
HOST_IP=192.168.56.10            # This is your VM's host-only static IP (accessible from Windows)
SERVICE_HOST=$HOST_IP
MYSQL_HOST=$HOST_IP
RABBIT_HOST=$HOST_IP
GLANCE_HOSTPORT=$HOST_IP:9292

# Credentials
ADMIN_PASSWORD=admin 
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD

# Networking settings for Neutron
Q_USE_SECGROUP=True
PUBLIC_INTERFACE=enp0s8         # The host-only interface on your VM (verify with: ip addr)
FLOATING_RANGE="192.168.56.0/24"  # The subnet of your host-only network
PUBLIC_NETWORK_GATEWAY="192.168.56.1"  # The Windows host's IP on the host-only network
Q_FLOATING_ALLOCATION_POOL=start=192.168.56.100,end=192.168.56.200  # Range for floating IPs
IPV4_ADDRS_SAFE_TO_USE="10.0.0.0/22"      # Private network range for instance fixed IPs

# Enable provider networking: Bridge the VM's external network to your host-only network
Q_USE_PROVIDERNET_FOR_PUBLIC=True
OVS_PHYSICAL_BRIDGE=br-ex
PUBLIC_BRIDGE=br-ex
OVS_BRIDGE_MAPPINGS=public:br-ex
```

Save and exit (Ctrl+O, Enter, then Ctrl+X).

**Explanation of Key Settings:**
- **HOST_IP & Service Settings:** Ensure all OpenStack components bind to `192.168.56.10`.
- **Credentials:** Set to `admin` for simplicity (change as needed).
- **Neutron Networking:**  
  - `PUBLIC_INTERFACE` tells DevStack to use `enp0s8` for floating IPs.
  - `FLOATING_RANGE` and `Q_FLOATING_ALLOCATION_POOL` define the available floating IPs (which you can assign to instances for SSH access from Windows).
  - `PUBLIC_NETWORK_GATEWAY` is set to the Windows host’s IP on the host-only network.
- **Provider Networking:** Configures Open vSwitch to bridge the external network (br-ex) to your host-only network.

### C. Start the DevStack Installation

Run:

```bash
./stack.sh
```

This process may take 10–20 minutes or longer, as it installs and configures all OpenStack components.

---

## 6. Verifying the DevStack Installation

### A. Check OpenStack Services

After installation completes, load the OpenStack environment:

```bash
source openrc admin admin
```

List the services:

```bash
openstack service list
```

### B. Access the Horizon Dashboard

On your Windows host, open a browser and go to:

```
http://192.168.56.10/dashboard
```

Log in with:
- **Username:** admin  
- **Password:** admin

### C. SSH Access to Instances

Once you launch an instance within OpenStack, allocate it a floating IP from the range `192.168.56.100-192.168.56.200`. You can then SSH into that instance from your Windows host:

```powershell
ssh <instance_user>@<floating_ip>
```

Make sure the instance’s security group allows SSH (port 22).

---

## 7. Summary and Next Steps

1. **VirtualBox Network Configuration:**  
   - NAT for internet and Host-Only for SSH/local access.
2. **Ubuntu VM Preparation:**  
   - Configure a static IP on the host-only interface (192.168.56.10).
   - Install and enable OpenSSH.
3. **Pre-requisites for DevStack:**  
   - Update system, create the `stack` user, disable firewall, (optionally) add swap.
4. **DevStack Installation:**  
   - Clone DevStack, configure `local.conf` with your network settings, and run `./stack.sh`.
5. **Verification:**  
   - Access the Horizon dashboard and SSH into instances (via floating IPs).

This tutorial provides all the steps and commands required to set up your OpenStack DevStack environment. Save this file in your GitHub repository as a reference or for others to use.

Happy deploying!
```

---

You now have a complete, detailed Markdown tutorial that covers every step we discussed. Simply copy this file and paste it into your GitHub repository for future reference.

### **Guide: Assigning a Static IP in VirtualBox for Ubuntu VM**  
📌 **Purpose:** Configure a **Host-Only Network** with a **static IP** to ensure stable internal networking for OpenStack.

---

### **Step 1: Configure Host-Only Network in VirtualBox**
1. Open **VirtualBox**.
2. Go to **File > Host Network Manager**.
3. Create or select an existing **Host-Only Network**.
4. Configure it as follows:
   - **IPv4 Address**: `192.168.56.1`
   - **IPv4 Network Mask**: `255.255.255.0`
   - **Disable DHCP** (to prevent IP conflicts).
5. Click **Apply** and **Close**.

---

### **Step 2: Attach Host-Only Network to the VM**
1. Open **VirtualBox**.
2. Select your **Ubuntu VM** > Click **Settings**.
3. Go to **Network > Adapter 2** (Leave Adapter 1 as NAT for internet access).
4. Enable Adapter and set:
   - **Attached to**: `Host-Only Adapter`
   - **Name**: Select the Host-Only network (e.g., `vboxnet0`).
5. Click **OK**.

---

### **Step 3: Assign a Static IP in Ubuntu VM**
1. Start the VM and log in.
2. Open a **terminal** and edit Netplan configuration:
   ```bash
   sudo nano /etc/netplan/00-installer-config.yaml
   ```
3. Add the following configuration (**proper indentation is required**):
   ```yaml
   network:
     ethernets:
       enp0s8:
         addresses:
           - 192.168.56.100/24
         routes:
           - to: default
             via: 192.168.56.1
         nameservers:
           addresses:
             - 8.8.8.8
     version: 2
   ```
   - **Replace `enp0s8`** with your actual **Host-Only adapter name** (`ip a` command can help verify).
   - Use `8.8.8.8` as DNS or any preferred DNS.

4. **Save and exit** (`Ctrl + X`, then `Y`, then `Enter`).

---

### **Step 4: Apply Changes**
1. Correct permissions for Netplan files:
   ```bash
   sudo chmod 600 /etc/netplan/*.yaml
   ```
2. Apply the Netplan configuration:
   ```bash
   sudo netplan apply
   ```
3. Verify the IP address:
   ```bash
   ip a
   ```
   You should see `192.168.56.100` assigned to `enp0s8`.

---

### **Step 5: Test Network Connectivity**
✅ **Test connection to the host machine**:
```bash
ping -c 4 192.168.56.1
```
✅ **Test internet access (if NAT is enabled on Adapter 1)**:
```bash
ping -c 4 8.8.8.8
```

---

### **Step 6: Ensure Static IP Persists After Reboot**
Your static IP **will persist** across reboots because it is stored in **Netplan**.  
To confirm, restart the VM:
```bash
sudo reboot
```
After reboot, run:
```bash
ip a
```
If `192.168.56.100` is still there, your configuration is persistent.

---

### **Common Issues & Fixes**
#### 🚨 *Issue: `gateway4 has been deprecated` Warning*  
✅ **Fix:** Replace `gateway4` with the `routes` section in Netplan.

#### 🚨 *Issue: IP Not Assigned After Reboot*  
✅ **Fix:** Ensure Netplan configuration is applied:
```bash
sudo netplan apply
```
✅ **Fix:** Restart networking:
```bash
sudo systemctl restart systemd-networkd
```

#### 🚨 *Issue: No Internet Access in VM*  
✅ **Fix:** Ensure **Adapter 1 (NAT) is enabled** in VirtualBox settings.

---

### **Conclusion**
- Static IP `192.168.56.100` is assigned successfully to `enp0s8`.
- The configuration **remains persistent** after reboot.
- You can now use this stable network for **OpenStack setup**.

---


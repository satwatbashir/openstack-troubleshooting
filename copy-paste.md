**Troubleshooting Guide: Copy-Paste Not Working in VirtualBox (Ubuntu VM)**

### **Issue:**
Copy-paste between the **Windows host** and **Ubuntu VM** in **VirtualBox** is not working, even after enabling bidirectional clipboard.

---

## **Step 1: Ensure Bidirectional Clipboard is Enabled**
1. Open **VirtualBox Manager**.
2. Select your Ubuntu VM (do not start it yet).
3. Click **Settings → General → Advanced**.
4. Ensure **Shared Clipboard** is set to **Bidirectional**.
5. Set **Drag and Drop** to **Bidirectional** (optional but useful).
6. Click **OK**.

---

## **Step 2: Install VirtualBox Guest Additions**
Guest Additions must be installed for clipboard sharing to work.

### **1️⃣ Insert Guest Additions CD**
1. Start the **Ubuntu VM**.
2. From the **VirtualBox menu**, go to:
   **Devices → Insert Guest Additions CD Image…**
3. Open **Terminal** in Ubuntu and run:
   ```bash
   sudo mount /dev/cdrom /mnt
   sudo /mnt/VBoxLinuxAdditions.run
   ```
   
4. If prompted about missing dependencies (e.g., `gcc`, `make`, `perl`), install them first:
   ```bash
   sudo apt update
   sudo apt install gcc make perl linux-headers-$(uname -r) dkms -y
   ```
5. Then **rerun** the Guest Additions installation:
   ```bash
   sudo /mnt/VBoxLinuxAdditions.run
   ```

---

## **Step 3: Reboot the VM**
After installing Guest Additions, restart your VM:
```bash
sudo reboot
```

---

## **Step 4: Verify Clipboard Functionality**
Once the system reboots, test **copy-paste** between Windows and Ubuntu.

If it **still doesn’t work**, try:
```bash
VBoxClient --clipboard
```
If clipboard starts working but stops after reboot, add it to startup:
```bash
echo "VBoxClient --clipboard" >> ~/.profile
```

---

## **Step 5: Restart VirtualBox Services (If Clipboard Still Fails)**
If clipboard sharing still does not work:
```bash
sudo systemctl restart vboxadd
sudo systemctl restart vboxadd-service
```
Manually restart clipboard services:
```bash
killall VBoxClient
VBoxClient --clipboard
VBoxClient --draganddrop
```

---

## **Step 6: Reinstall Guest Additions (If Needed)**
If nothing works, **uninstall and reinstall** Guest Additions:
```bash
sudo apt-get remove virtualbox-guest-utils virtualbox-guest-x11 virtualbox-guest-dkms -y
sudo apt-get update
sudo apt-get install virtualbox-guest-utils virtualbox-guest-x11 virtualbox-guest-dkms -y
sudo reboot
```

### **Final Check:**
- Ensure **Bidirectional Clipboard** is enabled.
- Ensure **Guest Additions** is installed correctly.
- Restart VirtualBox services if needed.

This guide ensures clipboard sharing works smoothly between **Windows host** and **Ubuntu VM** in VirtualBox.


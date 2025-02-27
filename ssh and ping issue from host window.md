# How to Remove SSH Keys and Reset MAC Address in Windows

## 1. Removing Old SSH Keys
If you've previously connected to a server using SSH and the IP or hostname changes, Windows might store an outdated key, preventing new connections.

### **Steps to Remove Old SSH Keys:**
1. Open **Command Prompt (cmd)** or **PowerShell**.
2. Run the following command to remove the old SSH key:
   ```sh
   ssh-keygen -R <IP-ADDRESS>
   ```
   Example:
   ```sh
   ssh-keygen -R 192.168.56.10
   ```
3. Try reconnecting to the server:
   ```sh
   ssh user@192.168.56.10
   ```
4. If prompted with **"Are you sure you want to continue connecting (yes/no)?"**, type `yes` and press **Enter**.

## 2. Clearing ARP Cache to Remove Stale MAC Address
If you've deleted and recreated a VM or network device, Windows might still hold onto the old MAC address, causing connectivity issues.

### **Steps to Remove the MAC Address Cache:**
1. Open **Command Prompt** as Administrator.
2. Run the following command to delete all cached ARP entries:
   ```sh
   arp -d *
   ```

   ```
4. Try pinging the VM again to confirm the issue is resolved:
   ```sh
   ping 192.168.56.10
   ```


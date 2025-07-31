### **Setting Up Kali Linux**

1. **Download the ISO**  
   Grab the Kali Linux ISO from the official site:  
   [https://www.kali.org/get-kali/#kali-platforms](https://www.kali.org/get-kali/#kali-platforms)

2. **Create a New VM**  
   - In VirtualBox, create a new machine for Kali.  
   - Name it whatever you like, but don’t attach an image yet.  
   - Set **Type** to `Linux` and **Version** to `Linux 2.6`.

3. **Allocate Resources**  
   - **Memory**: I assigned 4 GB (4096 MB).  
   - **CPU**: 1 core.  
   - **Disk Size**: 30 GB (you can adjust based on your storage).  

4. **Adjust System Settings**  
   - Go to **Settings → System**:  
     - Uncheck **Floppy**.  
     - Move **Hard Disk** to the top and **Optical** second.  
   - Under **Storage**, click on **Empty** and attach the Kali Linux ISO you downloaded earlier.  

5. **Network Configuration**  
   - Set **Adapter 1** to `Internal Network`.  
   - Adapter Type: **Paravirtualized Network (virtio-net)** for better performance.  

**Note:** *Internal Network creates an isolated network segment inside VirtualBox that only VMs in the same internal network can access.*

---

Now start both pfSense and Kali VMs.
<img width="640" height="480" alt="kali install" src="https://github.com/user-attachments/assets/e01e5e75-c3e7-4226-a9d7-530cd902e6d6" />

### **Installing Kali Linux**

- Start the VM and choose **Graphical Install**.
- The installation process is straightforward, just follow the prompts, set your language, username, and password, and log in once it’s done.

**Note:** Nothing tricky here, just make sure you remember your credentials for later.

---

### **First Steps After Installation**

Once you’re inside Kali:

1. Open a terminal and update the system:
```bash
sudo apt update && sudo apt full-upgrade
```
2. Next, log in to the **pfSense web interface** from Firefox on Kali.  
   - Use the **LAN IP** (in my case: `https://10.0.0.1`).  
   - Make sure you include `https` in the address.
<img width="1297" height="706" alt="pfsenselogin" src="https://github.com/user-attachments/assets/7b4c5076-c9ae-4694-ae72-9e4ee1706e65" />

### **Logging into pfSense & Initial Configuration**

Once you're inside Firefox on Kali, go to:

`https://10.0.0.1`

- The default pfSense login credentials are:
    - **Username:** `admin`
    - **Password:** `pfsense`

---

### **Running Through the Setup Wizard**

After logging in, pfSense will walk you through a setup wizard. Here's what I did at each step:

- **Step 1 & 2**: Just hit **Next,** no need to change anything here.
- **Step 3**:
    - Uncheck **“Override DNS”**.
    - This gives you full control over which DNS servers pfSense uses, which is especially useful for labs or tighter security setups.
    - I didn’t make any other changes, so I just hit **Next** again.
- **Step 5 (WAN Configuration)**:
    - Make sure to **uncheck** the option that says **“Block private networks from entering via WAN.”**
    - Since our WAN in this lab is actually connected to VirtualBox’s NAT (not a real ISP), it uses a **private IP,**in my case, `10.0.2.15`.
    - If you leave that box checked, pfSense will block all WAN traffic, which means **no internet in the lab.**

💡 *In this setup, Kali is connected to the internal network (LAN/OPT), not the WAN, so this won’t affect traffic between Kali and pfSense directly,but unchecking it keeps your lab flexible.*

- **Step 6**: Choose your own admin password.
- **Step 7**: Click **Reload** to apply everything.
- Then just hit **Finish**.<img width="1297" height="706" alt="dashboard after config" src="https://github.com/user-attachments/assets/18a9e324-eff3-4a92-8a8d-0be9a468d0fc" />

Once the wizard is complete and you’re at the dashboard, let’s make a few adjustments:

### Rename Interfaces (Optional)
- Go to **Interfaces → OPT1**.
- You can rename it if you want, but I left mine as OPT1.
- Click **Save**, then **Apply Changes**.
- Repeat the same for OPT2 if you’d like.

### Enable DNS Resolver Options
- Navigate to **Services → DNS Resolver**.
- Scroll down to **DHCP Registration** and **Static DHCP**.
- Check both boxes. This makes sure hostnames from DHCP leases show up in the DNS Resolver, which is super handy in labs.
<img width="1297" height="706" alt="dhcpresolverconfig" src="https://github.com/user-attachments/assets/a4063d2d-1665-4fc8-8432-9dc49c97ee6a" />

### Why We Enabled Those DNS Options

- **DHCP Registration**: This lets pfSense automatically track which devices have which IPs and hostnames. The benefit? You can use device names instead of memorizing IP addresses.
- **Static DHCP**: Ensures certain devices always get the same IP address, which is important for consistency in a lab setup.

After checking both, click **Save** and **Apply Changes**.

---

### Advanced DNS Settings

Next, go to **Advanced Settings** under the DNS Resolver page.

- In **Advanced Resolver Options**, enable:
    - **Prefetch Support**
    - **Prefetch DNS Key Support**

- Hit **Save** and **Apply Changes**.

These options help speed up DNS lookups and improve caching efficiency.
<img width="1297" height="706" alt="advanced resolver options" src="https://github.com/user-attachments/assets/c8b0a51c-8f53-4fc6-8d54-c2f55dbee20c" />

- **Prefetch Support**: Lets pfSense request DNS info ahead of time, so lookups feel faster when you actually need them.  
- **DNS Key Support**: Adds an extra security check to ensure DNS data hasn’t been tampered with.

Next, go to:  
**System → Advanced → Networking → Network Interfaces**

- Check **Hardware Checksum Offloading**.

This helps pfSense handle more network traffic efficiently and keeps performance smooth.

<img width="1297" height="706" alt="harwareChecksumOffloading" src="https://github.com/user-attachments/assets/8f154c90-b18f-442b-8e7f-cf2fc0035c61" />

Click **Save**, let pfSense reboot, and then log back in using the new password you set earlier.

That’s it! pfSense is fully configured and ready to go!

Now on the pfSense dashboard:

- Go to **Status → DHCP Leases**
- You can see that your Kali machine is in the Leases section
<img width="1297" height="706" alt="kaliInLeases" src="https://github.com/user-attachments/assets/135959a8-6927-4d12-9f06-349f6619b803" />

### Verifying Kali’s Connection to pfSense

Now that pfSense is fully configured, let’s confirm that Kali is properly connected.

- Open a terminal in Kali and run:

    ```bash
    ifconfig
    ip route
    ```

- In `ifconfig`, you should see the IP address assigned to Kali.
- Under `ip route`, look for:

    ```bash
    default via 10.0.0.1
    ```

    This matches pfSense’s **LAN IP**, which confirms that Kali is routing traffic through the firewall.

---

### Assigning a Static IP to Kali from pfSense

Right now, my Kali machine has the IP **10.0.0.11**. To make it static:

- Go to the **Leases** section in pfSense.
- Find the entry for Kali.
- Under **Actions**, click the white **“+”** button to convert it into a static lease.

This ensures Kali always gets the same IP address, which is super helpful for firewall rules and testing.
<img width="1297" height="706" alt="addstaticipforkali" src="https://github.com/user-attachments/assets/4b2e0cdc-a81a-49af-8255-ba0df15d4fc2" />

### Changing Kali’s IP Address

Once you click the “+” to make the IP static, you’ll see a field to set the address.  
I changed mine to **10.0.0.2**.

**Note:**  
When picking a static IP, make sure:  
- It’s outside the DHCP range (for example, below `10.0.0.11` or above `10.0.0.243`) so there are no conflicts.  
- It’s in the same subnet as the LAN.  
- Avoid reserved addresses like `.0` or `.255`.

After saving, apply the changes and restart networking in Kali (or just reboot the VM) to see the new IP in action.
<img width="1297" height="706" alt="newKali ip" src="https://github.com/user-attachments/assets/10edbbea-d103-404c-a9f5-78cfd023948f" />

### Applying the Static IP Changes

After saving and applying your changes in pfSense, if you check Kali’s IP address right away, it might still show the old one (`10.0.0.11`).  

To fix this, open a terminal in Kali and run:

```bash
sudo ip link set eth0 down && sudo ip link set eth0 up
```
This restarts the network interface eth0, helping it pick up the new IP and reset its state.

Now, run ifconfig again, and you should see the new static IP you assigned!

<img width="1297" height="706" alt="kali ip address fix" src="https://github.com/user-attachments/assets/889d9a68-64ef-4351-bc6e-4cd69e1370b2" />


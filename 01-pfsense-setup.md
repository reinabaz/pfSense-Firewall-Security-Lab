### **What This Lab Is About**

I wanted to build a hands-on network security setup where I could experiment safely, so I used **pfSense** as the firewall, **Kali Linux** to play the attacker, and **Ubuntu** as the victim machine. The idea was to practice **network segmentation**, set up **firewall rules**, and then test if those rules actually hold up under attack.

This lab takes you through everything I did. From installing pfSense and configuring interfaces to creating custom firewall rules and running some attack simulations to see what works and what doesn't.

---

### **What I Used**

For this setup, I spun up three virtual machines:

- **pfSense Firewall**
- **Ubuntu** (victim)
- **Kali Linux** (attacker)

I grabbed the pfSense Community Edition ISO from their official site:

https://www.pfsense.org/download/

### **Setting Up pfSense in VirtualBox**

After downloading the pfSense ISO from Netgate, it's time to set it up in our hypervisor. I'm using **VirtualBox** for this lab, but you can use any hypervisor you like.

1. **Create a New VM**
   - Name it whatever you want.
   - Select the ISO you downloaded earlier.
   - For **Type**, choose **BSD** (since pfSense is based on FreeBSD).
2. **Allocate Resources**
   - **Base Memory**: I gave it 2 GB (2048 MB).
   - **Storage**: 20 GB disk size should be plenty for this lab.
3. **Organize Your VMs**
   - I like to keep things tidy, so I put this VM into a **group** in VirtualBox.
   - To do this:
       - Right-click the VM → **Move to Group** → **New** → Give the group a name.
   - I named mine **Firewall**, but you can call it whatever makes sense for you.

### **Adjusting System Settings**

Next, open your new pfSense VM and go to:

**Settings → System**

- **Uncheck Floppy**: We don't need it since it's outdated and unnecessary.
- **Boot Order**: Move **Hard Disk** to the top, and keep **Optical** (where your pfSense ISO is) right after it.

This ensures the VM boots from the ISO first, and later from the installed pfSense system.

<img width="1366" height="768" alt="bootorder" src="https://github.com/user-attachments/assets/d505f9b6-3875-4dc4-9389-2a18ce93b119" />

**Optional Settings**  
There are a couple of things we don't really need for this lab:  
* **Audio**: You can turn this off under the Audio tab.  
* **USB**: Same thing,disable it if you want. It just keeps things cleaner.  

**Setting Up Networking**  
Now for the important part, the network adapters.  
* **Adapter 1**:  
   * Set it to **NAT** so pfSense can connect to the internet. This will act as the **WAN** interface in our lab.  
   * For the adapter type, I went with **Paravirtualized Network (virtio-net)** because it gives the best performance.
 <img width="1366" height="768" alt="nat" src="https://github.com/user-attachments/assets/c1a8e561-fb79-4cd9-b265-a3a0349f3193" />
 
- For Adapter 2:
    - Choose `Internal Network`
    - Adapter Type: `Paravirtualized Network(virtio-net)`
    - Name: **lan0** since this internal network will serve as our LAN in the lab.
<img width="1366" height="768" alt="lan0" src="https://github.com/user-attachments/assets/9c292477-84ae-401f-84a5-138eb562c282" />

- For Adapter 3:
    - Choose `Internal Network`
    - Adapter Type: `Paravirtualized Network(virtio-net)`
    - Name: **lan1**
<img width="1366" height="768" alt="lan1" src="https://github.com/user-attachments/assets/bc73ac45-59b5-40a2-854a-b22617e56d66" />

- For Adapter 4:
    - Choose `Internal Network`
    - Adapter Type: `Paravirtualized Network(virtio-net)`
    - Name: **lan2**
  <img width="1366" height="768" alt="lan2" src="https://github.com/user-attachments/assets/e947c692-f3ff-4786-91c6-99252c6bed93" />

 
### **Installing pfSense**

- Start your pfSense VM.  
- I kept all the default options and just hit **Continue** at each step.  
- Choose to install the **Community Edition (CE)**.  
- Again, leave the rest as default.  
- When prompted for the version, I went with the **stable release** because it’s well-tested and reliable.  
- Once the installation finishes, the VM will reboot.  
- After reboot, it’ll ask if you want to set up VLANs ,since we don’t need them for this lab, just type **n** and hit Enter.
<img width="720" height="400" alt="vlansetupquestion" src="https://github.com/user-attachments/assets/0dc91183-ba5d-4a24-bda6-386cfd470bb2" />

When asked to enter the WAN interface name or press `a` for autodetection:

- You’ll see a list like this (based on your setup):
    - First interface: `vtnet0`
    - Second interface: `vtnet1`
    - Third interface: `vtnet2`
    - Fourth interface: `vtnet3`
- Just go ahead and type `y` when it asks **Do you want to proceed?**

You will get the following after it’s done installing:

<img width="720" height="400" alt="welcometopfsense" src="https://github.com/user-attachments/assets/1c5f56f8-991c-4e9c-8c61-bd0d341dce79" />

Setting Up a Static IP for our LAN:
- Choose option 2 which is `Set interface(s) IP address`
<img width="720" height="400" alt="option2" src="https://github.com/user-attachments/assets/05a0cec8-2ba5-4a4f-8303-95d8318251a8" />

### Setting Up a Static IP for the LAN Interface

- Choose option 2 to set interface(s) IP address.  
- When asked to configure IPv4 via DHCP, type `n` to set a static IP manually.  
- Enter `10.0.0.1` for the new LAN IPv4 address.  
- Enter `24` for the subnet mask (255.255.255.0).  
- For WAN settings, just press Enter to skip.  
- Type `n` for IPv6 via DHCP6.  
- Press Enter to skip entering an IPv6 address.  
- Enable the DHCP server on LAN by typing `y`.  
- Set DHCP client range start to `10.0.0.11` and end to `10.0.0.243`.  
- When asked to revert to HTTP for the web interface, type `n` to keep HTTPS enabled.

<img width="720" height="400" alt="afterLANconfiguration" src="https://github.com/user-attachments/assets/4b225e60-50be-4fde-982a-176f43851630" />

### Configuring The Other LANs
<img width="720" height="400" alt="otherLANs" src="https://github.com/user-attachments/assets/20dcde23-f2fc-43b9-bf14-9c5d9db23a54" />

### Configuring OPT1 Interface

- Choose option 2 to configure interfaces.
- Select option 3 for OPT1.
- When asked if you want to configure IPv4 via DHCP, type `n` to assign a static IP.
- Set the new OPT1 IPv4 address to `10.6.6.1`.
- Enter subnet bit count as `24` (equals `255.255.255.0`).
- For WAN settings, press Enter to skip.
- When asked about IPv6 via DHCP6, type `n`, then press Enter to skip IPv6 configuration.
- Enable DHCP server on OPT1 by typing `y`.
- Set the DHCP range:
  - Start: `10.6.6.11`
  - End: `10.6.6.243`
- When asked if you want to revert to HTTP for the web interface, type `n` to keep HTTPS.
- OPT1 is now configured with its own subnet for segmentation.
<img width="720" height="400" alt="OPT1" src="https://github.com/user-attachments/assets/289174d2-612e-4ca1-b26e-cfd9ccaebe85" />
<img width="720" height="400" alt="afterOPT1config" src="https://github.com/user-attachments/assets/57bbd3cb-801c-4d5c-8d80-c920311f1054" />

### Configuring OPT2 Interface

- Choose option 2 to configure interfaces.
- Select option 4 for OPT2.
- When asked if you want to configure IPv4 via DHCP, type `n` to assign a static IP.
- Set the IPv4 address to `10.80.80.1`.
- Enter subnet bit count as `24` (equals `255.255.255.0`).
- Skip WAN settings by pressing Enter.
- When asked about IPv6 via DHCP6, type `n`, then press Enter to skip IPv6 address prompt.
- When asked about enabling DHCP on OPT2, type `n` (no automatic IP assignments).
- When asked about reverting to HTTP for the web interface, type `n` to keep HTTPS.
- **Note:** OPT2 has DHCP disabled, so devices connected to OPT2 must be assigned IPs manually.
<img width="720" height="400" alt="OPT2" src="https://github.com/user-attachments/assets/ea47a22d-f298-4d94-a4db-aed29e7b3235" />

**NOTE:**

I picked **10.6.6.1** for OPT1 and **10.80.80.1** for OPT2 just to make them different from the LAN address (**10.0.0.1**).

There’s no special reason for these exact numbers, you can choose almost any private IPs you like, as long as:

1. They don’t overlap with each other.
2. They’re from a **private IP range**:
   - 10.x.x.x
   - 172.16.x.x – 172.31.x.x
   - 192.168.x.x

Keeping things unique and within these ranges will make sure your network runs smoothly.
<img width="720" height="400" alt="afterOPT2config" src="https://github.com/user-attachments/assets/1ee1eae6-1780-4a6c-8290-ab5a5f6f77c1" />

At this point, we’ve successfully set static IPs for **LAN**, **OPT1**, and **OPT2**.

The last step for now is to shut down pfSense so we can move on to the next part of the setup. To do that, choose option **6** (`Halt system`).


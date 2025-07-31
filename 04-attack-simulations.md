### Attack Simulation

### **Scenario 1: Basic Network Segmentation Test**

In this lab, we have:

- **Ubuntu VM** as the victim on `OPT1` (our internal LAN).
- **pfSense** as the firewall controlling traffic.
- **Kali Linux** as the attacker on a **bridged adapter**, representing an external attacker outside the internal network.

---

### **Testing the Setup**

1. **Verify Internet Connectivity on Both Machines**
    - Run `ping 8.8.8.8` on both Kali and Ubuntu, both should have successful replies, confirming internet access.
2. **Ping from Kali to Ubuntu**
    - Run `ping <Ubuntu IP>` from Kali.
    - This should **fail** because inbound traffic from outside to LAN is blocked by default firewall rules.
3. **TCP SYN Scan with Nmap**
    
    Run:
    
    ```bash
    
    nmap -sS <Ubuntu IP>
    ```
    
    Expected result:
    
    ```
    
    Note: Host seems down. If it is really up, but blocking our ping probes, try -Pn
    ```
    
    This shows the firewall is blocking probes from Kali to Ubuntu.
    
4. **Aggressive Scan / Version Detection**
    
    You can try:
    
    ```bash
    
    nmap -sV <Ubuntu IP>
    nmap -A <Ubuntu IP>
    ```
    
    Both should show similar "host down" messages, confirming the firewall blocks these scans.
    
5. **Test SSH Access from Kali**
    
    Run:
    
    ```bash
    
    ssh <Ubuntu IP>
    ```
    
    This should **fail**, as expected, due to the firewall blocking inbound SSH.
    
6. **Bruteforce SSH Using Hydra**
    
    Run:
    
    ```bash
    
    hydra -l ubuntu -P pass.txt ssh://<Ubuntu IP>
    ```
    
    Expected output includes a connection timeout error:
    
    ```
    
    [ERROR] could not connect to ssh://<Ubuntu IP>:22 - Timeout connecting to <Ubuntu I
    ```
    
    This confirms SSH is inaccessible from Kali, protecting the Ubuntu VM.
    

---

This confirms our network segmentation is working as intended.

### **Scenario 2:**

Now let’s simulate an attack where Kali is an outside attacker left on bridged adapter and ubuntu is the victim on OPT1

1. **Ping Ubuntu from Kali:**
    
    ```bash
    ping <Ubuntu IP>
    ```
    
    This fails, as expected, because inbound traffic from outside to the OPT1 subnet is blocked by default.
    
2. **Nmap SYN Scan from Kali to Ubuntu:**
    
    ```bash
    nmap -sS <Ubuntu IP>
    ```
    
    Output:
    
    ```
    Note: Host seems down. If it is really up, but blocking our ping probes, try -Pn
    ```
    
    This confirms that scanning is blocked.
    
3. **Nmap Aggressive and Version Scan:**
    
    ```bash
    nmap -sV <Ubuntu IP>
    nmap -A <Ubuntu IP>
    ```
    
    Both show the host as down, confirming no access.
    
4. **Attempt SSH Connection:**
    
    ```bash
    ssh ubuntu@<Ubuntu IP>
    ```
    
    This also fails because inbound SSH traffic on port 22 is blocked.
    
5. **Bruteforce SSH with Hydra:**
    
    ```bash
    hydra -l ubuntu -P pass.txt ssh://<Ubuntu IP>
    ```
    
    Result includes:
    
    ```
    [ERROR] could not connect to ssh://<Ubuntu IP>:22 - Timeout connecting to <Ubuntu IP>
    ```
    
    Again confirming no access from Kali to Ubuntu.


### **About OPT2:**

Since OPT2 has **no DHCP server and no internet access configured**, it’s effectively isolated.

Testing or accessing OPT2 from outside networks is tricky without manually assigning IP addresses and setting up routing. This isolation is intentional for tighter security.


### **Scenario 3: Attacker on LAN, Victim on OPT1 (LAN1)**

In this setup:

- **Kali (attacker)** is on `lan0` (LAN).
- **Ubuntu (victim)** is on `OPT1` (which we’re calling LAN1).

---

1. **Check Internet Connectivity**

    On both machines, ping `8.8.8.8` to confirm internet access — both should work fine.

2. **Ping Ubuntu from Kali**

    Running `ping <Ubuntu IP>` from Kali should succeed because the firewall allows replies from OPT1 to LAN (based on the second rule we set).

3. **TCP SYN Scan from Kali to Ubuntu:**

    ```bash
    nmap -sS <Ubuntu IP>
    ```

    Sample output:

    ```
    Host is up (0.011s latency).
    PORT   STATE SERVICE
    22/tcp open  ssh
    ```

    This shows port 22 (SSH) is open on Ubuntu.

4. **Aggressive and Version Scans:**

    These scans will also succeed because pfSense’s default LAN rule allows all outbound traffic from LAN to any destination, including OPT1.

5. **SSH Brute Force Using Hydra:**

    ```bash
    hydra -l ubuntu -P pass.txt ssh://<Ubuntu IP>
    ```

    Expected success output:

    ```
    1 of 1 target successfully completed, 1 valid password found
    ```

    This means the brute force attack succeeded, allowing SSH access.

---

### **Why Do Scans and Attacks from Kali → Ubuntu Work?**

Kali is on the LAN interface, which by default has a rule permitting:

`IPv4 * LAN subnets → * (Allow)`

This rule allows all traffic originating from LAN to any destination unless explicitly blocked. So, Nmap scans, SSH attempts, and brute force attacks from Kali to Ubuntu are allowed.

---

### **Why Can’t Ubuntu Move Laterally to LAN?**

Ubuntu sits on OPT1, where the firewall rules include:

- Allowing traffic only to **non-private IPs** (like internet addresses).
- A **block all** rule that prevents any other access.

Since LAN’s IP range is private (10.0.0.0/8), Ubuntu cannot reach LAN or pfSense internally, effectively **blocking lateral movement** from OPT1 to LAN.

This lab demonstrated how pfSense enforces strong network segmentation and firewall policies, allowing controlled communication while preventing lateral movement and external attacks.

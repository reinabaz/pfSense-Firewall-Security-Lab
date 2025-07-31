This guide walks you through setting up firewall rules to control traffic between your network segments, helping you secure and isolate your lab environment effectively.

### **Setting Up Firewall Rules**

- Navigate to **Firewall → Rules → LAN**.
- Click **Add** (use the **upper arrow** to add the rule at the top of the list).
<img width="1297" height="706" alt="add firewall rule" src="https://github.com/user-attachments/assets/5d3ebea7-4af9-47c0-b9bb-872860a603cd" />

**Creating a Firewall Rule to Block Traffic**

We want to create a rule that **blocks** traffic from LAN to WAN.

- **Action:** Select **Block**.
- **Address Family:** Choose **IPv4 + IPv6** to cover both protocols.
- **Protocol:** Set to **Any** to block all types of traffic.
- **Source:** Select **LAN subnets**.
- **Destination:** Select **WAN subnets**.
- **Description:** Always write a clear description explaining what the rule does, in this case, something like:

    *“Block access to the service on WAN interface.”*
<img width="1297" height="706" alt="block access to wan interface" src="https://github.com/user-attachments/assets/b86fff65-803d-4b2c-bf74-824a07f4cd0d" />

### **Creating Aliases for Private IP Ranges**

1. Go to **Firewall → Aliases → Add**.
2. Fill in the details:
   - **Name:** Choose a name that makes sense for you (e.g., `Private_Networks`).
   - **Description:** Something like *“Private IPv4 address space”*.
   - **Type:** Select **Network(s)**.
3. Add the following networks (these cover all the private IP ranges):
   - `10.0.0.0/8`
   - `172.16.0.0/12`
   - `192.168.0.0/16`
   - `127.0.0.0/8` (loopback range)

---

**Why these ranges?**

- The **RFC 1918 ranges** (`10.x.x.x`, `172.16.x.x–172.31.x.x`, and `192.168.x.x`) are reserved for local/private networks like homes, offices, and labs. They don’t route on the public internet.
- The **loopback range (`127.0.0.0/8`)** is used internally by devices to communicate with themselves.

Including all these in one alias simplifies firewall rule management and helps keep your network secure by clearly defining local IP spaces.
<img width="1297" height="693" alt="alias" src="https://github.com/user-attachments/assets/32aa8de9-6b8e-444f-874c-e3f7fcf6a295" />

### **Setting Firewall Rules for OPT1**

We’ll add several rules at the **end of the list** (downward arrow).

---

#### **1. Allow OPT1 to OPT2 Traffic**
- **Action:** Pass (leave as is)
- **Address Family:** IPv4 + IPv6
- **Protocol:** Any
- **Source:** OPT1 address (or whatever you named it)
- **Destination:** OPT2 address (or your chosen name)
- **Description:** *Allow traffic for all devices on the OPT1 network*
- **Save** and **Apply Changes**

---

#### **2. Allow OPT1 to Kali VM**
- **Action:** Pass
- **Address Family:** IPv4
- **Protocol:** Any
- **Source:** OPT1 subnet
- **Destination:** Address or Alias → Kali Linux IP (`10.0.0.2`)
- **Description:** *Allow traffic to Kali Linux VM*
- **Save** and **Apply Changes**

---

#### **3. Allow OPT1 to Non-Private IPs**
- **Action:** Pass
- **Address Family:** IPv4
- **Protocol:** Any
- **Source:** OPT1 subnet
- **Destination:** Check **Invert match**, then choose the alias we created for private IPs.
    - *(Invert match means: “all IPs except those in the alias,” so this allows traffic to any non-private IP.)*
- **Description:** *Allow to any non-private IPv4 address*
- **Save** and **Apply Changes**

> This rule isolates OPT1 from other private networks, reducing the risk of lateral movement by attackers across your internal network.  

---

#### **4. Block All Other Traffic from OPT1**
- **Action:** Block
- **Address Family:** IPv4 + IPv6
- **Protocol:** Any
- **Source:** OPT1 subnet
- **Destination:** Any
- **Description:** *Block access to everything else*
- **Save** and **Apply Changes**

<img width="1366" height="702" alt="OPT1 rules" src="https://github.com/user-attachments/assets/8a2f23d5-329c-4cf6-b9d8-882a10b3bce6" />

### **Diagram for OPT1 Rules**

```bash
OPT1 devices
│
▼
+-------------------+
| Is destination    |
| OPT2 address?     |───Yes──► Allow (Rule 1)
|                   |
+-------------------+
│ No
▼
+-------------------+
| Is destination    |
| Kali Linux IP?    |───Yes──► Allow (Rule 2)
|                   |
+-------------------+
│ No
▼
+------------------------------+
| Is destination NOT in private |
| IP ranges or loopback?        |───Yes──► Allow (Rule 3)
+------------------------------+
│ No
▼
+----------------+
| Block traffic  | (Rule 4)
+----------------+
```

### **Creating Firewall Rules for OPT2**

We’ll add these rules at the **end of the list for OPT2**:

---

#### **1. Block OPT2 Access to WAN**
- **Action:** Block
- **Address Family:** IPv4 + IPv6
- **Protocol:** Any
- **Source:** OPT2 subnet
- **Destination:** WAN subnet
- **Description:** *Block access to the service on WAN interface*
- **Save** and **Apply Changes**

---

#### **2. Block OPT2 Access to OPT1**
- **Action:** Block
- **Address Family:** IPv4 + IPv6
- **Protocol:** Any
- **Source:** OPT2 subnet
- **Destination:** OPT1 subnet
- **Description:** *Block traffic to OPT1 interface*
- **Save** and **Apply Changes**

---

#### **3. Allow OPT2 to All Other Destinations**
- **Action:** Pass
- **Address Family:** IPv4 + IPv6
- **Protocol:** Any
- **Source:** OPT2 subnet
- **Destination:** Any
- **Description:** *Allow traffic to all other subnets and the internet*
- **Save** and **Apply Changes**
<img width="1297" height="693" alt="OPT2 rules" src="https://github.com/user-attachments/assets/f198e083-0e92-4bdb-ad70-a41abdd46bf0" />

### **Diagram for OPT2 Rules**

```bash
OPT2 devices
     │
     ▼
+---------------------------+
| Is destination WAN subnet? |───Yes──► Block (Rule 1)
+---------------------------+
     │ No
     ▼
+---------------------------+
| Is destination OPT1 subnet? |───Yes──► Block (Rule 2)
+---------------------------+
     │ No
     ▼
+----------------+
| Allow traffic  | (Rule 3: Pass to all other subnets & internet)
+----------------+
```

### **Reboot pfSense**

After finishing all the firewall configurations, reboot pfSense to apply changes properly:

- Go to: **Diagnostics → Reboot → Submit**


##         **Samba and Samba server LINUX / / WINDOW**

---

Let’s break it down **step-by-step**, starting from *what Samba is*, then *how to configure it*, and finally how to *access it from both Linux and Windows systems.*

## **🧠 What is Samba?** 

**Samba** is an **open-source implementation of the SMB/CIFS protocol**, which allows **file and printer sharing** between **Linux/Unix** and **Windows** systems.

### **🔹 In short:**

* Samba lets **Linux share files/folders with Windows**.

* It also allows **Linux to access shared folders from Windows**.

* It uses the **SMB (Server Message Block)** protocol.

---

## **⚙️ Samba Server Installation and Configuration (Linux to Linux / Windows)**

We’ll do this in **three parts**:

---

### **🧩 Part 1 — Setup Samba Server (on Linux)**

#### **1️⃣ Install Samba packages**

> **NOTE:** For RHEL / CentOS / Fedora 
```
yum install samba samba-client samba-common -y
```
#### **2️⃣ Create a directory to share**
```
mkdir -p /srv/samba/shared
```

#### **3️⃣ Set proper permissions**
```
chmod -R 0777 /srv/samba/shared  
chown -R nobody:nobody /srv/samba/shared
```
*(In production, give limited permissions instead of 777.)*

---

#### **4️⃣ Backup and edit Samba configuration file**
```
cp /etc/samba/smb.conf /etc/samba/smb.conf.bak  
vi /etc/samba/smb.conf
```
At the **end of the file**,
add:
```
[SharedFolder]  
   comment = Public Share  
   path = /srv/samba/shared  
   browseable = yes  
   writable = yes  
   guest ok = yes  
   read only = no
```
>👉 **NOTE:** This creates a **public share** named `SharedFolder`.

---

#### **5️⃣ Enable and start Samba services**
```
systemctl enable smb nmb  
systemctl start smb nmb  
systemctl status smb nmb
```
---

#### **6️⃣ Allow Samba through firewall**

firewall-cmd --permanent --add-service=samba  
firewall-cmd --reload

---

#### **7️⃣ Verify configuration syntax**

testparm

> **NOTE->**✅ It should show “Loaded services file OK”.

---

### **🧩 Part 2 — Access Samba Share from Another Linux System**

#### **🧭 Option 1: Temporary Mount (manual)**

From the **client Linux machine**:
```
mkdir /mnt/samba
```
```
mount -t cifs //192.168.1.100/SharedFolder /mnt/samba -o guest
```
>**NOTE***(Replace `192.168.1.100` with your Samba server IP.)*

>**NOTE**To check:
```
df -h | grep samba  
ls /mnt/samba
```
#### **🧭 Option 2: Permanent Mount (automatic)**

Add this line to `/etc/fstab` on the client:
```
//192.168.1.100/SharedFolder /mnt/samba cifs guest,_netdev 0 0
```
Then mount:
```
mount -a
```
---

### **🧩 Part 3 — Access Samba Share from Windows**

#### **🪟 Step-by-Step:**

Press **Windows + R → type**
```
 \\192.168.1.100\\SharedFolder
```
1.   
2. Press **Enter** → You should see the folder open.

3. You can **map it as a network drive**:

   * Right-click “This PC” → “Map Network Drive”

   * Choose drive letter (e.g., `Z:`)

   * Folder: `\\192.168.1.100\SharedFolder`

   * Click **Finish**.

✅ Now Windows can read/write files in the Samba share.

---

## **🔐 (Optional) Password-Protected Samba Share**

If you want **user-based access**, do this instead of `guest ok = yes`.

Create a Linux user:
```
useradd sambauser  
passwd sambauser
```
1. 

Add to Samba password database:
```
 smbpasswd -a sambauser
```
2. 

Edit `/etc/samba/smb.conf`:

 [SecureShare]  
   comment = Secured Share  
   path = /srv/samba/secure  
   valid users = sambauser  
   guest ok = no  
   writable = yes

3. 

Restart service:
```
 systemctl restart smb nmb
```
4.   
5. Access using username/password:

   * On Windows: enter `sambauser` and password when prompted.

On Linux:
```
 mount -t cifs //192.168.1.100/SecureShare /mnt/samba \-o username=sambauser
```
* 

---

## **🧾 Summary Table**

| Action | Command/Step |
| ----- | ----- |
| Install Samba | `yum install samba samba-client -y` |
| Start Services | `systemctl enable --now smb nmb` |
| Add Share | Edit `/etc/samba/smb.conf` |
| Verify Config | `testparm` |
| Allow Firewall | `firewall-cmd --add-service=samba` |
| Linux Mount | `mount -t cifs //IP/share /mnt` |
| Windows Access | `\\IP\share` in Explorer |

---


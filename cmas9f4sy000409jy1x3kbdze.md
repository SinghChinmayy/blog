---
title: "How to Restore SSH Access to a Google Cloud VM After Blocking Port 22"
datePublished: Mon Dec 23 2024 18:30:00 GMT+0000 (Coordinated Universal Time)
cuid: cmas9f4sy000409jy1x3kbdze
slug: how-to-restore-ssh-access-to-a-google-cloud-vm-after-blocking-port-22
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/4vzYQAuMcMA/upload/9e9a9d7e3c6b3f9464eb90f18701c0cf.jpeg
tags: cloud, aws, ssh, google-cloud-platform

---

Accidentally blocking SSH (port 22) on your Google Cloud VM can be a nerve-wracking experience, especially if you’ve locked yourself out by misconfiguring the firewall. Fortunately, Google Cloud provides a robust way to recover access using the Serial Console and startup scripts. Here’s a refined, step-by-step guide inspired by Erik Fredericks’ excellent [YouTube tutorial](https://www.youtube.com/watch?v=8cKnIkYYsDQ&t=611s).

---

### **Why Does SSH Access Get Blocked?**

A common mistake is enabling a firewall (like `ufw`) inside your VM without explicitly allowing port 22. Even if your Google Cloud VPC firewall allows SSH, a restrictive internal firewall can block all incoming SSH connections, leaving you unable to connect.

---

## Step-by-Step Recovery Guide

### 1\. **Stop Your VM**

* Go to your VM instance in the Google Cloud Console.
    
* Click **Stop** to shut down the VM. This is required to make certain configuration changes.
    

---

### 2\. **Enable Serial Console Access**

* With the VM stopped, click **Edit**.
    
* Scroll to the **“Enable connecting to serial ports”** option and check the box.
    
* Save your changes. This feature lets you interact with your VM as if you were physically at its terminal.
    

---

### 3\. **Add a Startup Script to Create a Temporary Admin User**

* In the VM settings, find the **“Automation”** or **“Metadata”** section.
    
* Add a startup script like this:
    

```bash
#!/bin/bash
useradd -m tempadmin
echo 'tempadmin:TempPassword123!' | chpasswd
usermod -aG sudo tempadmin
```

Replace `tempadmin` and `TempPassword123!` with your own username and a strong, temporary password.

---

### 4\. **Restart the VM**

* Start your VM. The startup script runs automatically, creating a new user with sudo privileges.
    

---

### 5\. **Connect via Serial Console**

* From the VM’s page, click **Connect to Serial Console**.
    
* Log in with the temporary username and password you set in the script.
    

---

### 6\. **Re-Enable SSH Access**

* Once logged in, run:
    

```bash
sudo ufw allow 22
```

This command opens port 22 for SSH.

---

### 7\. **Clean Up for Security**

* **Stop the VM** again.
    
* **Remove the startup script** from the metadata to prevent it from running again.
    
* **Disable serial port access** for security.
    
* **Restart the VM**.
    
* **Log in via SSH** as usual.
    
* **Delete the temporary user** after confirming SSH works:
    

```bash
sudo deluser tempadmin
sudo rm -rf /home/tempadmin
```

---

## **Best Practices**

* **Always allow port 22** in both your VM and VPC firewalls before enabling `ufw`.
    
* **Remove temporary users and scripts** as soon as you regain access.
    
* **Regularly back up your VM** to avoid future lockouts.
    

---

## **Quick Reference Table**

| Step | Action |
| --- | --- |
| Stop VM | Shut down the affected VM |
| Enable Serial Console | Edit VM settings to allow serial port access |
| Add Startup Script | Insert script to create a temp admin user |
| Start VM | Boot up so the script runs |
| Connect via Serial Console | Log in with temporary credentials |
| Allow SSH in ufw | Run `sudo ufw allow 22` |
| Remove Script & Serial Access | Stop VM, remove script, disable serial, restart, delete temp user |

---
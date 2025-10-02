
# Kali to Splunk Log Forwarding

## Project Description
This project demonstrates the setup of **Splunk Enterprise** on a Windows 11 host and the **Splunk Universal Forwarder** on a Kali Linux virtual machine. The goal is to **forward system logs (auth.log and syslog) from Kali to Splunk** and visualize them through Splunk Web.

---

## Theory
- **Splunk Enterprise:** A platform for collecting, indexing, and visualizing machine-generated data.  
- **Splunk Universal Forwarder (UF):** A lightweight agent installed on a source machine (Kali) to send logs to Splunk Enterprise.  
- **Indexes & Sourcetypes:**  
  - `index = kali_logs` stores all Kali logs.  
  - `sourcetype = linux_secure` for `/var/log/auth.log`.  
  - `sourcetype = linux_syslog` for `/var/log/syslog`.  
- **Use Case:** Monitor authentication events, system logs, and generate reports or alerts for failed login attempts.

---

## Prerequisites
- Windows 11 host machine  
- Kali Linux VM (any version supporting Splunk UF)  
- Administrative / sudo privileges on both systems  
- Internet connection for downloading Splunk installers

---

## Step 1 — Install Splunk Enterprise on Windows 11
1. Download the Windows Splunk Enterprise `.msi` installer from [Splunk Downloads](https://www.splunk.com/en_us/download.html).  
2. Run the installer and follow the prompts.  
3. Open Splunk Web interface at:
   
   http://localhost:8000
   
4. Log in with the admin credentials created during installation.

---

## Step 2 — Install Splunk Universal Forwarder on Kali Linux
1. Download the UF `.deb` package:
   
   wget -O splunkforwarder-10.0.1-c486717c322b-linux-amd64.deb "https://download.splunk.com/products/universalforwarder/releases/10.0.1/linux/splunkforwarder-10.0.1-c486717c322b-linux-amd64.deb"
   
2. Install the package:
   
   sudo dpkg -i splunkforwarder-10.0.1-c486717c322b-linux-amd64.deb
   
3. Start Splunk UF for the first time:
   
   sudo /opt/splunkforwarder/bin/splunk start --accept-license
   

---

## Step 3 — Configure Forwarding to Splunk Enterprise
1. Add the Windows Splunk Enterprise host as a forward server:
   
   sudo /opt/splunkforwarder/bin/splunk add forward-server 192.168.61.1:9997 -auth admin:password
   
2. Verify the forward server is active:
   
   sudo /opt/splunkforwarder/bin/splunk list forward-server
   
   Expected output:
   
   Active forwards:
       192.168.61.1:9997
   

---

## Step 4 — Configure Monitored Logs
1. Add `/var/log/auth.log` and `/var/log/syslog` for monitoring:
   
   sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log
   sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/syslog
   
2. Verify monitored files:
   
   sudo /opt/splunkforwarder/bin/splunk list monitor
   
   Output should list:
   
   /var/log/auth.log
   /var/log/syslog
   

---

## Step 5 — Test Connectivity
- Check TCP connection from Kali to Windows host:
  
  telnet 192.168.61.1 9997
  
  - Successful connection indicates forwarder can send data.

---

## Step 6 — Generate Sample Logs (Failed Login)
1. Generate a failed sudo authentication attempt on Kali:
   
   sudo -k
   sudo <wrong-password-command>
   
2. Verify event appears in Splunk Web:
   
   index=kali_logs sourcetype=linux_secure "authentication failure"
   

---

## Step 7 — Verify Logs in Splunk Web
- Open Splunk Web and run searches:
  
  index=kali_logs sourcetype=linux_secure
  index=kali_logs sourcetype=linux_syslog
  
- Confirm events include:
  
  pam_unix(sudo:auth): authentication failure
  CRON sessions
  syslog events
  

---

## Summary
- Kali logs are successfully forwarded to Splunk Enterprise.  
- Splunk Web displays logs from monitored files.  
- Forwarder is active and monitoring is verified.  

---

## Commands Summary

# Install UF
sudo dpkg -i splunkforwarder-10.0.1-c486717c322b-linux-amd64.deb

# Start UF
sudo /opt/splunkforwarder/bin/splunk start --accept-license

# Add forward server
sudo /opt/splunkforwarder/bin/splunk add forward-server 192.168.61.1:9997 -auth admin:password

# Check forward server
sudo /opt/splunkforwarder/bin/splunk list forward-server

# Add monitors
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/syslog

# Check monitors
sudo /opt/splunkforwarder/bin/splunk list monitor

# Test connectivity
telnet 192.168.61.1 9997



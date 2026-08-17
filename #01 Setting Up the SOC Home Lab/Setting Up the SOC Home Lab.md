## 🎯 Objective

The goal of this lab is to build a small SOC environment consisting of:

* 🐧 Ubuntu → Splunk SIEM
* 🐉 Kali Linux → Attacker machine
* 🪟 Windows Server 2022 / Windows 10 → End device / victim
* 📡 Splunk Universal Forwarder → Sends Windows security logs to Splunk

![alt text](<Screenshot/My Homelab Architecture.png>)

---

## ⚙️ Steps to Set Up the SOC Home Lab

### 🖥️ Step 1: Install VirtualBox

**Oracle VirtualBox** was downloaded and installed on the host machine.

VirtualBox was used to create and run the virtual machines for the SOC Home Lab.

![alt text](<Screenshot/VirtualBox Installation.png>)

---

### 💿 Step 2: Download the Operating Systems

The ISO files for the operating systems used in the lab were downloaded:

* 🐧 **Ubuntu** → Splunk SIEM
* 🐉 **Kali Linux** → Attacker
* 🪟 **Windows Server 2022 / Windows 10** → End Device / Victim

![alt text](<Screenshot/Operating System Iso.png>)

---

### 🐧 Step 3: Install Ubuntu

A new virtual machine was created in VirtualBox and **Ubuntu** was installed using the downloaded ISO file.

Ubuntu was used to host **Splunk Enterprise**.

![alt text](<Screenshot/Installing Ubuntu.png>)

---

### 🐉 Step 4: Install Kali Linux

Another virtual machine was created in VirtualBox and **Kali Linux** was installed using the downloaded ISO file.

Kali Linux was configured as the **attacker machine**.

![alt text](<Screenshot/Installing Kali.png>)

---

### 🪟 Step 5: Install Windows

Another virtual machine was created in VirtualBox and **Windows Server 2022 / Windows 10** was installed using the downloaded ISO file.

This machine was configured as the **end device / victim**.

![alt text](<Screenshot/Installing Windows Server 2022.png>)
![alt text](<Screenshot/Installing Win10.png>)

---

### 📊 Step 6: Install Splunk Enterprise on Ubuntu

After installing Ubuntu, **Splunk Enterprise** was downloaded and installed.

Splunk Enterprise was configured as the **SIEM** for the lab to collect, search, and analyze security logs from the Windows endpoint.

![alt text](<Screenshot/Download Splunk Enterprise.png>)
![alt text](<Screenshot/Splunk Web Interface.png>)

---

### 📡 Step 7: Install Splunk Universal Forwarder on Windows Server 2022

After installing Windows, the **Splunk Universal Forwarder** was downloaded and installed.

The Universal Forwarder was configured to collect Windows security logs and send them to the **Splunk Enterprise server** running on Ubuntu.

![alt text](<Screenshot/Download Universal Forwarder.png>)
![alt text](<Screenshot/Installing Universal Forwarder.png>)

---

### 🌐 Step 8: Configure the Lab Network

The virtual machines were configured so that **Ubuntu, Kali Linux, and Windows** could communicate with each other.

The **Splunk Universal Forwarder** on Windows was also configured to forward Windows security logs to the **Splunk Enterprise server** running on Ubuntu.

The basic log flow is:

<p align="center">
  <img src="Screenshot/Lab Flow.png" alt="Lab Flow" width="500">
</p>

---

## 🖥️ Final Lab Environment

The SOC Home Lab consists of three virtual machines, each with a specific role:

* 🐉 **Kali Linux** → Attacker
* 🪟 **Windows Server 2022** → End Device / Victim
* 🐧 **Ubuntu** → Splunk SIEM

### 🐉 Kali Linux — Attacker

Kali Linux is configured as the **attacker machine** and will be used to simulate attacks against the Windows endpoint.

![Kali Linux](Screenshot/Kali.png)

---

### 🪟 Windows Server 2022 — End Device

Windows Server 2022 is configured as the **monitored endpoint**. The **Splunk Universal Forwarder** is installed to collect and forward its security logs to Splunk.

![Windows Server 2022](<Screenshot/Windows Server 2022.png>)

---

### 🐧 Ubuntu — Splunk SIEM

Ubuntu is configured to host **Splunk Enterprise**, which receives, searches, and analyzes the security logs collected from the Windows endpoint.

![Ubuntu](Screenshot/Ubuntu.png)

![alt text](<Screenshot/Splunk Web Interface.png>)

---

At this point, the virtual machines and Splunk components are installed and configured, providing the foundation for the SOC Home Lab.

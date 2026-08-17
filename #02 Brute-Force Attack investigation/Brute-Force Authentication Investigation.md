# 02: Brute-Force Authentication Investigation

---

## 🎯 Objective

The goal of this lab is to simulate a **brute-force authentication attempt** against a Windows Server 2022 endpoint and investigate the resulting security events using **Splunk Enterprise**.

The investigation demonstrates how a SOC analyst can:

* 🔐 Identify repeated failed authentication attempts
* 🔎 Investigate the source of the activity
* 🕵️ Pivot on a suspicious IP address
* 🕒 Review the authentication timeline
* 📊 Analyze Windows Security Events in Splunk
* ✅ Check whether the authentication attempts resulted in a successful login

The activity was intentionally generated inside the SOC Home Lab for testing and learning purposes.

![alt text](<screenshot/Bruteforce Investigation Architecture.png>)

---

## 🖥️ Lab Environment

The investigation used the following machines:

* 🐉 **Kali Linux** → Attacker machine
* 🪟 **Windows Server 2022** → Target endpoint
* 🐧 **Ubuntu** → Splunk Enterprise SIEM
* 📡 **Splunk Universal Forwarder** → Forwards Windows security logs to Splunk

The Windows Server 2022 machine was configured to forward its security logs to the Splunk Enterprise server through the Splunk Universal Forwarder.

---

## 👤 Step 1: Create Test Users

Several test accounts were created on the Windows Server 2022 machine using **Active Directory Users and Computers (ADUC)**.

The accounts created for the lab were:

```text
lab1
lab2
lab3
```

These accounts were used as controlled targets for the authentication simulation.

![alt text](<screenshot/ADUC Users.png>)

---

## 🐉 Step 2: Simulate the Authentication Attempts

The authentication attempts were generated from the **Kali Linux** machine.

I used the `smbclient` utility to simulate an SMB login to the Windows Server:

```bash
smbclient //192.168.56.101/share -U 'PATRICK\lab1'
```

An incorrect password was intentionally entered multiple times to generate failed authentication events.

The same type of authentication attempt was performed against the test accounts created on the Windows Server.

![Kali SMB Authentication](<Screenshot/Kali SMB Authentication.png>)

---

## 📡 Step 3: Forward the Windows Security Logs

The failed authentication attempts generated **Windows Security Event ID 4625**.

Event ID **4625** represents a failed logon attempt.

The **Splunk Universal Forwarder** collected the Windows Security events from the Windows Server and forwarded them to **Splunk Enterprise** running on Ubuntu.

The log flow was:

```text
Kali Linux
    ↓
SMB Authentication Attempts
    ↓
Windows Server 2022
    ↓
Windows Security Event ID 4625
    ↓
Splunk Universal Forwarder
    ↓
Splunk Enterprise
```

![alt text](<screenshot/Lab Flow.png>)

---

# 🔎 Investigation

For the investigation, I assumed that an alert had been generated for **suspicious authentication activity**.

The investigation started with identifying the failed authentication attempts and then pivoting on the suspicious source IP address.

---

## 🚨 Step 4: Identify Failed Authentication Attempts

The first step was to search for **Windows Event ID 4625** to identify failed authentication attempts.

The following SPL query was used:

```spl
index="windows" EventCode=4625
| stats count by Account_Name Account_Domain Source_Network_Address
| sort - count
| rename count as "Failure Attempts"
```

This query groups the failed authentication events by:

* Account name
* Account domain
* Source IP address
* Number of failed attempts

![alt text](<screenshot/SPL Search 1.png>)

The results showed multiple failed authentication attempts originating from:

```text
192.168.56.1
```

This IP address was identified as the suspicious source and became the main IOC used for the rest of the investigation.

---

## 🔍 Step 5: Pivot on the Source IP Address

After identifying `192.168.56.1` as the suspicious source IP, the next step was to investigate other authentication activity associated with that address.

The following SPL query was used:

```spl
index="windows" Source_Network_Address=192.168.56.1
| stats count by EventCode Account_Name Source_Network_Address
| sort - count
```

![alt text](<screenshot/SPL Search 2.png>)

This query allowed the authentication events to be grouped by **EventCode**, **Account_Name**, and **Source_Network_Address**.

The results showed repeated failed authentication events involving the test accounts:

```text
lab1
lab2
lab3
```

The repeated failed events from the same source IP indicated that the activity was not simply an isolated login failure.

---

## 🕒 Step 6: Investigate the Authentication Timeline

The next step was to review the individual authentication events and examine when the failed attempts occurred.

The following SPL query was used:

```spl
index="windows" Source_Network_Address=192.168.56.1
| table _time Account_Name Workstation_Name Logon_Type Failure_Reason
```

![alt text](<screenshot/SPL Search 3.png>)

This query provided several useful fields:

* `_time` → Time of the authentication attempt
* `Account_Name` → Account being targeted
* `Workstation_Name` → Workstation associated with the event
* `Logon_Type` → Type of authentication
* `Failure_Reason` → Reason for the failed authentication

The events showed:

```text
Workstation: KALI
Logon Type: 3
Failure Reason: Unknown user name or bad password.
```

**Logon Type 3** represents a network logon, which is consistent with the SMB authentication activity generated during the simulation.

The failure reason also showed that the authentication attempts were being rejected because of an incorrect username or password.

The combination of repeated failures, the KALI workstation, the source IP, and the network logon type provided additional context for the investigation.

---

## 🔐 Step 7: Check for Successful Authentication

After identifying the suspicious source IP and reviewing the failed authentication events, the next step was to determine whether the same source had successfully authenticated to the Windows endpoint.

Windows **Event ID 4624** represents a successful logon.

The following SPL query was used:

```spl
index="windows" Source_Network_Address=192.168.56.1 EventCode=4624
| table Account_Name Source_Network_Address EventCode
```

![alt text](<screenshot/SPL Search 4.png>)

The search returned:

```text
0 events
```

No successful authentication event was found from `192.168.56.1` during the investigated timeframe.

This was an important part of the investigation because it helped determine whether the authentication attempts resulted in a successful login.

---

## 📊 Step 8: Review the Authentication Activity

As a final review, the authentication activity associated with the identified source IP was summarized using:

```spl
index="windows" Source_Network_Address=192.168.56.1
| stats count by EventCode Account_Name Source_Network_Address
| sort - count
```

![alt text](<screenshot/SPL Search 2.png>)

The results showed repeated **Event ID 4625** failures associated with the source IP.

This confirmed the pattern of repeated failed authentication attempts against the Windows endpoint.

---

# 📝 Investigation Findings

The investigation identified the following information:

| Field | Finding |
|---|---|
| **Source IP** | `192.168.56.1` |
| **Source Workstation** | `KALI` |
| **Target** | Windows Server 2022 |
| **Target Accounts** | `lab1`, `lab2`, `lab3` |
| **Failed Logon Event** | `4625` |
| **Logon Type** | `3` |
| **Failure Reason** | Unknown user name or bad password |
| **Successful Logon Event** | `4624` |
| **Successful Authentication** | None observed |

---

# 🚨 Investigation Conclusion

The investigation identified repeated failed authentication attempts originating from:

```text
192.168.56.1
```

The activity targeted multiple Windows accounts, including:

```text
lab1
lab2
lab3
```

The authentication events originated from the **KALI** workstation and used **Logon Type 3**, which is consistent with network-based authentication through SMB.

The repeated **Event ID 4625** failures and the failure reason:

```text
Unknown user name or bad password.
```

were consistent with the simulated brute-force authentication activity.

A follow-up search for **Event ID 4624** from the same source IP returned no results.

Based on the available logs, the activity is consistent with a **brute-force/password-guessing attempt**, with no evidence of a successful authentication or account compromise during the investigated timeframe.

---

## 🔎 Analyst Assessment

| Category | Result |
|---|---|
| **Classification** | Brute-Force / Password Guessing Attempt |
| **Source IP** | `192.168.56.1` |
| **Source Host** | `KALI` |
| **Target** | Windows Server 2022 |
| **Target Accounts** | `lab1`, `lab2`, `lab3` |
| **Failed Logon Event** | `4625` |
| **Successful Logon Event** | `4624` |
| **Successful Authentication** | None observed |
| **Assessment** | No evidence of successful compromise |

---

## 🛡️ Recommended Response

If this activity were observed in a real environment, the following actions could be considered:

* Investigate whether the same source IP targeted other Windows endpoints.
* Review authentication activity before and after the detected failures.
* Continue monitoring for successful authentication attempts from the same source.
* Review the targeted accounts for additional suspicious activity.
* Consider blocking or restricting the source if the activity is confirmed to be malicious.
* Protect or reset targeted accounts if there is concern about credential exposure.

---

## 🎯 What This Lab Demonstrated

This investigation provided hands-on experience with:

* 🔐 Windows authentication events
* 📊 Splunk SPL queries
* 🔎 Event ID 4625 analysis
* 🔐 Event ID 4624 verification
* 🌐 Source IP investigation
* 🕒 Authentication timeline analysis
* 🕵️ IOC pivoting
* 📡 Splunk Universal Forwarder log collection
* 🚨 Basic brute-force investigation
* 📝 SOC investigation and documentation

The investigation followed a simple SOC workflow:

```text
Suspicious Authentication Alert
            ↓
Identify Failed Logins
            ↓
Identify Source IP
            ↓
Pivot on Source IP
            ↓
Review Authentication Timeline
            ↓
Identify Workstation / Logon Type / Failure Reason
            ↓
Check for Successful Authentication
            ↓
Determine Whether Compromise Occurred
            ↓
Document Findings
```

---

## 🎯 Final Conclusion

This lab demonstrated how a suspicious authentication alert can be investigated using **Splunk Enterprise** and Windows Security logs.

The simulated brute-force activity generated **Event ID 4625** events, which were collected by the **Splunk Universal Forwarder** and forwarded to Splunk for analysis.

The investigation successfully identified the suspicious source IP, targeted accounts, workstation, logon type, failure reason, and authentication timeline.

A final search for **Event ID 4624** found no successful authentication from the identified source IP.

Therefore, the activity was classified as a **brute-force/password-guessing attempt with no observed successful authentication** during the investigated timeframe.

> **Note:** All authentication attempts in this project were intentionally generated inside a controlled SOC Home Lab environment for learning, detection, and investigation purposes.
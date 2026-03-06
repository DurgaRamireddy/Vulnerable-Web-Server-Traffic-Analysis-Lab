# Vulnerable-Web-Server-Traffic-Analysis-Lab

## Objective

Perform an investigation of malicious network traffic targeting a vulnerable web server to determine:
- The source of the compromise
- The attack method used
- The impact of the incident
- Security improvements to prevent future attacks
Analysis was conducted in Kibana using Packetbeat logs and a forensic review of Lab2webdirectory.tar.gz, extracted from the web root directory.

## Tools & Environment
- Kibana (ELK Stack)
- Packetbeat Logs
- Linux Log Files
- Extracted Web Directory Archive (Lab2webdirectory.tar.gz)

## Investigation Methodology
A structured investigation process was followed:

**1. Network Analysis**
- Applied Kibana filters to review traffic to and from the web server.
- Identified suspicious source IP addresses.
- Examined HTTP requests and SSH connection metadata.

**2. Timeline Reconstruction**
- Correlated HTTP requests and SSH sessions.
- Determined the attack window and sequence of events.
  
**3. File System Analysis**
- Extracted and reviewed the web root directory.
- Searched for:
- Webshells
- Base64 encoded payloads
- Suspicious PHP or shell execution functions
- Identified exposed credentials in a debug file.
  
**4. Log Review**
- Reviewed:
    - /var/log/auth.log
    - Apache logs
    - Confirmed investigation relied primarily on ELK-captured network telemetry rather than host logs.
      
**5. Correlation**
Combined:
   - Network logs
   - Extracted web files
   - System logs

To determine:
   - How the attacker gained access
   - How the system was exploited
   - How the compromise was completed

## Key Findings
**Network Activity Summary**
| Time (UTC)  | Source IP      | Destination IP | Event                                     | Observation                             |
| ----------- | -------------- | -------------- | ----------------------------------------- | --------------------------------------- |
| 14:45:09    | 192.168.xx.xx | 192.168.xx.xxx | GET /admin.html                           | Recon attempt                           |
| 14:48:18    | 192.168.xx.xxx | 192.168.xx.xxx | GET /cgi-bin/pub/pki?cmd=serverInfo (404) | CGI probing                             |
| 14:49:31    | 192.168.xx.xxx | 192.168.xx.xxx | GET /apex/listenerConfigure               | Continued scanning                      |
| 14:50:05    | 192.168.xx.xxx | 192.168.xx.xxx| GET /debug/deletethis.html                | Credential exposure                     |
| 14:51–14:53 | 192.168.xx.xxx| 192.168.xx.xxx | HTTP & SSH (80/22)                        | Attack window (SSH session established) |

## File System Analysis
After extracting Lab2webdirectory.tar.gz, the web root contained:
- /debug
- /prod
 No webshells or malicious scripts were found.

However, the file: /debug/deletethis.html
Contained hardcoded credentials: 
Username: webserver
Password: toor

This file exposed internal login credentials and was likely intended for removal before deployment but remained publicly accessible.

## Log Verification
- /var/log/auth.log showed no live SSH logs in the VM snapshot.
- Apache logs were absent.
- Evidence suggests investigation relied on ELK network data rather than local host logging.

## Indicators of Compromise (IOCs)
| Type               | Indicator                                                                      |
| ------------------ | ------------------------------------------------------------------------------ |
| Attacker IP        | 192.168.xx.xxx                                                                 |
| Recon IP           | 192.168.xx.xxx                                                                 |
| Target IP          | 192.168.xx.xxx                                                               |
| Suspicious URLs    | /admin.html, /cgi-bin/pub/pki, /apex/listenerConfigure, /debug/deletethis.html |
| Leaked Credentials | webserver : toor                                                               |
| Affected Ports     | 80 (HTTP), 22 (SSH)                                                            |

## Attack Chain Summary

The compromise occurred in the following sequence:
**1. Reconnaissance**
- Scanned for administrative and CGI pages.
**2. Credential Discovery**
- Accessed /debug/deletethis.html.
- Retrieved plaintext SSH credentials.
**3. Initial Access**
- Used exposed credentials to SSH into the server (port 22).
**4. Post-Access Activity**
- Modified website files via SSH (defacement observed).
**5. No Persistence Identified**
- No webshells or backdoors discovered.

## Root Cause
Failure to remove development artifacts before deployment.

Specifically:
- Publicly accessible debug file
- Hardcoded credentials
- Lack of environment separation

## Remediation & Recommendations
**Immediate Actions**
| Action                                   | Purpose                        |
| ---------------------------------------- | ------------------------------ |
| Delete /debug directory                | Remove exposed sensitive files |
| Change webserver password                | Invalidate leaked credentials  |
| Restrict SSH to key-based authentication | Prevent password-based attacks |
| Isolate and reimage server               | Ensure clean system state      |

## Long-Term Security Improvements
- Apply the Principle of Least Privilege for service accounts.
- Separate development and production environments.
- Enable:
    - Web Application Firewall (WAF)
    - Intrusion Detection System (IDS)
- Regularly scan for exposed debug/config files.
- Implement centralized logging and SIEM monitoring.
- Enforce secure configuration management practices.

## Conclusion
The investigation confirmed that attacker IP 192.168.36.198 exploited an exposed debug file containing valid SSH credentials.

The compromise was not due to code injection but rather credential misuse resulting from poor deployment hygiene.


This lab demonstrates the importance of:
- Removing development artifacts before production deployment
- Protecting credentials
- Enforcing secure configuration management
- Maintaining centralized monitoring for early detection


## Author
Durga Sai Sri Ramireddy </br>
Master’s Student - Cybersecurity </br>
University of Houston


*This project was developed as part of academic coursework and expanded for cybersecurity portfolio demonstration.*

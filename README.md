# Ubuntu Defender Security Lab

## Overview

This project documents my work as the **Defender** in a controlled
Attack–Defend–Monitor cybersecurity lab.

The environment consisted of three VirtualBox machines:

- **Attacker:** Kali Linux machine used to generate controlled malicious traffic
- **Defender:** Ubuntu asset host that I configured and secured
- **Auditor:** Ubuntu monitoring machine running Prometheus and Grafana

My responsibility was to deploy the Defender's services, intentionally expose
selected weaknesses during the initial phase, apply defensive security controls,
and verify that the system could resist or detect simulated attacks.

> This project was completed entirely inside an isolated lab environment for
> educational and authorized security testing.

---

## My Role: Defender

I was responsible for configuring and securing the Ubuntu asset host.

My work included:

- Building a PHP and MySQL authentication application
- Securing the application against SQL injection
- Hardening the Ubuntu operating system
- Configuring Apache and ModSecurity
- Hardening MySQL
- Securing SSH with key-based authentication
- Configuring UFW firewall rules
- Deploying Fail2Ban for brute-force protection
- Enabling AppArmor and system auditing
- Deploying Prometheus exporters for external monitoring
- Configuring authentication for the monitoring interface
- Implementing IDS rules for common attack patterns
- Supporting attack simulations and collecting defensive evidence

---

## Lab Architecture

| Role | Operating System | Purpose |
|---|---|---|
| Attacker | Kali Linux | Reconnaissance and controlled attack simulation |
| Defender | Ubuntu Linux | Hosts and protects the web application and services |
| Auditor | Ubuntu Linux | Collects metrics, creates dashboards and generates alerts |

All three machines communicated through an isolated VirtualBox laboratory
network.

---

## Defender Services

The Defender hosted the following services:

- Apache web server
- PHP authentication application
- MySQL database
- OpenSSH
- UFW firewall
- Fail2Ban
- AppArmor
- Auditd
- ModSecurity
- Prometheus Node Exporter
- Apache Exporter
- Intrusion detection rules

---

## Web Application

I developed a small PHP authentication application containing:

- `signin.php` for user authentication
- `dashboard.php` as a session-protected page
- A MySQL `users` table for storing account information

### Initial vulnerable version

The initial login implementation constructed SQL statements directly from user
input. This intentionally allowed the team to demonstrate how unsafe query
construction can expose an application to SQL injection.

### Secured version

I replaced the vulnerable database query with prepared statements and parameter
binding.

The secured implementation:

- Prevented user input from modifying the SQL query
- Rejected SQL injection authentication-bypass attempts
- Used session checks to restrict access to the dashboard
- Escaped displayed user-controlled content
- Restricted database privileges

During validation, a SQL injection payload was submitted to the login form. The
application returned `Login failed`, demonstrating that the authentication
bypass was unsuccessful.

---

## Ubuntu Hardening

I applied multiple operating-system security controls.

### Password policy

A stronger password policy was enforced using PAM:

- Minimum password length of 12 characters
- Character variation requirements
- Retry restrictions for invalid passwords

### Automatic security updates

I configured unattended upgrades so that security patches could be installed
automatically.

### Mandatory access control

AppArmor was enabled to restrict the resources and files that applications could
access.

### System auditing

Auditd was installed and enabled to record security-relevant system events.

### File permissions

Sensitive system files were restricted, including:

- `/root`
- `/etc/shadow`
- `/etc/gshadow`

### Removable-storage restriction

USB storage loading was disabled as an additional control against unauthorized
removable devices.

---

## Apache and Web-Server Hardening

I configured Apache with additional HTTP security headers, including:

- `X-Content-Type-Options`
- `X-Frame-Options`
- `Referrer-Policy`
- `Permissions-Policy`

These controls reduce exposure to MIME-sniffing, clickjacking, unnecessary
referrer leakage and unauthorized browser features.

### ModSecurity

ModSecurity was installed and enabled in blocking mode to inspect incoming web
requests and block patterns associated with attacks such as:

- SQL injection
- Cross-site scripting
- Remote code execution attempts

### HTTPS

HTTPS was enabled for encrypted browser-to-server communication. HTTP traffic
was redirected to HTTPS through Apache.

---

## MySQL Hardening

I hardened the MySQL installation by:

- Enabling password-strength validation
- Removing anonymous database users
- Preventing remote root login
- Removing default test databases
- Using a dedicated application database account
- Restricting database permissions
- Avoiding root credentials inside the application

---

## SSH Hardening

SSH was secured using public-key authentication.

The following controls were applied:

- Generated an ED25519 key pair
- Added the public key to the authorized server account
- Disabled password authentication
- Disabled direct root SSH login
- Restricted SSH access through the firewall

These controls reduce the risk of credential guessing and direct root-account
attacks.

---

## Fail2Ban Protection

Fail2Ban was configured to monitor failed SSH authentication attempts.

The SSH jail was configured to:

- Detect repeated failed logins
- Ban a source after five failed attempts
- Maintain the ban for one hour

During testing, the attacker generated repeated failed SSH logins. Fail2Ban
identified the source and added its address to the banned-IP list.

---

## Firewall Configuration

I configured UFW using a default-deny inbound policy.

Only services required by the lab were permitted, including:

- SSH for controlled administrative access
- HTTP and HTTPS for the web application
- Monitoring exporters for the Auditor
- Required application and database connectivity

Monitoring ports were restricted to the Auditor machine instead of being
available to every system on the network.

---

## Monitoring Integration

I deployed Node Exporter under a dedicated non-root system account.

The exporter was configured as a systemd service so that it:

- Started automatically during boot
- Ran without root privileges
- Exposed host metrics to the Auditor
- Allowed system activity to be visualized in Prometheus and Grafana

Authentication was added to protect access to the monitoring interface.

The Grafana dashboards were implemented by the Auditor, while my Defender
machine supplied the system metrics, web-server statistics and security events.

---

## Detection Rules

The Defender was configured to detect activity associated with:

- Repeated SSH authentication attempts
- SQL injection patterns
- TCP port scanning
- Abnormally high SYN-packet activity

These rules were validated using controlled traffic generated by the Attacker.

---

## Security Validation

| Test | Defensive control | Result |
|---|---|---|
| SSH brute-force simulation | SSH hardening, Fail2Ban and IDS | Source detected and banned |
| SQL injection attempt | Prepared statements, ModSecurity and IDS | Login bypass failed and activity was detected |
| TCP port scan | Firewall logging and IDS rule | Scanning pattern generated an alert |
| SYN-flood simulation | SYN threshold IDS rule | Abnormal SYN activity was detected |
| Vulnerability scan | Combined hardening controls | No critical, high or medium findings reported |

The Auditor used the Defender's metrics and security events to display spikes
and alerts in Grafana during the controlled attack simulations.

---

## Technologies

- Ubuntu Linux
- VirtualBox
- Apache
- PHP
- MySQL
- OpenSSH
- UFW
- Fail2Ban
- AppArmor
- Auditd
- ModSecurity
- Prometheus
- Node Exporter
- Apache Exporter
- Grafana
- Linux systemd

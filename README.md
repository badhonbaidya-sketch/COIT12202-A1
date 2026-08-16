# COIT12202 Assignment 1 – Security Portfolio

## Secure PKI, Password Security, SSH Hardening and Fail2Ban Implementation

This repository contains the implementation, configuration, testing, and evidence for the **COIT12202 Assignment 1 Security Portfolio**.

The purpose of this laboratory was to design and implement several fundamental Linux security controls in an isolated Ubuntu 20.04 LTS environment.

The implementation covers four major security areas:

1. OpenSSL Public Key Infrastructure (PKI) and HTTPS
2. Password hashing and password security
3. SSH hardening using Ed25519 certificates
4. SSH attack protection using Fail2Ban

The repository contains configuration files, public certificates, public keys, command evidence, documentation, and screenshots demonstrating that each security control was implemented and tested successfully.

> **Security Notice:** Private keys, passwords, passphrases, GitHub credentials, tokens, `/etc/shadow` contents, and other sensitive information are intentionally excluded from this repository.

---

# 1. Lab Environment

The laboratory was implemented using two Ubuntu virtual machines on an isolated network.

| Component | Hostname / Role | IP Address | Purpose |
|---|---|---|---|
| Server VM | `secure-server.local` | `192.168.88.128` | PKI, Nginx HTTPS, SSH CA, hardened SSH and Fail2Ban |
| Client VM | Client/Test Machine | `192.168.88.130` | Certificate validation, HTTPS testing, SSH authentication and security testing |

## Server Operating System

Ubuntu 20.04.6 LTS was used for the server environment.

## Main Technologies

The implementation uses:

- Ubuntu Linux
- OpenSSL
- Nginx
- PAM
- `pam_pwquality`
- SHA-512 password hashing
- OpenSSH
- Ed25519 cryptography
- OpenSSH Certificates
- SSH Certificate Authority
- Fail2Ban
- Git
- GitHub

---

# 2. Architecture Overview

The laboratory contains two separate certificate-based security systems.

## 2.1 HTTPS PKI Architecture

The HTTPS infrastructure uses a hierarchical Certificate Authority design.

```text
                   Root Certificate Authority
                     COIT12202 Root CA
                            |
                            |
                            v
               Intermediate Certificate Authority
                  COIT12202 Intermediate CA
                            |
                            |
                            v
                     Server Certificate
                    secure-server.local
                            |
                            |
                            v
                         Nginx
                     HTTPS / TLS 1.3
                            |
                            |
                            v
                       Client VM
                    192.168.88.130
```

The Root CA signs the Intermediate CA.

The Intermediate CA signs the HTTPS server certificate.

The Nginx server presents the server certificate and intermediate certificate as a certificate chain.

The client trusts the Root CA and therefore can validate the complete certificate chain.

---

# 3. Part A – OpenSSL Public Key Infrastructure

A private Public Key Infrastructure was created using OpenSSL.

The PKI contains:

- Root Certificate Authority
- Intermediate Certificate Authority
- Server private key
- Server certificate signing request
- Signed server certificate
- Certificate chain
- Nginx HTTPS configuration

---

## 3.1 Root Certificate Authority

A Root CA was created as the highest trust authority in the laboratory PKI.

The Root CA certificate contains CA-specific extensions including:

```text
Basic Constraints:
    CA:TRUE, pathlen:1

Key Usage:
    Certificate Sign
    CRL Sign
```

The Root CA uses SHA-256 with RSA for certificate signing.

The Root CA is self-signed, meaning its Subject Key Identifier and Authority Key Identifier identify the same trust authority.

The Root CA private key is intentionally **not included in this repository**.

---

## 3.2 Intermediate Certificate Authority

An Intermediate CA was created beneath the Root CA.

Architecture:

```text
Root CA
   |
   +---- signs ----> Intermediate CA
```

Using an Intermediate CA provides better security architecture than directly issuing all server certificates from the Root CA.

The certificate chain was verified using OpenSSL.

Example verification:

```bash
openssl verify -x509_strict \
  -CAfile /opt/pki/root-ca/certs/root-ca.crt \
  /opt/pki/intermediate-ca/certs/intermediate-ca.crt
```

Expected result:

```text
intermediate-ca.crt: OK
```

This confirms that the Intermediate CA certificate is correctly signed by the Root CA.

---

# 4. HTTPS Server Certificate

A server certificate was generated for:

```text
secure-server.local
```

The certificate includes the appropriate hostname information so that clients can validate the identity of the HTTPS server.

The server certificate was signed by:

```text
COIT12202 Intermediate CA
```

The trust path therefore becomes:

```text
COIT12202 Root CA
        |
        v
COIT12202 Intermediate CA
        |
        v
secure-server.local
```

---

# 5. Nginx HTTPS Configuration

Nginx was configured on:

```text
192.168.88.128
```

The HTTPS virtual server uses:

```text
https://secure-server.local/
```

The Nginx configuration uses the server private key and full certificate chain.

The configuration also allows modern TLS protocols:

```text
TLSv1.2
TLSv1.3
```

HTTP traffic can be redirected to HTTPS to ensure encrypted communication.

---

# 6. HTTPS Verification

HTTPS was tested from the Client VM using:

```bash
curl -v https://secure-server.local/
```

Successful testing demonstrated:

```text
SSL connection using TLSv1.3
```

The client also confirmed:

```text
subjectAltName: host "secure-server.local" matched cert's "secure-server.local"
```

and:

```text
SSL certificate verify ok.
```

The web server returned:

```text
HTTP/1.1 200 OK
```

The test page displayed:

```text
HTTPS is Working
COIT12202 PKI certificate chain verification successful.
```

This confirms that:

- DNS/host resolution worked
- TCP port 443 was reachable
- TLS negotiation succeeded
- The hostname matched the certificate
- The certificate chain was trusted
- Nginx successfully served the HTTPS content

---

# 7. Part B – Password Security

Password security was implemented using Ubuntu PAM security controls.

The implementation covers:

- SHA-512 password hashing
- Password quality requirements
- Test user creation
- Password hash verification
- Account lockout investigation/testing

---

# 8. SHA-512 Password Hashing

Ubuntu 20.04 was configured to use SHA-512 for new local password hashes.

The configuration was verified in:

```text
/etc/login.defs
```

with:

```text
ENCRYPT_METHOD SHA512
```

SHA-512 password hashes on this system normally begin with:

```text
$6$
```

This prefix identifies the SHA-512 crypt password format.

Changing `ENCRYPT_METHOD` does not automatically convert existing user password hashes. Therefore, a test user's password was created/changed before verifying the resulting format.

---

# 9. Password Quality Policy

The `libpam-pwquality` package was installed to enforce stronger password requirements.

The password policy was configured using:

```text
/etc/security/pwquality.conf
```

The laboratory policy included:

```text
minlen = 14
minclass = 3
difok = 4
maxrepeat = 3
```

These controls improve password strength by requiring sufficient password length and character diversity while reducing repetitive or overly similar passwords.

---

# 10. Password Security Test Account

A dedicated test account was created:

```text
securitytest
```

The account was used only for laboratory testing.

Account status was verified using:

```bash
passwd -S securitytest
```

Password hash format was inspected carefully without publishing the complete `/etc/shadow` entry.

> `/etc/shadow` must never be committed to the repository because it contains sensitive password hash information.

---

# 11. Account Lockout Testing

Ubuntu PAM account lockout functionality using `pam_faillock` was investigated and tested.

The policy values used for the laboratory were:

```text
deny = 5
fail_interval = 900
unlock_time = 900
silent
```

This represents:

| Setting | Purpose |
|---|---|
| `deny = 5` | Lock after five failed authentication attempts |
| `fail_interval = 900` | Count failures within a 15-minute period |
| `unlock_time = 900` | Automatically unlock after 15 minutes |
| `silent` | Reduce account-information disclosure |

Because PAM configuration directly affects authentication, backup copies of PAM configuration files were created before modification.

This was particularly important because an incorrect PAM stack can prevent legitimate SSH authentication.

---

# 12. Part C – SSH Hardening

OpenSSH was hardened using modern Ed25519 cryptography and OpenSSH certificates.

The implementation includes:

- Ed25519 host key
- SSH Certificate Authority
- Signed host certificate
- Ed25519 client/user key
- Signed user certificate
- Trusted User CA
- Certificate-based authentication
- Restricted SSH configuration

---

# 13. SSH Certificate Authority

A dedicated SSH Certificate Authority was created using Ed25519.

Example:

```bash
ssh-keygen -t ed25519 \
  -f ssh_user_host_ca \
  -C "COIT12202 SSH CA"
```

The SSH CA fingerprint used in the laboratory was:

```text
SHA256:ZkXaKkBEWX7XE82MjeC3YV6eTDJhaILd5AQ7rIH8FsQ
```

The CA private key is secret and is therefore excluded from GitHub.

Only the public CA key is suitable for repository evidence.

---

# 14. SSH Host Certificate

An Ed25519 SSH host key was created for the server.

The server host public key was signed by the SSH CA.

Host certificate ID:

```text
web01-host-2026
```

Host certificate principals:

```text
secure-server.local
192.168.88.128
```

The certificate was verified using:

```bash
ssh-keygen -L \
  -f /etc/ssh/ssh_host_ed25519_key-cert.pub
```

The certificate showed:

```text
Type: ssh-ed25519-cert-v01@openssh.com host certificate
```

and identified the COIT12202 SSH CA as the signing authority.

---

# 15. SSH User Certificate

An Ed25519 key pair was generated on the Client VM for:

```text
securitytest
```

The public key was signed by the SSH CA.

User certificate ID:

```text
securitytest-2026
```

Principal:

```text
securitytest
```

The certificate was verified using:

```bash
ssh-keygen -L -f ~/.ssh/id_ed25519-cert.pub
```

The user certificate was successfully recognized as:

```text
ssh-ed25519-cert-v01@openssh.com user certificate
```

---

# 16. SSH Certificate Authentication Architecture

The authentication process can be represented as:

```text
                  SSH Certificate Authority
                           |
              +------------+------------+
              |                         |
              v                         v
       Host Certificate           User Certificate
              |                         |
              v                         v
      secure-server.local          securitytest
              ^                         |
              |                         |
              +---------- SSH ----------+
                         |
                         v
                    Client VM
```

The server trusts the SSH User CA.

Therefore, a valid user certificate signed by that CA and containing the appropriate principal can authenticate the intended user.

---

# 17. Successful Certificate-Based SSH Authentication

Certificate authentication was successfully tested from:

```text
Client VM: 192.168.88.130
```

to:

```text
Server VM: 192.168.88.128
```

using the `securitytest` account.

The server authentication log confirmed:

```text
Accepted publickey for securitytest
```

and identified:

```text
ED25519-CERT
ID securitytest-2026
```

This provides strong evidence that authentication was performed using the signed SSH certificate.

---

# 18. SSH Hardening Configuration

The SSH server was configured with security-focused settings such as:

```text
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
PermitRootLogin no

HostKey /etc/ssh/ssh_host_ed25519_key
HostCertificate /etc/ssh/ssh_host_ed25519_key-cert.pub

TrustedUserCAKeys /etc/ssh/user_ca.pub

X11Forwarding no
AllowAgentForwarding no
PermitEmptyPasswords no
```

These settings reduce the SSH attack surface by disabling unnecessary or weaker authentication mechanisms and trusting the configured SSH CA.

Before restarting SSH, configuration syntax was checked using:

```bash
sudo sshd -t
```

No output from this command indicates successful syntax validation.

---

# 19. Part D – Fail2Ban

Fail2Ban was configured to protect the SSH service against repeated failed login attempts.

Configuration file:

```text
/etc/fail2ban/jail.local
```

Laboratory configuration:

```ini
[DEFAULT]
bantime = 10m
findtime = 10m
maxretry = 3

[sshd]
enabled = true
port = 22
backend = systemd
```

Configuration was validated using:

```bash
sudo fail2ban-client -t
```

The configuration test returned successfully.

---

# 20. Fail2Ban Testing

Intentional failed SSH authentication attempts were generated from the Client VM:

```text
192.168.88.130
```

using an invalid account such as:

```bash
ssh wronguser@192.168.88.128
```

Fail2Ban detected the failed authentication events.

The Fail2Ban log showed entries such as:

```text
[sshd] Found 192.168.88.130
```

and subsequently:

```text
[sshd] Ban 192.168.88.130
```

The log later showed:

```text
[sshd] Unban 192.168.88.130
```

after the configured ban duration.

This demonstrates that Fail2Ban:

1. Monitored SSH authentication events
2. Detected repeated failures
3. Identified the attacking source IP
4. Applied a temporary ban
5. Automatically removed the ban after the configured period

---

# 21. Security Testing Summary

| Security Control | Test | Result |
|---|---|---|
| Root CA | CA extensions inspected | PASS |
| Intermediate CA | Verified against Root CA | PASS |
| Server Certificate | Signed by Intermediate CA | PASS |
| HTTPS Certificate Chain | Client validation | PASS |
| TLS | TLS 1.3 connection established | PASS |
| HTTPS | HTTP 200 over TLS | PASS |
| SHA-512 | Password hash format tested | PASS |
| Password Quality | PAM policy configured | PASS |
| SSH Host Key | Ed25519 | PASS |
| SSH Host Certificate | Signed by SSH CA | PASS |
| SSH User Key | Ed25519 | PASS |
| SSH User Certificate | Signed by SSH CA | PASS |
| SSH Certificate Login | `securitytest` authentication | PASS |
| Fail2Ban | Failed SSH attempts detected | PASS |
| Fail2Ban Ban | Client IP banned | PASS |

---

# 22. Repository Structure

```text
COIT12202-A1/
│
├── README.md
│
├── openssl-ca/
│   ├── README.md
│   ├── root-ca.crt
│   ├── intermediate-ca.crt
│   ├── securelab.crt
│   ├── fullchain.crt
│   ├── server-ext.cnf
│   └── commands.txt
│
├── password-hashing/
│   ├── README.md
│   ├── pwquality.conf
│   └── commands.txt
│
├── ssh-hardening/
│   ├── README.md
│   ├── sshd_config
│   ├── user_ca.pub
│   ├── host-cert.pub
│   ├── user-cert.pub
│   ├── jail.local
│   └── commands.txt
│
├── activity-evidence/
│   ├── week1/
│   ├── week2/
│   ├── week3/
│   └── week4/
│
└── screenshots/
    ├── pki-chain.png
    ├── https-verified.png
    ├── password-quality.png
    ├── password-hash-format.png
    ├── account-lockout.png
    ├── ssh-user-cert.png
    ├── ssh-host-cert.png
    └── fail2ban.png
```

---

# 23. Evidence Guide

The `screenshots/` directory contains visual evidence for the major assessment outcomes.

### PKI Chain

`pki-chain.png`

Shows certificate-chain verification between the Root CA, Intermediate CA, and server certificate.

### HTTPS Verification

`https-verified.png`

Shows successful HTTPS/TLS validation from the Client VM.

### Password Quality

`password-quality.png`

Shows the configured password complexity controls and/or password quality testing.

### Password Hash Format

`password-hash-format.png`

Shows evidence that the test account uses the expected SHA-512 password hash format without exposing sensitive hash data unnecessarily.

### Account Lockout

`account-lockout.png`

Shows account lockout configuration/testing evidence.

### SSH Host Certificate

`ssh-host-cert.png`

Shows the signed Ed25519 SSH host certificate and its principals.

### SSH User Certificate

`ssh-user-cert.png`

Shows the signed certificate for the `securitytest` user.

### Fail2Ban

`fail2ban.png`

Shows Fail2Ban detecting failed SSH authentication and banning the testing Client VM.

---

# 24. Security and Secret Management

Sensitive information has intentionally been excluded from version control.

The `.gitignore` protects common secret file types such as:

```gitignore
*.key
*.pem
*.p12
*.pfx
*.secret
*.pass
.env

id_ed25519
ssh_user_host_ca
root-ca.key
intermediate-ca.key
```

The following information must **never** be committed:

- Root CA private key
- Intermediate CA private key
- SSH CA private key
- Server private key
- Client SSH private key
- GitHub SSH private key
- Passwords
- Passphrases
- GitHub Personal Access Tokens
- `/etc/shadow`
- Full password hashes
- Environment files containing credentials

Public certificates and public keys may be included as assessment evidence.

---

# 25. Key Security Concepts Demonstrated

This laboratory demonstrates several important cybersecurity principles.

## Defence in Depth

Multiple security layers were implemented instead of relying on a single control.

```text
Strong Password Policy
        +
Certificate Authentication
        +
SSH Hardening
        +
Fail2Ban
        +
TLS/HTTPS
        =
Defence in Depth
```

## Least Privilege

The `securitytest` account was used as a normal user and was not granted unnecessary administrative privileges.

## Certificate-Based Trust

Both HTTPS and SSH use certificate-based trust relationships.

## Secure Authentication

Ed25519 SSH certificates provide stronger centralized authentication management compared with distributing unmanaged individual public keys.

## Brute-Force Protection

Fail2Ban monitors authentication failures and temporarily blocks suspicious source addresses.

## Confidentiality and Integrity

HTTPS/TLS protects network communication against unauthorized reading and modification.

---

# 26. Validation Commands

Important commands used to validate the implementation include:

```bash
# Verify Intermediate CA
openssl verify -x509_strict \
  -CAfile root-ca.crt \
  intermediate-ca.crt

# HTTPS validation
curl -v \
  --cacert /opt/pki/root-ca/certs/root-ca.crt \
  https://secure-server.local/

# SSH Server host certificate
ssh-keygen -L \
  -f /etc/ssh/ssh_host_ed25519_key-cert.pub

# SSH Client user certificate
ssh-keygen -L \
  -f ~/.ssh/id_ed25519-cert.pub

# Validate SSH configuration
sudo sshd -t

# Display effective SSH configuration
sudo sshd -T

# Fail2Ban configuration validation
sudo fail2ban-client -t

# Fail2Ban status
sudo fail2ban-client status

# SSH jail status
sudo fail2ban-client status sshd
```

---

# 27. Final Outcome

The laboratory successfully demonstrated the implementation of a multi-layer Linux security environment.

The completed environment provides:

```text
                    COIT12202 Security Lab
                              |
        +---------------------+---------------------+
        |                     |                     |
        v                     v                     v
   OpenSSL PKI          Password Security      SSH Security
        |                     |                     |
        v                     v                     v
 Root + Intermediate      SHA-512 + PAM       Ed25519 + SSH CA
        |                                           |
        v                                           v
 HTTPS / TLS                                Certificate Login
        |                                           |
        +--------------------+----------------------+
                             |
                             v
                         Fail2Ban
                             |
                             v
                    Brute-Force Protection
```

The major assessment outcomes were successfully demonstrated through configuration files, command output, authentication logs, certificates, and screenshots contained in this repository.

---

# Author

**COIT12202 Assignment 1 Security Portfolio**

Badhon Badiya - Central Queensland University (CQU)

---

# Disclaimer

This environment was created for educational and laboratory purposes.

The IP addresses, certificates, user accounts, security tests, and authentication attempts documented in this repository were performed within an isolated laboratory environment.

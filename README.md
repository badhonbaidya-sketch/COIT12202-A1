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



---

# Validation Commands

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



The major assessment outcomes were successfully demonstrated through configuration files, command output, authentication logs, certificates, and screenshots contained in this repository.

---

# Author

**COIT12202 Assignment 1 Security Portfolio**

Badhon Badiya - Central Queensland University (CQU)

---

# Disclaimer

This environment was created for educational and laboratory purposes.

The IP addresses, certificates, user accounts, security tests, and authentication attempts documented in this repository were performed within an isolated laboratory environment.

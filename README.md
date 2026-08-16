# COIT12202 Assignment 1 – Security Portfolio

## Overview

This repository contains the implementation and evidence for the
COIT12202 Assignment 1 security laboratory.

The portfolio demonstrates:

- OpenSSL Root CA and Intermediate CA
- Server certificate generation and signing
- HTTPS configuration with Nginx
- Certificate chain verification
- Ubuntu password security
- SHA-512 password hashing
- Password quality enforcement
- Account lockout testing
- OpenSSH hardening
- Ed25519 SSH keys
- SSH CA host certificates
- SSH CA user certificates
- Certificate-based SSH authentication
- Fail2Ban SSH protection

## Lab Environment

| System | Role | IP Address |
|---|---|---|
| secure-server.local | Security / Web / SSH Server | 192.168.88.128 |
| Client VM | Testing Client | 192.168.88.130 |

## Repository Structure

```text
COIT12202-A1/
├── README.md
├── openssl-ca/
├── password-hashing/
├── ssh-hardening/
├── activity-evidence/
└── screenshots/

# Post-Quantum Security Scanning Toolkit

This documentation covers the source-build setup of **OpenSSL 3.6.0** and **Nmap 7.98**. By linking Nmap against a custom OpenSSL build, we enable support for **Post-Quantum Cryptography (PQC)** testing, specifically the `X25519MLKEM768` (ML-KEM) hybrid group.

## 1. System Preparation

Update system packages and install the core dependencies required to compile both projects.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential checkinstall zlib1g-dev libssl-dev \
                     perl-modules perl-doc libpcap-dev libpcre3-dev
```

## 2. Build OpenSSL 3.6.0 (Post-Quantum Ready)

**Source:** [openssl.org](https://www.openssl.org/) | [GitHub Releases](https://github.com/openssl/openssl/releases)

Standard system OpenSSL often lacks the latest PQC identifiers. We build version 3.6.0 to support modern TLS 1.3 key exchanges.

### Installation:

```bash
wget https://github.com/openssl/openssl/releases/download/openssl-3.6.0/openssl-3.6.0.tar.gz
tar -xzf openssl-3.6.0.tar.gz
cd openssl-3.6.0/

./config --prefix=/usr/local --openssldir=/usr/local/ssl
make -j$(nproc)
sudo make install
```

### Environment Configuration:

To ensure the system uses the new OpenSSL version, add these to your `~/.bashrc`:

```bash
export PATH=/usr/local/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/lib64:$LD_LIBRARY_PATH
source ~/.bashrc
```

## 3. Build Nmap 7.98 (PQC-Enabled)

**Source:** [nmap.org/dist/](https://nmap.org/dist/)

We bypass `apt install nmap` and build from source to link it directly against the custom OpenSSL library at `/usr/local`.

### Installation:

```bash
wget https://nmap.org/dist/nmap-7.98.tar.bz2 --no-check-certificate
tar -xvjf nmap-7.98.tar.bz2
cd nmap-7.98/

# Pointing to our custom OpenSSL build
./configure --with-openssl=/usr/local
make -j$(nproc)
sudo make install
```

### Environment Configuration for Nmap:

If `nmap` is not immediately recognized or you want to ensure the direct path to the new binary, add the following to your `~/.bashrc`:

```bash
# Direct access to locally built Nmap
export PATH=/usr/local/bin/nmap:$PATH
source ~/.bashrc
```

## 4. Usage & Auditing Commands

### Verifying PQC Support (OpenSSL)

Check if a site supports the hybrid Post-Quantum ML-KEM exchange:

```bash
# Verified success on Cloudflare (ngkorefoundation.com)
openssl s_client -connect ngkorefoundation.com:443 -tls1_3 -groups X25519MLKEM768

# Test fallback for banks (Axis Bank example)
openssl s_client -connect www.axis.bank.in:443 -tls1_2
```

### Advanced Nmap Auditing

| Objective                    | Command                                                                                  |
| ---------------------------- | ---------------------------------------------------------------------------------------- |
| **PQC & Cipher Scan**        | `nmap --script ssl-enum-ciphers -p 443 <target>`                                         |
| **Certificate Info**         | `nmap --script ssl-cert -p 443 <target>`                                                 |
| **Full Vulnerability Check** | `nmap --script ssl-cert-intaddr,ssl-heartbleed,ssl-dh-params,ssl-poodle -p 443 <target>` |
| **Security Headers**         | `nmap --script http-security-headers -p 443 <target>`                                    |
| **Service Versioning**       | `nmap -sV -p 443 <target>`                                                               |

## 5. Summary of Key Locations

- **OpenSSL Binary:** `/usr/local/bin/openssl`
- **Nmap Binary:** `/usr/local/bin/nmap`
- **Nmap Scripts (NSE):** `/usr/local/share/nmap/scripts/`

> [!NOTE]
> Always ensure your `PATH` prioritizes `/usr/local/bin` to avoid using the older `/usr/bin/` system defaults.

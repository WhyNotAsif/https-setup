# FastAPI HTTPS Setup on Oracle VPS (Free) using Caddy + DuckDNS

This guide explains how to host a FastAPI application with free HTTPS SSL on an Oracle Cloud Ubuntu VPS using:

- FastAPI
- Uvicorn
- Caddy
- DuckDNS

No paid domain required.

---

# Final Result

Your API:

```bash
http://localhost:8001
```

Becomes:

```bash
https://yourdomain.duckdns.org
```

with automatic HTTPS SSL.

---

# Requirements

- Oracle Cloud Free Tier VPS
- Ubuntu Server
- Python + FastAPI installed
- FastAPI app already running
- Free DuckDNS account

---

# 1. Run FastAPI

Start your FastAPI application:

```bash
uvicorn main:app --host 0.0.0.0 --port 8001
```

Example:

```bash
uvicorn main:app --host 0.0.0.0 --port 8001
```

Test locally:

```bash
curl http://localhost:8001
```

---

# 2. Create Free DuckDNS Domain

Open:

https://www.duckdns.org

Login and create a subdomain.

Example:

```bash
myfastapi.duckdns.org
```

Add your Oracle VPS public IP.

Example:

```bash
161.33.130.201
```

Save/update.

---

# 3. Verify Domain Points to VPS

Run:

```bash
ping myfastapi.duckdns.org
```

Expected:

```bash
PING myfastapi.duckdns.org (161.33.130.201)
```

---

# 4. Open Oracle Cloud Firewall Ports

Oracle Cloud blocks ports by default.

Go to:

```text
Oracle Cloud
→ Networking
→ Virtual Cloud Networks
→ Your VCN
→ Security Lists
→ Default Security List
→ Add Ingress Rule
```

Add these rules:

---

## Rule 1

| Field | Value |
|---|---|
| Source CIDR | `0.0.0.0/0` |
| IP Protocol | `TCP` |
| Destination Port Range | `80` |

---

## Rule 2

| Field | Value |
|---|---|
| Source CIDR | `0.0.0.0/0` |
| IP Protocol | `TCP` |
| Destination Port Range | `443` |

Save both rules.

---

# 5. Install Caddy

Update packages:

```bash
sudo apt update
```

Install required dependencies:

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
```

Add Caddy GPG key:

```bash
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' \
| sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
```

Add Caddy repository:

```bash
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' \
| sudo tee /etc/apt/sources.list.d/caddy-stable.list
```

Install Caddy:

```bash
sudo apt update
sudo apt install caddy -y
```

---

# 6. Configure Caddy

Open Caddy configuration:

```bash
sudo nano /etc/caddy/Caddyfile
```

Remove everything inside and paste:

```caddy
myfastapi.duckdns.org {
    reverse_proxy localhost:8001
}
```

Replace:

```text
myfastapi.duckdns.org
```

with your actual DuckDNS domain.

Save file:

```text
CTRL + X
Y
ENTER
```

---

# 7. Restart Caddy

```bash
sudo systemctl restart caddy
```

---

# 8. Verify Caddy Status

```bash
sudo systemctl status caddy
```

Expected:

```text
active (running)
```

---

# 9. Test HTTPS

Open:

```bash
https://myfastapi.duckdns.org
```

FastAPI docs:

```bash
https://myfastapi.duckdns.org/docs
```

You should now have:

- Free HTTPS
- Automatic SSL certificates
- Auto-renewal
- Permanent domain
- Reverse proxy setup

---

# Architecture

```text
Internet
   ↓
HTTPS (DuckDNS Domain)
   ↓
Caddy Reverse Proxy
   ↓
localhost:8001
   ↓
FastAPI (Uvicorn)
```

---

# Useful Commands

## Restart Caddy

```bash
sudo systemctl restart caddy
```

## Check Caddy Status

```bash
sudo systemctl status caddy
```

## View Caddy Logs

```bash
journalctl -u caddy --no-pager --lines=50
```

## Check Domain IP

```bash
ping myfastapi.duckdns.org
```

---

# Optional: Run FastAPI in Background

Install screen:

```bash
sudo apt install screen -y
```

Create session:

```bash
screen -S fastapi
```

Run app:

```bash
uvicorn main:app --host 0.0.0.0 --port 8001
```

Detach screen:

```text
CTRL + A
then D
```

Reconnect later:

```bash
screen -r fastapi
```

---

# Notes

- Works on Oracle Free Tier VPS
- Works on low RAM servers (512MB)
- No paid domain required
- No Certbot required
- No Nginx required
- Caddy automatically handles SSL certificates

---

# Stack Used

- FastAPI
- Uvicorn
- Caddy
- DuckDNS
- Oracle Cloud Ubuntu VPS

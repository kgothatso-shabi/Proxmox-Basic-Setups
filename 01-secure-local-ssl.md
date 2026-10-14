# Guide 01: Securing Proxmox Web UI with a Trusted Local CA (SSL)

By default, Proxmox VE uses a self-signed SSL certificate, which triggers browser security warnings (`"Your connection is not private"`). 

If your Proxmox server runs on a private local network and you don't want to expose it to the public internet via a public domain, you can set up a **Local Certificate Authority (CA)** using `mkcert`. This gives you a fully trusted green padlock in your browser without security exceptions.

---

## Prerequisites
* A client computer (Windows, macOS, or Linux) used to manage Proxmox.
* Proxmox installed with a local hostname (e.g., `pve.home.local`) and local IP address.

---

## Step 1: Install `mkcert` on Your Local Management Machine
Install `mkcert` on the computer you use to access your Proxmox dashboard:

* **Windows (via PowerShell/Winget):**
  ```powershell
  winget install FiloSottile.mkcert

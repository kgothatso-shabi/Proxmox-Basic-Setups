# Guide 01: Securing Proxmox Web UI with a Trusted Local CA (SSL)

By default, Proxmox VE uses a self-signed SSL certificate, which triggers browser security warnings such as **"Your connection is not private."**

If your Proxmox server runs on a private local network and you don't want to expose it to the public internet via a public domain, you can set up a **Local Certificate Authority (CA)** using `mkcert`. This allows your browser to trust the Proxmox certificate without requiring you to bypass security warnings.

---

## Prerequisites

- A client computer running Windows, macOS, or Linux to manage Proxmox.
- Proxmox VE installed with a local hostname (e.g., `pve.home.local`) and a local IP address.

---

## Step 1: Install `mkcert` on Your Local Management Machine

Install `mkcert` on the computer you use to access your Proxmox dashboard.

### Windows (PowerShell/Winget)

```powershell
winget install FiloSottile.mkcert
```

### Linux (Ubuntu/Debian)

```bash
sudo apt install libnss3-tools

wget -O mkcert https://github.com/FiloSottile/mkcert/releases/download/v1.4.4/mkcert-v1.4.4-linux-amd64

chmod +x mkcert
sudo mv mkcert /usr/local/bin/
```

### macOS (Homebrew)

```bash
brew install mkcert
```

---

## Step 2: Initialize the Local CA Trust Store

Run the following command once to create your local CA and register it with your system and supported browser trust stores.

```bash
mkcert -install
```

---

## Step 3: Generate the SSL Certificate Files

Generate the certificate and private key for your Proxmox server. Replace the placeholder hostname and IP address with your actual Proxmox details.

```bash
mkcert pve.yourdomain.local 192.168.X.X
```

This generates two files in your current working directory:

1. `pve.yourdomain.local+1.pem` — The certificate.
2. `pve.yourdomain.local+1-key.pem` — The private key.

**Important:** Generate the certificate on the management computer where you installed `mkcert`. The certificate must include the exact hostname and IP address you intend to use when accessing Proxmox.

---

## Step 4: Upload the Certificate to Proxmox

1. Log in to the Proxmox Web UI at `https://192.168.X.X:8006`.
2. In the left-hand menu, click your **Proxmox node**.
3. Expand **System** and select **Certificates**.
4. Click **Upload Custom Certificate**.
5. Open the generated `.pem` certificate file in a text editor.
6. Copy its entire contents and paste them into the **Certificate File** field.
7. Open the generated `-key.pem` private key file in a text editor.
8. Copy its contents and paste them into the **Private Key** field.
9. Click **Upload**.

**Security note:** Keep the private key confidential. Do not share it or upload it to public websites.

---

## Step 5: Verify the Secure Connection

1. Refresh the Proxmox Web UI or restart your browser.
2. Access Proxmox using the hostname or IP address included in the certificate.
3. Confirm that the browser no longer displays certificate security warnings.

If the browser still shows a warning, verify that:

- The local CA is trusted by the client computer and browser.
- The certificate includes the hostname or IP address used to access Proxmox.
- The correct certificate and matching private key were uploaded.
- The browser is accessing the server using the exact hostname or IP address included in the certificate.

---

## Expected Result

Your browser should recognize the Proxmox SSL certificate as trusted, allowing you to access the Web UI without the usual self-signed certificate warning.

**Note:** A trusted certificate encrypts the connection and establishes certificate trust. It does not, by itself, secure Proxmox against unauthorized access or replace other security measures.
---

## Step 6: Renew the SSL Certificate

Certificates generated with `mkcert` are valid for a limited period. When the certificate expires, browsers may display security warnings again.

To renew the certificate, generate a new certificate and private key, then upload them to Proxmox.

### 6.1: Check the Certificate Expiry Date

On your management computer, open a terminal in the directory containing your existing certificate and run:

```bash
openssl x509 -in pve.yourdomain.local+1.pem -noout -dates
```

Example output:

```text
notBefore=Oct 14 10:00:00 2026 GMT
notAfter=Jan 12 10:00:00 2027 GMT
```

The `notAfter` value indicates when the certificate expires.

### 6.2: Generate a Replacement Certificate

Run `mkcert` again, specifying the same hostname and IP address used for the original certificate.

```bash
mkcert pve.yourdomain.local 192.168.X.X
```

Replace `pve.yourdomain.local` and `192.168.X.X` with your actual Proxmox hostname and IP address.

The command generates a new certificate and private key. Depending on the existing files in the directory, the filenames may have a different numeric suffix.

**Important:** Keep the new certificate and private key together. Ensure you upload the matching pair.

### 6.3: Upload the Replacement Certificate to Proxmox

1. Log in to the Proxmox Web UI.
2. Select your **Proxmox node**.
3. Navigate to **System → Certificates**.
4. Select **Upload Custom Certificate**.
5. Select the newly generated certificate file (`.pem`).
6. Select its matching private key file (`-key.pem`).
7. Upload the replacement certificate.

Proxmox should begin using the replacement certificate after the upload and the required service reload.

### 6.4: Verify the Renewal

Refresh the Proxmox Web UI and confirm that the browser no longer displays certificate warnings.

You can also check the new certificate's validity dates:

```bash
openssl x509 -in pve.yourdomain.local+1.pem -noout -dates
```

Confirm that the `notAfter` date is in the future.

### 6.5: Renewal Notes

- **Do not run `mkcert -install` every time you renew.** Run it again only if you need to install or restore the local CA trust on the management computer.
- **Keep the original CA intact.** If you use the same local CA, the new certificate should remain trusted by computers that already trust that CA.
- **Back up your CA carefully.** The CA's root certificate and private key are normally stored in the `mkcert` certificate directory. Protect the CA private key because anyone who obtains it could issue certificates trusted by your local devices.
- **Renew before expiry.** Check the expiry date periodically and replace the certificate in advance.
- **Verify the active certificate.** Checking the local `.pem` file confirms its validity dates, but you should also verify the certificate actually presented by Proxmox after uploading.


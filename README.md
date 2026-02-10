# Azure Email Server - One-Click Deploy

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fakingscote%2Fazure-email-server%2Fmain%2Fazuredeploy.json)

Deploy a lightweight Ubuntu VM with a fully configured email server (Postfix + Dovecot) and web-based email client (Roundcube) in a single click.

## What Gets Deployed

| Resource | Details |
|----------|---------|
| **VM** | Ubuntu 22.04 LTS, Standard_B1s (lightweight) |
| **Public IP** | Dynamic, with DNS label matching the VM name |
| **NSG** | SSH (22), HTTP (80), HTTPS (443), SMTP (25), Submission (587), IMAP (143) |
| **Mail Server** | Postfix (SMTP) + Dovecot (IMAP/LMTP) |
| **Mail Client** | Roundcube Webmail for the VM admin user |

## Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `vmName` | VM name | — |
| `adminUsername` | SSH admin user | `azureuser` |
| `adminPassword` | Password for VM and email user | — |
| `vmSize` | Azure VM size | `Standard_B1s` |

## How It Works

1. Click the **Deploy to Azure** button above.
2. Fill in the required parameters (VM name, password).
3. Wait for the deployment to complete (~5-10 minutes).
4. Access Roundcube webmail at: `http://mail-<unique-id>.<region>.cloudapp.azure.com`

## Login Credentials

| Service | Username | Password |
|---------|----------|----------|
| **SSH** | Value of `adminUsername` (default: `azureuser`) | Your `adminPassword` |
| **Roundcube Webmail** | Same as `adminUsername` (default: `azureuser`) | Your `adminPassword` |

The VM admin user is also the email user. Emails are addressed to `<adminUsername>@<public-dns>`.

## Architecture

```
┌──────────────────────────────────────────┐
│              Ubuntu 22.04 VM             │
│                                          │
│  ┌──────────┐  ┌─────────┐  ┌────────┐  │
│  │ Postfix  │  │ Dovecot │  │ Apache │  │
│  │ (SMTP)   │──│ (IMAP)  │──│  +RC   │  │
│  │ :25/:587 │  │  :143   │  │  :80   │  │
│  └──────────┘  └─────────┘  └────────┘  │
│                                          │
│  User: <adminUsername> (login via Roundcube)  │
└──────────────────────────────────────────┘
         │
    Public IP + DNS
    <vmname>.<region>.cloudapp.azure.com
```

## Email Addresses

- **Inbox**: `<adminUsername>@<public-dns>`
- **Send/Receive**: Fully functional via Roundcube webmail

## Services (Auto-Start on Reboot)

All services are enabled via `systemctl enable` and will start automatically on reboot:

- `postfix` — SMTP mail server
- `dovecot` — IMAP server with LMTP delivery
- `apache2` — Web server for Roundcube

## Post-Deployment

### Check setup logs
```bash
ssh azureuser@<fqdn>
sudo cat /var/log/email-setup.log
```

### Verify services
```bash
sudo systemctl status postfix dovecot apache2
```

### Send a test email
```bash
echo "Test" | mail -s "Test Email" ashley@<your-domain>
```

## Important Notes

- **Azure blocks outbound port 25** on most subscriptions by default. You may need to [request removal of the restriction](https://learn.microsoft.com/en-us/azure/virtual-network/troubleshoot-outbound-smtp-connectivity) to send emails externally.
- **DNS**: For external email delivery, configure MX records for your domain pointing to the VM's public IP/FQDN.
- **Security**: The Roundcube webmail has no authentication. This is intentional per requirements but should only be used in controlled environments.
- **TLS**: Uses self-signed certificates by default. For production, configure Let's Encrypt.

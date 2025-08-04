# BOWA VPS Setup - Website Deployment with NGINX and SSL

This repository contains a GitHub Actions workflow that automatically:
- 🔧 Installs NGINX if not present
- 📁 Creates website directories at `/var/www/${WEBSITE_FOLDER}`
- 🔒 Sets up Let's Encrypt SSL certificates
- ⚙️ Configures NGINX with security headers and optimizations

## Prerequisites

### 1. VPS Server Requirements s
- Ubuntu/Debian-based Linux server
- Root or sudo access
- Domain name pointing to your server's IP address
- SSH access enabled

### 2. GitHub Repository Setup

#### Required GitHub Secrets
You need to add the following secret to your GitHub repository:

1. Go to your repository → Settings → Secrets and variables → Actions
2. Add the following secret:

| Secret Name | Description | Example |
|-------------|-------------|---------|
| `SSH_PRIVATE_KEY` | Private SSH key for server access | Your RSA private key content |

#### Generating SSH Key Pair
If you don't have an SSH key pair:

```bash
# Generate new SSH key pair
ssh-keygen -t rsa -b 4096 -C "github-actions@yourdomain.com"

# Copy public key to your server
ssh-copy-id -i ~/.ssh/id_rsa.pub root@your-server-ip

# Copy private key content for GitHub secret
cat ~/.ssh/id_rsa
```

## Usage

### Running the Workflow

1. Go to your repository → Actions tab
2. Select "Deploy Website with NGINX and SSL" workflow
3. Click "Run workflow"
4. Fill in the required inputs:

| Input | Description | Example |
|-------|-------------|---------|
| `website_folder` | Folder name for your website | `my-awesome-site` |
| `domain_name` | Your domain name | `example.com` |
| `email` | Email for Let's Encrypt | `admin@example.com` |
| `server_host` | Server IP or hostname | `192.168.1.100` |
| `server_user` | SSH username | `root` |

5. Click "Run workflow"

### What the Workflow Does

#### Step 1: NGINX Installation
- Checks if NGINX is installed
- If not present, installs and configures NGINX
- Enables and starts the NGINX service

#### Step 2: Website Directory Creation
- Creates `/var/www/${WEBSITE_FOLDER}` directory
- Sets proper ownership (`www-data:www-data`)
- Sets appropriate permissions (755)
- Creates a default `index.html` file

#### Step 3: Let's Encrypt Setup
- Installs Certbot if not present
- Creates SSL certificate for your domain
- Configures automatic certificate renewal
- Sets up a cron job for renewal

#### Step 4: NGINX Configuration
- Creates optimized NGINX site configuration
- Includes security headers
- Enables file caching
- Links configuration to sites-enabled

#### Step 5: Verification
- Verifies NGINX is running
- Checks website directory exists
- Validates NGINX configuration
- Confirms SSL certificate is active

## File Structure After Deployment

```
/var/www/
└── ${WEBSITE_FOLDER}/
    └── index.html

/etc/nginx/
├── sites-available/
│   └── ${DOMAIN_NAME}
└── sites-enabled/
    └── ${DOMAIN_NAME} -> ../sites-available/${DOMAIN_NAME}

/etc/letsencrypt/
└── live/
    └── ${DOMAIN_NAME}/
        ├── cert.pem
        ├── chain.pem
        ├── fullchain.pem
        └── privkey.pem
```

## Security Features

- **Security Headers**: X-Frame-Options, X-XSS-Protection, Content-Security-Policy
- **SSL/TLS Encryption**: Automatic HTTPS redirect
- **File Protection**: Denies access to sensitive files
- **Automatic Renewal**: SSL certificates auto-renew via cron job

## Troubleshooting

### Common Issues

1. **SSH Connection Failed**
   - Verify SSH key is correct in GitHub secrets
   - Check server IP address
   - Ensure SSH service is running on server

2. **Domain Not Accessible**
   - Verify domain DNS points to server IP
   - Check firewall allows ports 80 and 443
   - Ensure NGINX is running: `systemctl status nginx`

3. **SSL Certificate Failed**
   - Verify domain is accessible via HTTP first
   - Check email address is valid
   - Ensure domain DNS is properly configured

### Manual Verification

Connect to your server and run:

```bash
# Check NGINX status
systemctl status nginx

# Check website directory
ls -la /var/www/

# Check SSL certificate
certbot certificates

# Test NGINX configuration
nginx -t
```

## Customization

### Adding More Security
You can enhance the NGINX configuration by editing the workflow to include:
- Rate limiting
- IP whitelisting
- Additional security headers
- Custom error pages

### Multiple Websites
Run the workflow multiple times with different inputs to deploy multiple websites on the same server.

## Support

If you encounter issues:
1. Check the GitHub Actions logs for detailed error messages
2. Verify all prerequisites are met
3. Test SSH connection manually
4. Ensure domain DNS is configured correctly

---

**Note**: This workflow requires a VPS server with root access. It's designed for Ubuntu/Debian systems and may need modifications for other Linux distributions.
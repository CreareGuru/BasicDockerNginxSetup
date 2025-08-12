# Set up SSL & Redirect with NGINX and Certbot

This guide will help you set up SSL with NGINX using Certbot to automatically obtain and configure SSL certificates for your domain.

**Important:** Before proceeding, ensure that the DNS A record for your domain (e.g., `example.domain.com`) is correctly set up and propagating. The domain should point to your server's IP address for Certbot to verify and issue the SSL certificate.

| Type	| Host	| Value	| TTL |
|---|---|---|---|
| A	| example.domain.com	| 192.168.1.100	| 3600 |

As a side note, this was done on Ubuntu 22

---

## 1.1. Install UFW (Uncomplicated Firewall)

First, ensure that UFW is installed to manage firewall rules on your server.

```bash
sudo apt install ufw
```

## 1.2. Open the required ports on your firewall

Add or remove any specific ports that you may or may not need.

```bash
sudo ufw allow 22/tcp #SSH Port
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw reload
sudo ufw status
```
---

## 2.1 Install NGINX

NGINX will act as the reverse proxy for your web application. Install it with the following command:

```bash
sudo apt install nginx
```

## 2.2 Ensure a Valid Nginx Server Block Exists for example.domain.com

The first step is to create a server block configuration for your domain. This is similar to a virtual host configuration in Apache.
Create a configuration file for your site:

```bash
sudo nano /etc/nginx/sites-available/example.domain.com
```

## 2.3 In this file, paste the following basic configuration (adjust as needed):

```nginx
server {
    listen 80;
    server_name example.domain.com;

    location / {
        proxy_pass http://localhost:5678; # <-- the port or url you are trying to point to
        proxy_http_version 1.1;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

```

This config sets up the basic reverse proxy. It listens on port 80 (HTTP) for requests to example.domain.com and forwards them to a local application running on port 5678.

Explanation of the proxy headers:
- proxy_set_header Host $host; passes the original host header.
- proxy_set_header X-Real-IP $remote_addr; forwards the real client IP.
- proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; adds the forwarding header.
- proxy_set_header X-Forwarded-Proto $scheme; ensures that the protocol (HTTP or HTTPS) is forwarded.

Save and exit the file.

---

## 2.4 Enable the Site with a Symbolic Link

To enable the site, create a symbolic link in the sites-enabled directory:

```bash
sudo ln -s /etc/nginx/sites-available/example.domain.com /etc/nginx/sites-enabled/
```
This tells NGINX to include the configuration for your domain.

Test the NGINX configuration to ensure there are no errors:

```bash
sudo nginx -t
```

If everything is correct, reload NGINX to apply the changes:

```bash
sudo systemctl reload nginx
```

---

## 3.1 Install Certbot

Now that NGINX is configured, the next step is to secure your site with SSL using Certbot.

Install Certbot and the NGINX plugin (if not installed already):
```bash
sudo apt install python3-certbot-nginx
```

## 3.2 Run Certbot to obtain and install SSL Certificate

```bash
sudo certbot --nginx -d example.domain.com
```
Certbot will prompt you through the setup process, including agreeing to terms and selecting whether to redirect all HTTP traffic to HTTPS.

What happens during this process:
- Certbot will obtain a free SSL certificate from Let’s Encrypt.
- Certbot will automatically modify your NGINX configuration to enable HTTPS (port 443).
- The NGINX service will be reloaded to apply the changes.

---

## 4 Test HTTPS and your redirect

After Certbot finishes, you can test if SSL is working correctly by visiting your site in a browser.
```bash
https://example.domain.com
```
If everything is set up correctly, your site should now load securely over HTTPS with a valid SSL certificate.

---

# Recap

- [x] You installed UFW, NGINX, and Certbot.
- [x] You configured NGINX to act as a reverse proxy for your domain.
- [x] You used Certbot to automatically secure your site with SSL certificates.
- [x] Your site is now accessible via HTTPS and is secure.

[![Buy me a coffee](https://img.buymeacoffee.com/button-api/?text=Buy%20me%20a%20coffee&emoji=&slug=wynandnel&button_colour=FFDD00&font_colour=000000&font_family=Cookie&outline_colour=000000&coffee_colour=ffffff)](https://www.buymeacoffee.com/wynandnel)

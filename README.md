# BasicDockerNginxSetup
Basic step by step setup for Nginx and Docker Containers


# Set up SSL & Redirect with NGINX and Certbot

This guide will help you set up SSL with NGINX using Certbot to automatically obtain and configure SSL certificates for your domain.

---

### 1. Install UFW (Uncomplicated Firewall)

First, ensure that UFW is installed to manage firewall rules on your server.

```bash
sudo apt install ufw
2. Install NGINX
NGINX will act as the reverse proxy for your web application. Install it with the following command:

bash
Copy
Edit
sudo apt install nginx
Alternatively, you can build the stack directly inside Portainer if you're using Docker.

🔹 Step 2: Access the UI
Open port 81 on your firewall to access the NGINX UI.

bash
Copy
Edit
sudo ufw allow 81/tcp
sudo ufw reload
This ensures that the NGINX service can be accessed through the firewall.

✅ 1. Ensure a Valid Nginx Server Block Exists for example.domain.com
The first step is to create a server block configuration for your domain. This is similar to a virtual host configuration in Apache.

Create a configuration file for your site:

bash
Copy
Edit
sudo nano /etc/nginx/sites-available/example.domain.com
In this file, paste the following basic configuration (adjust as needed):

nginx
Copy
Edit
server {
    listen 80;
    server_name example.domain.com;

    location / {
        proxy_pass http://localhost:5678;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
This config sets up the basic reverse proxy. It listens on port 80 (HTTP) for requests to example.domain.com and forwards them to a local application running on port 5678.

Explanation of the proxy headers:

proxy_set_header Host $host; passes the original host header.

proxy_set_header X-Real-IP $remote_addr; forwards the real client IP.

proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; adds the forwarding header.

proxy_set_header X-Forwarded-Proto $scheme; ensures that the protocol (HTTP or HTTPS) is forwarded.

Save and exit the file.

✅ 2. Enable the Site
To enable the site, create a symbolic link in the sites-enabled directory:

bash
Copy
Edit
sudo ln -s /etc/nginx/sites-available/example.domain.com /etc/nginx/sites-enabled/
This tells NGINX to include the configuration for your domain.

Test the NGINX configuration to ensure there are no errors:

bash
Copy
Edit
sudo nginx -t
If everything is correct, reload NGINX to apply the changes:

bash
Copy
Edit
sudo systemctl reload nginx
✅ 3. Run Certbot to Obtain SSL Certificate
Now that NGINX is configured, the next step is to secure your site with SSL using Certbot.

Install Certbot and the NGINX plugin (if not installed already):

bash
Copy
Edit
sudo apt install python3-certbot-nginx
Next, run Certbot to automatically obtain and install an SSL certificate:

bash
Copy
Edit
sudo certbot --nginx -d example.domain.com
Certbot will prompt you through the setup process, including agreeing to terms and selecting whether to redirect all HTTP traffic to HTTPS.

What happens during this process:

Certbot will obtain a free SSL certificate from Let’s Encrypt.

Certbot will automatically modify your NGINX configuration to enable HTTPS (port 443).

The NGINX service will be reloaded to apply the changes.

✅ 4. Test HTTPS
After Certbot finishes, you can test if SSL is working correctly by visiting your site in a browser.

Navigate to:

arduino
Copy
Edit
https://example.domain.com
If everything is set up correctly, your site should now load securely over HTTPS with a valid SSL certificate.

🔑 Recap
You installed UFW, NGINX, and Certbot.

You configured NGINX to act as a reverse proxy for your domain.

You used Certbot to automatically secure your site with SSL certificates.

Your site is now accessible via HTTPS and is secure.

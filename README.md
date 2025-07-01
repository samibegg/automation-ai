Guide: Install n8n with Nginx & HTTPS on Debian 12This guide provides all the commands needed to install the n8n workflow automation tool on a Debian 12 server, configure Nginx as a secure reverse proxy, and enable HTTPS with a free SSL certificate from Let's Encrypt using Certbot.Prerequisite 1: Configure Your DNSIMPORTANT: Before starting, you must point your domain to your server's IP address. The HTTPS certificate process will fail otherwise.Find the External IP address of your Google Compute instance.Go to your DNS provider's control panel (e.g., Google Domains, Cloudflare, GoDaddy).Create an A record for the domain you will use (e.g., n8n.your_domain.com).Point this A record to the External IP address of your server.Note: DNS changes can take some time to become active globally. You can use a tool like whatsmydns.net to check if your A record has propagated.Prerequisite 2: Configure Google Cloud FirewallCRITICAL: Google Cloud Platform has its own network firewall. You must allow HTTP and HTTPS traffic to your instance.Go to the VPC network -> Firewall page in the Google Cloud Console.Click CREATE FIREWALL RULE.Name: Give it a memorable name, like allow-http-https.Targets: Apply the rule to your specific VM instance using a network tag (e.g., webserver). Ensure your VM has this tag.Source filter: Set to IPv4 ranges.Source IPv4 ranges: Enter 0.0.0.0/0 to allow traffic from any IP address.Protocols and ports:Select Specified protocols and ports.Check the box for tcp and enter the ports 80, 443.Click Create.Prerequisite 3: Set Your Domain and EmailRun the following commands in your terminal to set your domain name and email address as shell variables.Replace your_domain.com and you@example.com with your actual information.DOMAIN="your_domain.com"
EMAIL="you@example.com"

# Sanity check: Echo the variables to ensure they are set correctly.
echo "My domain is: $DOMAIN"
echo "My email is: $EMAIL"
Step 1: Update System and Install Prerequisitessudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install -y curl wget gnupg
Step 2: Install Node.js v20curl -fsSL [https://deb.nodesource.com/setup_20.x](https://deb.nodesource.com/setup_20.x) | sudo -E bash -
sudo apt-get install -y nodejs
Step 3: Install n8n and pm2sudo npm install n8n -g
sudo npm install pm2 -g
Step 4: Start n8n and Configure for Auto-Startup# Start n8n
pm2 start n8n

# Save the current process list
pm2 save

# Generate and configure the startup script
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u $USER --hp /home/$USER
Step 5: Install Nginx Web Serversudo apt-get install -y nginx
Step 6: Configure Nginx for Your Domainsudo tee /etc/nginx/sites-available/n8n > /dev/null <<EOF
server {
    listen 80;
    server_name $DOMAIN;

    location / {
        proxy_pass http://localhost:5678;
        proxy_set_header Connection '';
        proxy_http_version 1.1;
        chunked_transfer_encoding off;
        proxy_buffering off;
        proxy_cache off;
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto \$scheme;
    }
}
EOF
Step 7: Enable the Nginx Site and Restartsudo ln -s -f /etc/nginx/sites-available/n8n /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
Step 8: Configure Local Firewall (UFW)sudo apt-get install -y ufw
sudo ufw allow ssh
sudo ufw allow 'Nginx Full'
yes | sudo ufw enable
Step 9: Install Certbot and Obtain SSL Certificatesudo apt-get install -y certbot python3-certbot-nginx
sudo certbot --nginx -d $DOMAIN --agree-tos --email $EMAIL --redirect --non-interactive
✅ Installation Complete!Your n8n instance is now installed, running, and accessible securely at https://your_domain.com. Certbot will automatically handle SSL certificate renewals.

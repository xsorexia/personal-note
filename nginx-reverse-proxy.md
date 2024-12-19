# Nginx reverse proxy (subdomain) setup

> This document will guide you through the process for setting up an Nginx web server reverse proxy. It allows you to set up multiple subdomains from a single web server. This might save your ass later in life.

**ver 0.1** | 2024.12.19 | xsorexia


### Set up reverse proxy
**Set up Lightsail Subdomain**
> This process may vary if you aren't using Lightsail to manage your DNS.
1. In **Lightsail > Domains**, create DNS zone to register existing domain names to your AWS account (if not already).
2. From the selected domain (DNS zone), add an **A Record** with the name of the subdomain, and set **Route traffic to** to the static IP.

**Configure Nginx**
1. Create a directory in **/var/www/** to place documents.
   ```bash
   sudo mkdir /var/www/folder_name
   ```
2. Create a document called *folder_name* and open it in a text editor.
   ```bash
   sudo nano /etc/nginx/sites-available/folder_name
   ```
3. If you are using PHP, run the following command to check the PHP version and edit the Step 4's configuration.
   ```bash
   php -v
   ```
4. Copy and paste the following configuration.
   ```
   server {
    listen 80;
    server_name subdomain_name.xsorexia.com;
    root /var/www/folder_name;

    index index.php index.html;

    # If you’re not using PHP you can omit this block
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }
   }
   ```
5. Run the following command to activate the configuration.
   ```bash
   sudo ln -s /etc/nginx/sites-available/folder_name /etc/nginx/sites-enabled
   ```
6. Run the following command to check for syntax errors.
   ```bash
   sudo nginx -t
   ```
7. If no errors are found, restart nginx.
   ```bash
   sudo systemctl restart nginx
   ```

### Set up SSL (HTTPS)
**Set up using Certbot**
1. If not installed already, install certbot.
   ```bash
   sudo apt update
   sudo apt install certbot python3-certbot-nginx
   ```
2. Run the following command.
   ```bash
   sudo certbot --nginx
   ```
3. Access your website through HTTPS.
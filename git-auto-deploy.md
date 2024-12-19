# Git Auto Deploy

> This document will guide you through the process for creating an automated workflow to deploy your websites to Nginx web server after each commit. This might save your ass later in life.

**ver 0.1** | 2024.12.19 | xsorexia


### Configure your server
**Create SSH key pair**
1. Check for existing SSH keys on your instance. If *id_rsa* file exists, we can skip to step 3.
    ```bash
    ls ~/.ssh
    ```
2. If there's no key pair, generate a new one. Press **Enter** to save it in the default location. You don't need to set a passphrase.
   ```bash
   ssh-keygen -t rsa -b 4096 -C "your-email@example.com"
   ```
3. Copy the public key
   ```bash
   cat ~/.ssh/id_rsa.pub
   ```

**Configure the SSH key pair with GitHub**
1. Navigate to **Setings > SSH and GPG keys**, and click **New SSH Key**.
2. Paste the copied public key value into the **Key** field, and add the key.
3. Test the connection.
   ```bash
   ssh -T git@github.com
   ```
   If successful, the following response will be printed out.
   ```
   Hi your-username! You've successfully authenticated, but GitHub does not provide shell access.
   ```

**Install and initialize git on your instance** (if not already)
1. Run the following commands.
   ```bash
   sudo apt update
   sudo apt install git
   ```
2. Initialize git repository on your instance.
   ```bash
   git init
   git remote add origin git@github.com:your-username/your-repository.git
   ```

### Set up auto-deployment
**Set up GitHub Actions**
1. Create a GitHub Actions Workflow file, and commit the changes. Save it in **/.github/workflows/deploy.yml**.
    ```yaml
      name: Deploy to Lightsail

      on:
      push:
         branches:
            - main # Adjust this if your deployment branch is different

      jobs:
      deploy:
         runs-on: ubuntu-latest

         steps:
         - name: Checkout Repository
            uses: actions/checkout@v3

         - name: Deploy via SSH
            uses: appleboy/ssh-action@v0.1.6
            with:
            host: ${{ secrets.LIGHTSAIL_IP }}
            username: ${{ secrets.LIGHTSAIL_USER }}
            key: ${{ secrets.LIGHTSAIL_PRIVATE_KEY }}
            port: 22
            script: |
               cd /var/www/visitors
               git fetch origin main
               git stash
               git pull origin main
               sudo systemctl reload nginx
    ```
2. Obtain your .pem file for your instance. Copy starting from --- BEGIN RSA PRIVATE KEY --- to --- END RSA PRIVATE KEY ---.
   ```bash
   cat /path/to/your-key.pem
   ```
      > "--- BEGIN.. ---" and "--- END.. ---" *MUST BE INCLUDED*. If not, the deploy will fail with the following error: ssh.ParsePrivateKey: ssh: no key found.
3. Navigate to **Settings > Secrets and variables > Actions** in your repository.
   - LIGHTSAIL_IP: Public IP address
   - LIGHTSAIL_USER: Username (ubuntu, etc.)
   - LIGHTSAIL_PRIVATE_KEY: Copied private key from Step 2.
  
### You're all set!
**Test the workflow**
1. Add all your files to the repository, then commit/push all the changes.
2. From your instance, run the following.
   ```bash
   sudo chown -R ubuntu:ubuntu /var/www/folder_name
   git reset --hard origin/main
   git pull origin main
   ```
3. Check the status 
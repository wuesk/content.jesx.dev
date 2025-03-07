# Make your own website

You have your favorite website, possibly a social media one, where you're posting your content. Be it art, funny cat photos, or memes, you can share them with your online friends and/or followers. But then the website ends up in different owners' hands, the rules get changed, and everything you posted no longer belongs to you. You look for another website, but eventually, the same thing happens.

![img](https://content.jesx.dev/articles/make-your-own-website/bird_butterfly.jpg)

I want to give you a solution: <b>MAKE YOUR OWN WEBSITE</b>. You, and ONLY YOU, decide what's on your website. You can make your own rules. All you need is about $30 a year.

## Build a website
First thing you need is the website itself. For starters, it can even be a single HTML page. This website you're looking at right now is built with Svelte, but in this tutorial we will use simple single html page website.

What should you put on the website? Whatever you want! It most likely should be about YOU and your thoughts. Some people already have their own websites—check these out, you will get inspired.

## Get a domain and VPS
After you're done with your website, you need two things: a domain name and a place to host your website.

For the domain name, I recommend Cloudflare Registrar. They offer domains at wholesale prices (around $10 per year for most .com domains) with no markup. As a bonus, when you register through Cloudflare, your domain automatically gets their security and performance features for free.

You'll also need somewhere to host your website's files. I personally use a VPS (Virtual Private Server) from OVH, which costs about $2 per month. It's very basic but more than enough for a personal website. Think of a VPS as your own little computer in the cloud—you can put whatever you want on it and have complete control.

If you don't like the VPS provider, you can always use something else (seems like DigitalOcean is also good).


## Secure your VPS
Before hosting the website, let's make your VPS more secure. By default, most VPS providers give you a server with basic security settings that need to be improved. These things are most basic ones I did on my server, if you want to be even more secure, there's a lot of materials online how to make your VPS even more safe.


### Keep your server updated
For Ubuntu, it's as simple as:
```bash
sudo apt update && sudo apt upgrade

# If system requires restart after updates
sudo reboot
```

### Change the SSH port
The default SSH port (22) is constantly under attack from bots trying to break in. Changing it to random high number (like 34567), which only you know, reduces the noise in your logs. Remember to keep note of it, because you will use it now instead of 22.

```bash
# Edit SSH config file
sudo nano /etc/ssh/sshd_config

# Find the line with "Port 22" and change it to your chosen port
Port 34567

# Restart SSH service to apply changes
sudo systemctl restart sshd
```

### Set UP SSH Key Auth
Using SSH keys is much more secure than passwords. Here's how you set it up.
```bash
# On your local computer, generate SSH key pair
ssh-keygen -t ed25519 -C "your@email.com"

# Copy your public key to the server
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@your-server

# After confirming you can log in with the key, disable password authentication
# Edit /etc/ssh/sshd_config again and set:
PasswordAuthentication no
```

### Disable root login
Never allow direct root login. Always log in as a regular user and use `sudo` when needed.
```bash
# In /etc/ssh/sshd_config, set:
PermitRootLogin no
```

### Lock down unused ports
By default, your VPS might have more open ports than needed. Let's set up a basic firewall.
```bash
# Install and enable UFW (Uncomplicated Firewall)
sudo apt install ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow your new SSH port
sudo ufw allow 34567/tcp

# Allow HTTP and HTTPS for your website
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Enable the firewall
sudo ufw enable
```

## Web Server
After securing your VPS, it's time to setup Nginx, which will serve your website to visitors.

### Connecting Domain to Your server
Before we set up the web server, we need to point your domain to your VPS's IP address. You need TWO type A entries. In cloudflare case, it should look something like this:
![img](http://localhost:8080/files/articles/make-your-own-website/domain_settings.jpg)

After that, wait for like 5 to up to 30 minutes for DNS propagation. To check if your domain is pointing to the right place, you can use:
```bash
ping yourdomain.com
```

### Install and Configure Nginx
Installation (on Ubuntu)
```bash
# Install Nginx
sudo apt update
sudo apt install nginx

# Start Nginx and enable it to run at boot
sudo systemctl start nginx
sudo systemctl enable nginx
```
Configuration for the website
```bash
# Create a new site configuration
sudo nano /etc/nginx/sites-available/yourdomain.com

# Add this basic configuration:
server {
    listen 80;
    listen [::]:80;
    server_name yourdomain.com www.yourdomain.com;
    root /var/www/yourdomain.com;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}

# Create a symbolic link to enable the site
sudo ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled/
```

### Set Up your website
```bash
# Create website directory
sudo mkdir -p /var/www/yourdomain.com

# Set ownership
sudo chown -R $USER:$USER /var/www/yourdomain.com

# Set proper permissions
sudo chmod -R 755 /var/www/yourdomain.com
```

### Adding SSL with Let's Encrypt
Let's make your website secure (HTTPS and green padlock thing)
```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Get SSL certificate
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

### Upload your website
Next step is to upload your website files to `/var/www/yourdomain.com`. Some VPS providers allow FTP access to the server, so you can put the files this way. If you're on Linux, you can use `rsync`
```bash
# From your local computer
rsync -avz --delete /path/to/your/website/ user@yourdomain.com:/var/www/yourdomain.com/
```
I personally usually just `git clone` the website on the server.

## Testing your setup
Finally, you can probably see your website under `yourdomain.com`. Remember, to keep your server up to date, and make backups every once in a while (especially of your configs like Nginx), so you can easly restore them. You can also learn more about VPS security online, for example on this [website](https://www.bluehost.com/blog/vps-security/).

Have fun!

P.S. If something doesn't work from the guide, please let me know on [X](https://x.com/jesx64) :) 
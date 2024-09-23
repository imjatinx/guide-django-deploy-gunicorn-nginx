# guide-django-deploy-gunicorn-nginx

Deploying a Django application using **Gunicorn** and **Nginx** is a common setup and works very well for production environments. Here’s a step-by-step guide to help you get started:

### Step 1: Install Nginx

If you haven't already installed Nginx, you can do so with:

```bash
sudo apt update
sudo apt install nginx
```

### Step 2: Configure Gunicorn

Assuming you already have Gunicorn set up to serve your Django application, you can start it as follows (for example):

```bash
gunicorn --workers 3 --bind 127.0.0.1:8000 your_project.wsgi:application
```

### Step 3: Create an Nginx Configuration File

Create a new configuration file for your Django application. You can do this in the Nginx `sites-available` directory:

```bash
sudo nano /etc/nginx/sites-available/your_project
```

Add the following configuration, modifying paths and domain names as needed:

```nginx
server {
    listen 80;
    server_name your_domain.com www.your_domain.com;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /path/to/your/staticfiles;
    }

    location /media/ {
        root /path/to/your/mediafiles;
    }

    location / {
        proxy_pass http://127.0.0.1:8000;
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

### Step 4: Enable the Nginx Configuration

Create a symbolic link to the `sites-enabled` directory to enable your configuration:

```bash
sudo ln -s /etc/nginx/sites-available/your_project /etc/nginx/sites-enabled/
```

### Step 5: Test Nginx Configuration

Before restarting Nginx, test the configuration to ensure there are no syntax errors:

```bash
sudo nginx -t
```

### Step 6: Restart Nginx

If the test is successful, restart Nginx to apply the changes:

```bash
sudo systemctl restart nginx
```

### Step 7: Start Gunicorn as a Service (Optional)

To ensure Gunicorn starts automatically, create a systemd service file for it:

```bash
sudo nano /etc/systemd/system/gunicorn.service
```

Add the following configuration:

```ini
[Unit]
Description=gunicorn daemon for Django application
After=network.target

[Service]
User=your_user
Group=www-data
WorkingDirectory=/path/to/your/project
ExecStart=/path/to/your/venv/bin/gunicorn --workers 3 --bind 127.0.0.1:8000 your_project.wsgi:application

[Install]
WantedBy=multi-user.target
```

Then enable and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl start gunicorn
sudo systemctl enable gunicorn
```

### Step 8: Configure Firewall (if applicable)

If you have a firewall enabled, allow traffic on HTTP (port 80) and HTTPS (port 443 if you plan to use SSL):

```bash
sudo ufw allow 'Nginx Full'
```

### Step 9: (Optional) Set Up SSL with Let's Encrypt

If you want to secure your site with HTTPS, you can use Certbot to obtain an SSL certificate:

```bash
sudo apt install certbot python3-certbot-nginx
```

Then run Certbot:

```bash
sudo certbot --nginx
```

Follow the prompts to obtain and configure your SSL certificate.

---

That’s it! You should now have your Django application running with Gunicorn and Nginx. Let me know if you have any questions or need further clarification!

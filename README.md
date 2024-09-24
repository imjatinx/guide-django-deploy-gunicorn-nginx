# guide-django-deploy-gunicorn-nginx

Deploying a Django application on an Nginx server involves several steps. Here is a comprehensive guide to setting up your Django application with Nginx and Gunicorn (as the WSGI server):

### Step 1: Prepare the Server
Ensure that you have a Django project and Nginx installed on your server. If not, install the necessary packages:

- **Install Nginx**:
  
    ```bash
    sudo apt update
    sudo apt install nginx
    ```

- **Install Gunicorn** in your Django virtual environment:

    ```bash
    pip install gunicorn
    ```

### Step 2: Configure Gunicorn for Your Django Application

Go to your Django project directory and test if Gunicorn can serve your application.

```bash
cd /path/to/your/project
gunicorn --workers 3 your_project.wsgi:application
```

- This command runs Gunicorn with 3 workers, serving the Django application defined in the `wsgi.py` file.
- By default, it will serve the application on port `8000`. You can add `--bind 127.0.0.1:8000` if needed.

If this works and you can access your application via IP:PORT (http://127.0.0.1:8000), proceed to the next step.

### Step 3: Create a Systemd Service for Gunicorn

To make Gunicorn run as a background service, create a systemd service file:

```bash
sudo nano /etc/systemd/system/gunicorn.service
```

Add the following content to the file:

```ini
[Unit]
Description=gunicorn daemon for Django application
After=network.target

[Service]
User=your-username
Group=www-data
WorkingDirectory=/path/to/your/project
ExecStart=/path/to/your/venv/bin/gunicorn --workers 3 --bind 127.0.0.1:8000 your_project.wsgi:application

[Install]
WantedBy=multi-user.target
```

- Replace `your-username` with your system's username.
- Replace `/path/to/your/project` with the path to your Django project directory.
- Replace `/path/to/your/venv/` with the path to your Python virtual environment where Gunicorn is installed.

### Step 4: Start and Enable Gunicorn Service

Run the following commands to start the Gunicorn service and enable it to start on boot:

```bash
sudo systemctl start gunicorn
sudo systemctl enable gunicorn
```

Check the status of the Gunicorn service to make sure it’s running:

```bash
sudo systemctl status gunicorn
```

### Step 5: Configure Nginx to Proxy Requests to Gunicorn

Next, you need to configure Nginx to handle client requests and proxy them to Gunicorn. Create an Nginx server block file:

```bash
sudo nano /etc/nginx/sites-available/your_project
```

Add the following content to the file:

```nginx
server {
    listen 80;
    server_name your_domain.com www.your_domain.com;

    location = /favicon.ico { access_log off; log_not_found off; }
    
    location /static/ {
        alias /path/to/your/staticfiles/;  # Replace with the actual static files path
    }

    location /media/ {
        alias /path/to/your/mediafiles/;  # Replace with the actual media files path
    }

    location / {
        proxy_pass http://127.0.0.1:8000; # Point to your Gunicorn server
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

- Replace `your_domain.com` with your domain.
- Update the paths to your `static` and `media` files.
- The `proxy_pass` line directs traffic to the Gunicorn server running on `http://127.0.0.1:8000`.

### Step 6: Enable the Nginx Server Block

Enable the new Nginx configuration by creating a symbolic link:

```bash
sudo ln -s /etc/nginx/sites-available/your_project /etc/nginx/sites-enabled/
```

Remove the default configuration if you no longer need it:

```bash
sudo rm /etc/nginx/sites-enabled/default
```

### Step 7: Test Nginx Configuration

Test the Nginx configuration to ensure there are no syntax errors:

```bash
sudo nginx -t
```

### Step 8: Restart Nginx

Restart Nginx to apply the changes:

```bash
sudo systemctl restart nginx
```

### Step 9: Set Up SSL (Optional)

If you want to secure your site with HTTPS, you can use Let’s Encrypt:

1. Install Certbot:

    ```bash
    sudo apt install certbot python3-certbot-nginx
    ```

2. Obtain SSL certificate and configure Nginx:

    ```bash
    sudo certbot --nginx
    ```

3. Certbot will ask for your domain and automatically configure your Nginx for SSL.

### Step 11: Verify Everything

Visit your domain (or server IP address) in a browser to verify that the application is running correctly through Nginx.

---

That's it! You've successfully deployed your Django application using Gunicorn and Nginx.

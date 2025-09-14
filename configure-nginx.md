# Setup Nginx as a Reverse Proxy for Express API

## 1. Install Nginx

Nginx is available in the default Ubuntu repositories, so we can install it using `apt`.

### Update our package lists:

```bash
sudo apt update
```

### Install Nginx:

```bash
sudo apt install nginx
```

After the installation, Nginx should start automatically. We can verify this by checking its status:

```bash
sudo systemctl status nginx
```

If the status is **"active (running)"**, we're good to go.

---

## 2. Configure Nginx as a Reverse Proxy

The main Nginx configuration files are in the `/etc/nginx` directory.  
We will create a new configuration file for our Express API inside the `sites-available` directory and then enable it.

### Create a new Nginx configuration file:

```bash
sudo nano /etc/nginx/sites-available/user-api.conf
```

### Add the following configuration to the file:

```nginx
server {
    listen 80;
    server_name our_vps_ip_address; # Or our domain name

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

Save the file and exit the editor.

### Enable the configuration by creating a symbolic link:

```bash
sudo ln -s /etc/nginx/sites-available/user-api.conf /etc/nginx/sites-enabled/
```

### Test Nginx configuration for syntax errors:

```bash
sudo nginx -t
```

If the test is successful, we'll see a message confirming the syntax is OK.

### Reload Nginx to apply the new configuration:

```bash
sudo systemctl reload nginx
```

---

## 3. Adjust the Firewall

Finally, we need to allow Nginx to receive external traffic on ports **80 (HTTP)** and **443 (HTTPS)** through our firewall.

### Check the available UFW application profiles for Nginx:

```bash
sudo ufw app list
```

We should see profiles like `'Nginx Full'`, `'Nginx HTTP'`, and `'Nginx HTTPS'`.

### Allow traffic for Nginx HTTP:

```bash
sudo ufw allow 'Nginx HTTP'
```

### Sometimes Nginx HTTP is not available in the list, direct allow port 80:

```bash
sudo ufw allow '80/tcp'
```

### Confirm the new rule is active:

```bash
sudo ufw status
```

We should now be able to access our Express API at:

```
http://our_vps_ip_address
```

without specifying port **3000**.

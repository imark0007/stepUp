
Here is a detailed and **error-free** step-by-step guide to installing **Nginx, MySQL, and PHP 8.2** (with prerequisites) on **Ubuntu 20.04** in a real-time **office environment**. This guide ensures minimal downtime and smooth configuration.  

---

# **Complete Installation Guide: Nginx, MySQL, and PHP 8.2 on Ubuntu 20.04**

## **Prerequisites**
- A **fresh Ubuntu 20.04** server with **sudo/root access**.
- Internet connectivity.
- A static IP or domain name if required.
- An understanding of firewalls and access control lists (ACLs).

---

## **Step 1: Update and Upgrade System Packages**
Before installing any software, update and upgrade your system:
```bash
sudo apt update && sudo apt upgrade -y
```
This ensures that all packages and dependencies are up to date.

---

## **Step 2: Install Nginx Web Server**
### **1. Install Nginx**
```bash
sudo apt install nginx -y
```
### **2. Enable and Start Nginx**
```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```
### **3. Verify Nginx Installation**
Check if Nginx is running:
```bash
sudo systemctl status nginx
```
If installed correctly, you should see **"active (running)"**.

### **4. Adjust Firewall Settings (If UFW is Enabled)**
Allow HTTP and HTTPS traffic:
```bash
sudo ufw allow 'Nginx Full'
```
Check the status:
```bash
sudo ufw status
```

### **5. Test Nginx**
Open a browser and visit your **server's IP** or **domain name**:
```
http://your-server-ip
```
You should see the **"Welcome to Nginx"** page.

---

## **Step 3: Install MySQL Server**
### **1. Install MySQL**
```bash
sudo apt install mysql-server -y
```
### **2. Secure MySQL Installation**
Run the security script:
```bash
sudo mysql_secure_installation
```
- It will ask security-related questions:
  - **Set root password?** Yes
  - **Remove anonymous users?** Yes
  - **Disallow root login remotely?** Yes (For security)
  - **Remove test database?** Yes
  - **Reload privilege tables?** Yes

### **3. Check MySQL Service**
```bash
sudo systemctl status mysql
```
If MySQL is not running, start it:
```bash
sudo systemctl start mysql
```

### **4. Enable MySQL to Start on Boot**
```bash
sudo systemctl enable mysql
```

### **5. Log into MySQL**
```bash
sudo mysql -u root -p
```
Enter the password you set earlier.

### **6. Create a New Database and User (Best Practice)**
Instead of using the **root** account, create a dedicated database user:
```sql
CREATE DATABASE mydatabase;
CREATE USER 'myuser'@'localhost' IDENTIFIED BY 'mypassword';
GRANT ALL PRIVILEGES ON mydatabase.* TO 'myuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
Replace `mydatabase`, `myuser`, and `mypassword` with your values.

---

## **Step 4: Install PHP 8.2 and Required Extensions**
### **1. Add PHP 8.2 Repository**
Ubuntu 20.04 does not come with PHP 8.2, so we need to add the repository:
```bash
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update
```

### **2. Install PHP 8.2 and Required Modules**
```bash
sudo apt install php8.2 php8.2-fpm php8.2-mysql php8.2-cli php8.2-curl php8.2-xml php8.2-mbstring php8.2-zip unzip -y
```

### **3. Verify PHP Installation**
```bash
php -v
```
You should see output similar to:
```
PHP 8.2.x (cli) (built: ...)
```

### **4. Enable and Start PHP-FPM**
```bash
sudo systemctl enable php8.2-fpm
sudo systemctl start php8.2-fpm
```

### **5. Check PHP-FPM Service**
```bash
sudo systemctl status php8.2-fpm
```

---

## **Step 5: Configure Nginx to Use PHP**
### **1. Create a New Nginx Server Block**
Create a new configuration file:
```bash
sudo nano /etc/nginx/sites-available/mywebsite
```
Paste the following configuration (modify it as needed):
```nginx
server {
    listen 80;
    server_name yourdomain.com;
    root /var/www/mywebsite;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```
### **2. Create a Web Root Directory**
```bash
sudo mkdir -p /var/www/mywebsite
```
Set the correct permissions:
```bash
sudo chown -R www-data:www-data /var/www/mywebsite
sudo chmod -R 755 /var/www/mywebsite
```

### **3. Enable the Site and Restart Nginx**
```bash
sudo ln -s /etc/nginx/sites-available/mywebsite /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

### **4. Test PHP Processing**
Create a test PHP file:
```bash
sudo nano /var/www/mywebsite/index.php
```
Add the following code:
```php
<?php
phpinfo();
?>
```
Save and close the file.

### **5. Verify PHP via Browser**
Visit:
```
http://your-server-ip/index.php
```
You should see the **PHP Info Page**.

---

## **Step 6: Optimize Performance and Security**
### **1. Secure Nginx and PHP**
- Disable **expose_php**:
  ```bash
  sudo nano /etc/php/8.2/fpm/php.ini
  ```
  Find:
  ```
  expose_php = On
  ```
  Change it to:
  ```
  expose_php = Off
  ```
  Save and exit.

- Restart PHP-FPM:
  ```bash
  sudo systemctl restart php8.2-fpm
  ```

### **2. Remove PHP Info File**
After testing, remove `index.php`:
```bash
sudo rm /var/www/mywebsite/index.php
```

---

## **Step 7: Final Testing**
### **1. Restart All Services**
```bash
sudo systemctl restart nginx
sudo systemctl restart mysql
sudo systemctl restart php8.2-fpm
```

### **2. Check Running Services**
```bash
sudo systemctl status nginx mysql php8.2-fpm
```

### **3. Monitor Logs for Errors**
```bash
sudo tail -f /var/log/nginx/error.log
sudo tail -f /var/log/mysql/error.log
sudo tail -f /var/log/php8.2-fpm.log
```

---

## **Conclusion**
This guide ensures that **Nginx, MySQL, and PHP 8.2** are installed and configured correctly for an **office environment** with **minimal errors**. Following these best practices will ensure a stable, secure, and optimized **LAMP/LEMP stack**.

## THANK YOU🚀

# WordPress Docker Development Environment Setup Guide

## Table of Contents

1. [Docker Installation](#docker-installation)
   - [Windows](#windows)
   - [Mac](#mac)
     - [Docker Desktop for Mac](#docker-desktop-for-mac)
     - [Docker Engine (Native)](#docker-engine-native)
   - [Linux](#linux)
2. [Setting Up the Development Environment](#setting-up-the-development-environment)
3. [File Configurations](#file-configurations)
4. [Starting and Stopping the Environment](#starting-and-stopping-the-environment)
5. [Accessing WordPress](#accessing-wordpress)
6. [Plugin Development](#plugin-development)

## Docker Installation

### Windows

1. **System Requirements**:

   - Windows 10 64-bit: Pro, Enterprise, or Education (Build 16299 or later).
   - Windows 11 64-bit: Home or Pro version 21H2 or higher.
   - Enable Hyper-V and Containers Windows features.

2. **Download Docker Desktop**:

   - Visit the official Docker website: <https://www.docker.com/products/docker-desktop>
   - Click on "Download for Windows"

3. **Install Docker Desktop**:

   - Run the downloaded installer (Docker Desktop Installer.exe)
   - Follow the installation wizard, keeping the default settings
   - When prompted, ensure the "Use WSL 2 instead of Hyper-V" option is selected on Windows 10
   - Click "Ok" to start the installation

4. **Start Docker Desktop**:

   - Once installed, start Docker Desktop from the Windows Start menu
   - Wait for Docker to start (the whale icon in the taskbar will stop animating)

5. **Verify Installation**:
   - Open a command prompt or PowerShell
   - Run `docker --version` to check Docker is installed correctly
   - Run `docker run hello-world` to verify Docker can pull and run images

### Mac

On Mac, you have two main options for running Docker: Docker Desktop for Mac and Docker Engine (native). Each has its advantages and use cases.

#### Docker Desktop for Mac

Docker Desktop for Mac is the easiest and most common way to run Docker on macOS. It provides a user-friendly interface and simplifies the Docker experience.

1. **System Requirements**:

   - macOS 11 (Big Sur) or newer for Intel Macs
   - macOS 12 (Monterey) or newer for Apple Silicon Macs
   - At least 4GB of RAM

2. **Download Docker Desktop**:

   - Visit the official Docker website: <https://www.docker.com/products/docker-desktop>
   - Click on "Download for Mac"
   - Choose the appropriate version for your Mac's processor (Intel or Apple Silicon)

3. **Install Docker Desktop**:

   - Open the downloaded .dmg file
   - Drag the Docker icon to the Applications folder
   - Open Docker from the Applications folder

4. **Start Docker Desktop**:

   - Docker Desktop will start automatically
   - You may need to authorize Docker with your system password

5. **Verify Installation**:
   - Open Terminal
   - Run `docker --version` to check Docker is installed correctly
   - Run `docker run hello-world` to verify Docker can pull and run images

**Advantages of Docker Desktop:**

- Easy to install and update
- Includes Docker Compose and Kubernetes
- Provides a graphical user interface for managing containers
- Automatic updates and easy version management
- Integration with macOS native hypervisor framework

**Disadvantages:**

- Uses more system resources compared to native Docker
- May have performance overhead due to virtualization

#### Docker Engine (Native)

Running Docker natively on macOS is possible but requires more setup. This approach is suitable for advanced users who need more control or better performance.

1. **Install Homebrew** (if not already installed):

   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

2. **Install Docker Engine**:

   ```bash
   brew install docker
   ```

3. **Install Docker Machine** (optional, for managing Docker hosts):

   ```bash
   brew install docker-machine
   ```

4. **Create a Docker Machine** (if using Docker Machine):

   ```bash
   docker-machine create --driver virtualbox default
   ```

5. **Set up environment variables**:

   ```bash
   eval $(docker-machine env default)
   ```

6. **Verify installation**:

   ```bash
   docker run hello-world
   ```

**Advantages of Native Docker:**

- Potentially better performance as it runs directly on the host
- More control over Docker configuration
- Lower resource usage

**Disadvantages:**

- More complex setup and management
- Manual updates
- Lacks the user-friendly interface of Docker Desktop
- May require additional tools for full functionality (e.g., Docker Compose needs to be installed separately)

**Choosing Between Docker Desktop and Native Docker:**

- For most developers, especially those new to Docker, Docker Desktop is recommended due to its ease of use and full feature set.
- If you're an advanced user, need maximum performance, or want more control over your Docker environment, running Docker natively might be preferable.
- Consider your specific needs, such as resource usage, performance requirements, and comfort level with command-line tools when making your decision.

### Linux

Docker Desktop is available for some Linux distributions, but the traditional Docker Engine installation is more common. Here's how to install Docker Engine on Ubuntu:

1. **Update the apt package index**:

   ```bash
   sudo apt-get update
   ```

2. **Install packages to allow apt to use a repository over HTTPS**:

   ```bash
   sudo apt-get install \
       apt-transport-https \
       ca-certificates \
       curl \
       gnupg \
       lsb-release
   ```

3. **Add Docker's official GPG key**:

   ```bash
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   ```

4. **Set up the stable repository**:

   ```bash
   echo \
     "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
     $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

5. **Install Docker Engine**:

   ```bash
   sudo apt-get update
   sudo apt-get install docker-ce docker-ce-cli containerd.io
   ```

6. **Verify installation**:

   ```bash
   sudo docker run hello-world
   ```

7. **(Optional) Add your user to the docker group to run Docker without sudo**:

   ```bash
   sudo usermod -aG docker $USER
   ```

   Log out and back in for this to take effect.

For other Linux distributions, please refer to the official Docker documentation.

## Setting Up the Development Environment

1. Create a project directory and necessary files:

   ```bash
   mkdir wordpress-plugin-dev
   cd wordpress-plugin-dev
   mkdir www
   touch docker-compose.yml
   touch nginx.conf
   ```

2. Download WordPress and extract it:

   ```bash
   cd www
   wget https://wordpress.org/latest.tar.gz
   tar -xzvf latest.tar.gz
   mv wordpress/* .
   rm -rf wordpress latest.tar.gz
   ```

## File Configurations

### docker-compose.yml

```yaml
version: "3"

services:
  nginx:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./www:/var/www/html
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - php

  php:
    image: php:8.1-fpm
    volumes:
      - ./www:/var/www/html

  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

### nginx.conf

```nginx
server {
    listen 80;
    server_name localhost;
    root /var/www/html;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```

### wp-config.php

Create this file in the `www` directory:

```php
<?php
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wordpress' );
define( 'DB_PASSWORD', 'wordpress' );
define( 'DB_HOST', 'mysql' );
define( 'DB_CHARSET', 'utf8' );
define( 'DB_COLLATE', '' );

define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );

$table_prefix = 'wp_';

define( 'WP_DEBUG', true );

if ( ! defined( 'ABSPATH' ) ) {
    define( 'ABSPATH', __DIR__ . '/' );
}

require_once ABSPATH . 'wp-settings.php';
```

## Starting and Stopping the Environment

To start the environment:

```bash
docker-compose up -d
```

To stop the environment:

```bash
docker-compose down
```

## Accessing WordPress

Access WordPress in your browser at `http://localhost`. Complete the WordPress installation process.

## Plugin Development

To develop your plugin:

1. Create a new directory in `www/wp-content/plugins/` with your plugin name.
2. Start developing your plugin files in this directory.
3. Changes will be immediately reflected in the running WordPress instance.

This setup provides you with:

- Nginx web server running on port 80
- PHP 8.1 with FPM
- MySQL 5.7 database
- WordPress installation
- Local directory (`www`) mounted to the container for easy file access and modification

The MySQL data is persisted in a Docker volume, so your database will persist across container restarts.

# Self-Hosting Setup Powered by Coolify on Hetzner

## Exmplified with Plausible Analytics

> **Security Note**: For security purposes, specific details such as exact file paths, SSH configurations, and command structures have been intentionally obfuscated in this guide. This prevents potential attackers from gaining insights into the server structure while still providing valuable guidance for setup. When following this guide, replace the placeholder values (indicated by [brackets]) with appropriate values for your environment.

This guide documents the complete process of setting up Coolify and deploying Plausible Analytics on a Hetzner server, including all steps, problems encountered, and their solutions.

## Table of Contents

- [Understanding the Components](#understanding-the-components)
- [Initial Server Setup](#initial-server-setup)
- [Installing Coolify](#installing-coolify)
- [Coolify Configuration](#coolify-configuration)
- [SSH Configuration Issues](#ssh-configuration-issues)
- [Adding a Server in Coolify](#adding-a-server-in-coolify)
- [Deploying Plausible Analytics](#deploying-plausible-analytics)
- [Troubleshooting](#troubleshooting)
- [Security Best Practices](#security-best-practices)
- [Resource Requirements](#resource-requirements)

## Understanding the Components

### Plausible Analytics

Plausible Analytics is a lightweight, open-source web analytics tool that doesn't use cookies and is fully compliant with GDPR, CCPA, and PECR. The Community Edition (CE) is the self-hosted version that you can deploy on your own server.

### Coolify

Coolify is a self-hosted Platform as a Service (PaaS) that simplifies deploying and managing applications, databases, and services. It's similar to services like Heroku or Vercel but runs on your own infrastructure.

### Coolify v3 vs v4

- **Coolify v4** is the latest version with improved UI, better performance, and more features.
- **Coolify v3** is the stable version but has fewer features compared to v4.

### Advantages of Using Coolify

1. **Simplified Deployment**: Coolify provides a user-friendly interface for deploying applications.
2. **Automatic SSL**: Handles SSL certificate generation and renewal automatically.
3. **Easy Updates**: Simplifies the process of updating your applications.
4. **Resource Management**: Provides a dashboard to monitor and manage resources.
5. **One-Click Deployments**: Offers pre-configured templates for popular applications (including Plausible).
6. **Database Management**: Integrated tools for managing databases.

### Differences Between Coolify and Ansible

Coolify and Ansible serve different purposes:

**Coolify:**

- A self-hosted Platform as a Service (PaaS)
- Designed specifically for deploying and managing web applications
- Focuses on providing a user-friendly interface for container-based deployments
- Acts as an application deployment and management platform
- Container-based (uses Docker)
- Web interface-driven with visual dashboards
- Opinionated about how applications should be deployed
- Tightly integrated with specific technologies (Docker, Traefik)

**Ansible:**

- A configuration management and IT automation tool
- Designed for automating infrastructure provisioning, configuration management, and application deployment
- Focuses on defining infrastructure as code
- Acts as a general-purpose automation platform
- Agentless architecture (uses SSH)
- YAML-based configuration files (playbooks)
- Highly flexible and can work with virtually any system
- Technology-agnostic (can manage containers, VMs, bare metal, etc.)

## Initial Server Setup

### Server Requirements

- Ubuntu 24.04 (recommended)
- At least 4GB RAM
- SSH access with root or sudo privileges

### Basic Security Setup

```bash
# Update your system
apt update && apt upgrade -y

# Install basic tools
apt install -y curl wget git jq openssl

# Set up firewall (allow SSH, HTTP, HTTPS)
ufw allow 22
ufw allow 80
ufw allow 443
ufw allow 8000  # For Coolify web interface
ufw allow 3000  # For Coolify real-time service
ufw allow 9000  # For Coolify proxy service
ufw enable
```

## Installing Coolify

### Installation Command

```bash
curl -fsSL https://cdn.coollabs.io/coolify/install.sh | bash
```

### Verifying Installation

After running the installation script, you should see output similar to:

```
Welcome to Coolify Installer!
This script will install everything for you. Sit back and relax.
Source code: https://github.com/coollabs/io/coolify/blob/main/scripts/install.sh

---------------------------------------------
| Operating System  | ubuntu 24.04
| Docker            | 27.0
| Coolify           | 4.0.0-beta.397
| Helper            | 1.0.7
| Realtime          | 1.0.6
---------------------------------------------
```

The installation process:

1. Installs required packages (curl, wget, git, jq, openssl)
2. Checks/installs Docker Engine
3. Configures Docker settings
4. Sets up directories at /data/coolify
5. Configures SSH keys for server management
6. Installs and starts Coolify

### Checking Coolify Version

To check which version of Coolify you have installed:

```bash
docker exec -it $(docker ps -q --filter name=coolify-app) cat /app/version
```

Or check the running container's image tag:

```bash
docker ps --format "{{.Image}}" | grep coolify
```

## Coolify Configuration

### Accessing Coolify

Access the Coolify web interface at: `http://YOUR_SERVER_IP:8000`

### Initial Setup Choices

When setting up Coolify, you'll be presented with two options:

1. **Localhost**: Deploy resources on the same server where Coolify itself is running
2. **Remote Server**: Connect Coolify to separate servers where your applications will be deployed

#### Localhost vs Remote Server

**Localhost Option**:

- Simplest setup - no additional servers needed
- No additional costs
- Good for testing or personal projects with low traffic
- Not recommended for production environments
- All resources share the same server resources

**Remote Server Option**:

- Recommended for production environments
- Better resource isolation
- Better security and stability
- Easier to scale by adding more servers
- Requires additional servers (more cost)

#### Using Your Current Server as Both

You can use your current Hetzner server (where Coolify is installed) as both:

- The Coolify management server
- A "Remote Server" for deploying applications

This gives you the management benefits of the "Remote Server" option without requiring additional Hetzner plans.

**Differences between "Remote Server" (Same Machine) vs "Localhost"**:

1. **Resource Management**:

   - Remote Server approach: Applications run in a separate Docker context from Coolify
   - Localhost approach: Applications share the same Docker context as Coolify

2. **Future Migration**:

   - Remote Server approach: Makes future migration easier
   - Localhost approach: Migration requires reconfiguring applications

3. **Management Interface**:
   - Remote Server approach: Clearer separation in the UI
   - Localhost approach: Everything is managed as part of the local system

### Advanced Coolify Knowledge

#### Understanding Docker Integration

Coolify uses Docker behind the scenes:

1. **Container Management**: Coolify creates and manages Docker containers for your applications
2. **Docker Networks**: Uses Docker networks for container communication
3. **Volume Management**: Creates Docker volumes for persistent data
4. **Docker Compose**: Uses Docker Compose files to define multi-container applications

#### Coolify Directory Structure

Important directories in a Coolify installation (note: paths have been generalized for security purposes):

```
/[coolify-data-path]/
├── [app-data]/       # Application data
├── [backup-data]/    # Backup files
├── [db-data]/        # Database data
├── [proxy-config]/   # Proxy configuration
├── [services-data]/  # Service data
├── [source-files]/   # Coolify source files
└── [ssh-config]/     # SSH keys and configuration
```

#### Coolify Logs and Debugging

When troubleshooting issues (commands generalized for security purposes):

1. **Application Logs**: Available in the Coolify dashboard
2. **Coolify Logs**:
   ```bash
   # Command generalized for security
   docker logs [container-id]
   ```
3. **Database Logs**:
   ```bash
   # Command generalized for security
   docker logs [db-container-id]
   ```
4. **Proxy Logs**:
   ```bash
   # Command generalized for security
   docker logs [proxy-container-id]
   ```

#### Coolify Backup and Restore

To backup your Coolify installation (commands generalized for security):

1. **Database Backup**:
   ```bash
   # Command generalized for security
   docker exec -it [container-id] [backup-command] > [backup-file]
   ```
2. **Configuration Backup**:
   ```bash
   # Command generalized for security
   cp -r /[config-path]/.env /[backup-destination]/
   ```

## SSH Configuration Issues

### Understanding SSH Keys and Files

#### SSH Key Pairs

SSH keys work as a pair:

1. **Private Key**:

   - Your secret key that should NEVER be shared
   - Typically stored on your local machine
   - Usually located in `~/.ssh/` directory (e.g., `id_rsa`, `id_ed25519`)
   - Used to prove your identity to remote servers

2. **Public Key**:
   - Can be freely shared
   - Derived from your private key
   - Usually has `.pub` extension (e.g., `id_rsa.pub`, `id_ed25519.pub`)
   - Placed on servers you want to access in `~/.ssh/authorized_keys`

#### The known_hosts File

The `known_hosts` file in your SSH directory is an important security feature:

1. **Purpose**: Stores fingerprints of servers you've connected to previously
2. **Location**: Usually at `~/.ssh/known_hosts` on your local machine
3. **Security Function**: Helps detect man-in-the-middle attacks by warning you if a server's fingerprint changes
4. **First Connection**: When you connect to a server for the first time, SSH will ask you to verify and save the fingerprint

You might have both `known_hosts` and `known_hosts.old` files - the latter is a backup created when the file is updated.

#### Multiple SSH Keys

Many users have multiple SSH keys for different purposes:

1. **Identifying the Right Key**: If you have multiple keys, it can be confusing to know which one to use with Coolify
2. **Key Pairs**: Each private key (e.g., `id_rsa`) has a corresponding public key (e.g., `id_rsa.pub`)
3. **Checking Keys**: List your SSH keys with `ls -la ~/.ssh/` to see all available keys
4. **Matching Keys**: To determine which private key corresponds to a public key on your server:

   ```bash
   # Get fingerprint of server's public key
   ssh-keygen -lf ~/.ssh/authorized_keys

   # Get fingerprint of your local private key
   ssh-keygen -lf ~/.ssh/id_rsa
   ```

5. **Testing Keys**: Try connecting with a specific key:
   ```bash
   ssh -i ~/.ssh/id_rsa user@your-server-ip
   ```

### SSH Authentication Methods

There are two main ways to access a server via SSH:

**Password Authentication**:

- Requires username, IP address, and password
- Less secure than key-based authentication
- Simpler to set up initially

**Key-Based Authentication**:

- More secure than password authentication
- No passwords transmitted over the network
- Requires setting up SSH key pairs

### SSH Configuration for Coolify

#### Problem: Permission Denied Errors

When adding a server to Coolify, you might encounter:

```
Error: root@your-server-ip: Permission denied (publickey,password).
```

This happens because:

1. Root login might be disabled
2. Password authentication might be disabled
3. SSH keys might not be properly set up

#### Solution: Configure SSH for Root Access

1. Edit SSH configuration:

```bash
sudo vim /etc/ssh/sshd_config
```

2. Add or modify these lines:

```
# Custom Coolify Temporary Permission
# Note: The actual configuration has been modified for security purposes
PermitRootLogin [secure-setting]
# Additional SSH security settings omitted for security
```

This allows root login with SSH keys only (more secure).

3. Restart SSH service:

```bash
sudo systemctl restart ssh
```

4. Set up SSH keys for root (commands generalized for security):

```bash
# Commands to set up SSH keys have been generalized for security
sudo mkdir -p /[root-ssh-path]/
sudo cp [source-keys] /[root-ssh-path]/
sudo chown -R [owner]:[group] /[root-ssh-path]/
sudo chmod [permission-level] /[root-ssh-path]/
sudo chmod [permission-level] /[root-ssh-path]/authorized_keys
```

5. Verify the setup (commands generalized for security):

```bash
# Commands to verify SSH setup have been generalized for security
# Check if the directory was created
sudo ls -la /[root-path]/ | grep .ssh

# Check if the authorized_keys file exists and has content
sudo ls -la /[root-ssh-path]/
sudo cat /[root-ssh-path]/authorized_keys

# Test SSH login
ssh [user]@[server-address]
```

#### Problem: First-time SSH Connection Warning

When connecting to a host for the first time, you'll see:

```
The authenticity of host 'localhost (::1)' can't be established.
ED25519 key fingerprint is SHA256:EXAMPLE_FINGERPRINT_VALUE_REPLACE_WITH_YOUR_OWN.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

**Solution**: Type `yes` to continue. This adds the host key to your known_hosts file.

#### Problem: SSH Still Asking for Password

If SSH still asks for a password after setting up keys:

**Solutions**:

1. Check SSH configuration:

```bash
sudo grep -i "PermitRootLogin\|PasswordAuthentication" /etc/ssh/sshd_config
```

2. Fix permissions:

```bash
sudo chmod 700 /root/.ssh
sudo chmod 600 /root/.ssh/authorized_keys
sudo chown -R root:root /root/.ssh
```

3. Restart SSH:

```bash
sudo systemctl restart ssh
```

4. Try specifying the key explicitly:

```bash
ssh -i ~/.ssh/id_rsa root@localhost
```

#### Problem: SSH Service Name Difference

On Ubuntu, the SSH service is named differently:

**Solution**: Use the correct service name:

```bash
sudo systemctl restart ssh
```

Instead of `sshd.service` (which is used on some other distributions).

#### Problem: Permission Denied When Accessing /root Directory

```
cd root/
-bash: cd: root/: Permission denied
```

**Solution**: Use sudo for operations involving the /root directory:

```bash
sudo mkdir -p /root/.ssh
sudo cp ~/.ssh/authorized_keys /root/.ssh/
```

#### Problem: Vim Permission Error When Saving

When editing system files with vim, you might encounter:

```
"sshd_config" E212: Can't open file for writing
```

**Solutions**:

1. Make sure to use sudo when opening the file:

```bash
sudo vim /etc/ssh/sshd_config
```

2. If you're already in vim and encounter this error, you can save with sudo from within vim:

```
:w !sudo tee %
```

Then enter your password when prompted and type `:q!` to quit.

#### Problem: AllowGroups SSH Configuration

Some Ubuntu servers have an `AllowGroups` directive in SSH config that can prevent root login:

**Solution**: Check if this directive exists and add root to it if needed:

```bash
sudo grep "AllowGroups" /etc/ssh/sshd_config
```

If found, edit the file:

```bash
sudo vim /etc/ssh/sshd_config
```

And change:

```
AllowGroups admin
```

To:

```
AllowGroups admin root
```

## Adding a Server in Coolify

### Server Creation Screen

When adding a server in Coolify, you'll need to provide:

- **Name**: A descriptive name for your server
- **Description** (Optional): Additional information about the server
- **IP Address**: The public IP address of your server

### Problem: IP Address Already in Use

```
IP address is already in use by another team.
```

**Solution**: This means the server is already registered in Coolify. You can:

1. Use the existing server configuration in the dashboard
2. Use Localhost instead of Remote Server
3. Check team settings if in a multi-team setup

### Problem: Real-time Service Warning

```
WARNING: Cannot connect to real-time service
This will cause unusual problems on the UI!
```

**Solution**: This warning indicates that Coolify's WebSocket connections for live UI updates aren't working properly. To fix this:

1. Open the required ports on your server:

```bash
sudo ufw allow 8000/tcp  # Coolify web interface
sudo ufw allow 3000/tcp  # Coolify real-time service
sudo ufw allow 9000/tcp  # Coolify proxy service
sudo ufw status          # Verify the rules
```

2. Make sure Docker network is properly configured:

```bash
docker network ls | grep coolify  # Check if coolify network exists
```

3. Restart Coolify completely (commands generalized for security):

```bash
# Commands generalized for security
cd /[coolify-path]/
docker compose down
docker compose up -d
```

**Important Notes**:

- It's safe to exit the validation page if it appears stuck - the server setup may still be proceeding in the background
- You can refresh the page to see if progress has been made
- Even with this warning, you can still use Coolify, but you'll need to manually refresh the page to see updates
- If validation completes successfully despite this warning, you can proceed with using Coolify

**Checking Validation Status**:
If you're unsure whether validation is still in progress, you can check the logs (command generalized for security):

```bash
# Command generalized for security
docker logs [container-id] | tail -n [log-lines]
```

**Alternative Approach**:
If you continue to have UI update issues, consider:

1. Using the "Localhost" option instead of "Remote Server"
2. Completing the setup and then restarting Coolify afterward
3. Checking if your browser has any extensions blocking WebSocket connections

### Problem: Docker Version Compatibility

During server validation, you might see errors about Docker version compatibility:

```
ERROR: '24.0.9' not found amongst apt-cache madison results
```

**Solution**: This is usually not critical as Coolify will attempt to install a compatible version. You can safely continue if Docker is already installed and working.

### Problem: SSH Key Selection for Coolify

When Coolify asks for an SSH key, it can be confusing which key to use.

**Solution**:

1. For simplicity, let Coolify generate its own key
2. If using your own key, make sure to use the private key from your local machine that corresponds to a public key in the server's authorized_keys file
3. To identify the correct key, compare the public keys:

```bash
# On your local machine
cat ~/.ssh/id_rsa.pub

# On your server
cat ~/.ssh/authorized_keys
```

## Deploying Plausible Analytics

### Using Coolify to Deploy Plausible

1. In the Coolify dashboard, click on "New Resource"
2. Scroll down to "Services" and search for "Plausible Analytics"
3. Click on Plausible Analytics to configure it:
   - Set a name for your deployment
   - Configure the domain (e.g., plausible.yourdomain.com)
   - Set environment variables:
     - `BASE_URL`: https://plausible.yourdomain.com
     - `SECRET_KEY_BASE`: Generate a random string using `openssl rand -base64 48`
   - Leave other settings at their defaults
4. Click "Deploy"

### DNS Configuration

1. Add an A record in your domain's DNS settings:
   ```
   plausible.yourdomain.com -> your-server-ip
   ```
2. Wait for DNS propagation (can take up to 24 hours, but often much faster)

### Accessing Plausible

1. Visit your Plausible instance at https://plausible.yourdomain.com
2. Create your admin account
3. Add your first website:
   - Enter the domain name
   - Configure additional settings as needed

### Adding the Tracking Script

1. In your Plausible dashboard, click on the website you added
2. Click on "Add snippet" to get your tracking code
3. Add this code to the `<head>` section of your website:

```html
<script
  defer
  data-domain="yourdomain.com"
  src="https://plausible.yourdomain.com/js/script.js"
></script>
```

### Plausible Analytics Configuration

#### Environment Variables

Important environment variables for Plausible:

- `BASE_URL`: The URL where Plausible will be accessible (e.g., https://plausible.yourdomain.com)
- `SECRET_KEY_BASE`: A random string for encryption (generate with `openssl rand -base64 48`)
- `PORT`: The port Plausible will listen on (default: 8000)
- `DISABLE_REGISTRATION`: Set to "true" to prevent new user registrations

#### Database Configuration

Plausible uses two databases:

1. **PostgreSQL**: For user accounts and website configurations
2. **ClickHouse**: For storing and querying analytics data

Both are automatically set up by Coolify when deploying Plausible.

#### Scaling Considerations

As your analytics data grows:

1. **CPU Usage**: ClickHouse can be CPU-intensive for sites with high traffic
2. **Memory Requirements**: Consider increasing RAM if you have many sites or high traffic
3. **Disk Space**: Analytics data will grow over time, monitor disk usage
4. **Backup Strategy**: Regular backups are essential, especially for the ClickHouse data

#### Custom Domain Setup

For a professional setup:

1. **DNS Configuration**: Create an A record pointing to your server IP
2. **SSL Certificate**: Coolify handles this automatically with Let's Encrypt
3. **Proxy Headers**: If using an external proxy, ensure proper headers are set

## Troubleshooting

### SSH Connection Issues

- Verify SSH configuration in `/etc/ssh/sshd_config`
- Check permissions on SSH key files
- Ensure the correct SSH service name is used (`ssh` not `sshd` on Ubuntu)
- Try connecting with verbose output: `ssh -v root@your-server-ip`

### Docker Issues

- Check if Docker is running: `systemctl status docker`
- Verify Docker network: `docker network ls | grep coolify`
- Check Docker logs: `docker logs $(docker ps -q --filter name=coolify-app)`

### Coolify UI Issues

- Ensure ports 8000, 3000, and 9000 are open
- Check browser console for errors
- Try clearing browser cache or using incognito mode

### Plausible Deployment Issues

- Check Plausible container logs in Coolify dashboard
- Verify environment variables are set correctly
- Ensure DNS is properly configured

## Security Best Practices

### SSH Security

- Use key-based authentication instead of passwords
- Disable root password login (`PermitRootLogin prohibit-password`)
- Disable password authentication entirely when possible
- Use strong SSH keys (ED25519 or RSA with 4096 bits)

### Firewall Configuration

- Only open necessary ports
- Use UFW for simple firewall management
- Consider implementing fail2ban for brute force protection

### Regular Updates

- Keep the server OS updated
- Update Coolify regularly
- Keep Plausible and other applications updated

## Resource Requirements

### For Plausible Analytics

- At least 2GB of RAM is recommended
- CPU must support SSE 4.2 or NEON instruction set (for ClickHouse)
- Minimal disk space for the application, more for storing analytics data

### For Coolify

- Minimal overhead, but allocate at least 1GB of RAM
- Docker and Docker Compose are required
- Additional resources depending on how many applications you deploy

### Scaling Considerations

- Monitor resource usage as your analytics data grows
- Consider separating Coolify and Plausible to different servers for production use
- Back up your data regularly

## Common Problems and Solutions

### SSH Key Problems

#### Problem: Multiple SSH Keys Confusion

When you have multiple SSH keys and aren't sure which one to use.

**Solution**:

1. List all your keys: `ls -la ~/.ssh/`
2. Check which key is used by default: `ssh -v user@server` (look for "Offering public key")
3. Try connecting with specific keys: `ssh -i ~/.ssh/specific_key user@server`

#### Problem: SSH Agent Not Running

SSH keys not being offered automatically.

**Solution**:

1. Start SSH agent: `eval "$(ssh-agent -s)"`
2. Add your key: `ssh-add ~/.ssh/id_rsa`

### Docker Problems

#### Problem: Docker Permission Denied

```
Got permission denied while trying to connect to the Docker daemon socket
```

**Solution**:

1. Add your user to the docker group: `sudo usermod -aG docker $USER`
2. Log out and back in, or run: `newgrp docker`

#### Problem: Docker Network Issues

Containers can't communicate with each other.

**Solution**:

1. Check if the network exists: `docker network ls`
2. Create the network if missing: `docker network create coolify`
3. Restart Coolify: `cd /data/coolify/source && docker compose down && docker compose up -d`

### Plausible Problems

#### Problem: ClickHouse High CPU Usage

ClickHouse database consuming too much CPU.

**Solution**:

1. Limit the number of websites you track
2. Consider upgrading your server
3. Adjust ClickHouse settings (advanced):
   ```bash
   docker exec -it $(docker ps -q --filter name=clickhouse) clickhouse-client
   ```

#### Problem: Missing Analytics Data

Analytics not showing up for your website.

**Solution**:

1. Verify the tracking script is correctly installed
2. Check for content blockers in your browser
3. Ensure your domain is correctly configured in Plausible
4. Check for JavaScript errors on your website

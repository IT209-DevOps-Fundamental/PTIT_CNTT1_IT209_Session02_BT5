# Web Directory Permission Management Report

## Overview
This document details the configuration process for Linux directory permissions (`chown` and `chmod`) applied to a web project. The objective is to grant the standard `devops` user write access for seamless source code updates without requiring root privileges, while ensuring the Nginx service (running under the `www-data` group) retains the exact read access needed to serve the website without encountering `403 Forbidden` errors.

## Execution Steps

### 1. Ownership Modification
The ownership of the web directory was transferred from `root` to the standard deployment user, and the group ownership was aligned with the web server process.

```bash
# Recursively change the owner to 'devops' and the group to 'www-data'
sudo chown -R devops:www-data /var/www/ptit-web/
```

### 2. Permission Restriction
Granular access rights were applied to differentiate between directories (requiring execution bits for traversal) and standard files. The permissions restrict outside users while satisfying the required read/write parameters.

```bash
# Set directory permissions: Owner(rwx), Group(r-x), Others(---)
sudo find /var/www/ptit-web/ -type d -exec chmod 750 {} \;

# Set file permissions: Owner(rw-), Group(r--), Others(---)
sudo find /var/www/ptit-web/ -type f -exec chmod 640 {} \;
```

## Verification

### Directory Structure and Ownership Check
To validate the applied permissions, a detailed directory listing was executed.

**Execution Log:**
```text
devops@server:~$ ls -la /var/www/ptit-web/
total 16
drwxr-x--- 3 devops www-data 4096 Oct  2 12:00 .
drwxr-xr-x 4 root   root     4096 Oct  2 11:30 ..
drwxr-x--- 2 devops www-data 4096 Oct  2 12:05 html

devops@server:~$ ls -la /var/www/ptit-web/html/
total 12
drwxr-x--- 2 devops www-data 4096 Oct  2 12:05 .
drwxr-x--- 3 devops www-data 4096 Oct  2 12:00 ..
-rw-r----- 1 devops www-data  402 Oct  2 12:10 index.html
```

### Write Privilege Test
A test was conducted under the standard `devops` user context to ensure write capabilities were functional without the use of `sudo`.

**Execution Log:**
```text
devops@server:~$ echo "<!-- Update -->" >> /var/www/ptit-web/html/index.html
devops@server:~$ cat /var/www/ptit-web/html/index.html | grep Update
<!-- Update -->
```
The text append operation succeeded without any `Permission denied` errors.

### Application Test
Accessing the web server via the browser at `http://<DROPLET_IP_ADDRESS>` successfully rendered the updated HTML content. The Nginx daemon served the file correctly without throwing a `403 Forbidden` response, confirming the `www-data` group permissions function precisely as designed.

***
*End of report*

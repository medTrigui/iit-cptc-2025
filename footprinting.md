# Service Footprinting Guide

## Oracle TNS (Port 1521/TCP)

### Basic Information
- **Port**: 1521
- **Protocol**: TCP
- **Service**: Oracle Transparent Network Substrate (TNS)

### Enumeration Steps

1. **Initial Port Scan and Service Detection**
   ```bash
   sudo nmap -p1521 -sV <ip> --open --script=oracle*
   ```

2. **Credential Enumeration using ODAT**
   - Install ODAT:
   ```bash
   sudo apt install odat
   ```
   - Run full enumeration:
   ```bash
   sudo odat all -s <ip>
   ```

3. **Setting up SQLPlus for Database Connection**

   First, download the required Oracle Instant Client packages:
   1. Visit [Oracle Instant Client Downloads](https://www.oracle.com/database/technologies/instant-client/linux-x86-64-downloads.html)
   2. Accept the license agreement
   3. Download both:
      - Basic Package (instantclient-basic-linux.x64-XX.X.0.0.0dbru.zip)
      - SQL*Plus Package (instantclient-sqlplus-linux.x64-XX.X.0.0.0dbru.zip)

   Then run the following installation script:
   ```bash
   #!/bin/bash

   # Exit on any error
   set -e

   echo "[*] Updating package list and installing unzip..."
   sudo apt-get update
   sudo apt-get install -y unzip libaio1

   # Verify the downloaded files exist
   if [ ! -f "instantclient-basic-linux.x64-*.zip" ] || [ ! -f "instantclient-sqlplus-linux.x64-*.zip" ]; then
       echo "[-] Error: Oracle Instant Client zip files not found!"
       echo "    Please download both Basic and SQL*Plus packages from Oracle website first."
       exit 1
   fi

   echo "[*] Creating Oracle directory..."
   sudo mkdir -p /opt/oracle

   echo "[*] Extracting packages..."
   sudo unzip -o "instantclient-basic-linux.x64-*.zip" -d /opt/oracle/
   sudo unzip -o "instantclient-sqlplus-linux.x64-*.zip" -d /opt/oracle/

   # Get the actual instant client directory name
   INSTANT_CLIENT_DIR=$(ls -d /opt/oracle/instantclient_* | head -n 1)

   if [ -z "$INSTANT_CLIENT_DIR" ]; then
       echo "[-] Error: Could not find instant client directory!"
       exit 1
   fi

   echo "[*] Setting up environment variables..."
   # Set up environment variables
   echo "export LD_LIBRARY_PATH=${INSTANT_CLIENT_DIR}:\$LD_LIBRARY_PATH" | sudo tee /etc/profile.d/oracle-instantclient.sh
   echo "export PATH=${INSTANT_CLIENT_DIR}:\$PATH" | sudo tee -a /etc/profile.d/oracle-instantclient.sh

   # Source the new environment variables
   source /etc/profile.d/oracle-instantclient.sh

   echo "[*] Verifying installation..."
   if command -v sqlplus >/dev/null 2>&1; then
       echo "[+] SQLPlus installed successfully!"
       sqlplus -v
   else
       echo "[-] Error: SQLPlus installation failed!"
       exit 1
   fi

   echo "[*] Installation complete! Please log out and log back in for environment variables to take effect."
   ```

### Notes
- After successful credential enumeration using ODAT, use SQLPlus to connect to the database
- Make the script executable: `chmod +x install-sqlplus.sh`
- The script requires root privileges and will prompt for sudo password
- Environment variables are set system-wide in `/etc/profile.d/`
- The script includes error checking and better feedback
- Dependencies like `libaio1` are now included
- You may need to log out and log back in for environment variables to take effect

### Connecting to Oracle Database
After installation, connect to the database using:
```bash
sqlplus username/password@//hostname:1521/service_name
``` 
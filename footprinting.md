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
   ```bash
   #!/bin/bash

   # Exit on any error
   set -e

   echo "[*] Updating package list and installing dependencies..."
   sudo apt-get update
   sudo apt-get install -y unzip libaio1 wget

   echo "[*] Downloading Oracle Instant Client packages..."
   wget https://download.oracle.com/otn_software/linux/instantclient/instantclient-basic-linuxx64.zip
   wget https://download.oracle.com/otn_software/linux/instantclient/instantclient-sqlplus-linuxx64.zip

   # Verify the downloaded files exist and have content
   if [ ! -s instantclient-basic-linuxx64.zip ] || [ ! -s instantclient-sqlplus-linuxx64.zip ]; then
       echo "[-] Error: Failed to download Oracle Instant Client packages!"
       echo "    Please check your internet connection or download manually from Oracle website."
       exit 1
   fi

   echo "[*] Creating Oracle directory..."
   sudo mkdir -p /opt/oracle

   echo "[*] Extracting packages..."
   sudo unzip -o instantclient-basic-linuxx64.zip -d /opt/oracle/
   sudo unzip -o instantclient-sqlplus-linuxx64.zip -d /opt/oracle/

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

   echo "[*] Cleaning up downloaded files..."
   rm -f instantclient-basic-linuxx64.zip instantclient-sqlplus-linuxx64.zip

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
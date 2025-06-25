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

   echo "[*] Creating Oracle directory..."
   sudo mkdir -p /opt/oracle

   echo "[*] Downloading Oracle Instant Client packages..."
   wget https://download.oracle.com/otn_software/linux/instantclient/instantclient-basic-linuxx64.zip
   wget https://download.oracle.com/otn_software/linux/instantclient/instantclient-sqlplus-linuxx64.zip

   # Verify the downloaded files exist and have content
   if [ ! -s instantclient-basic-linuxx64.zip ] || [ ! -s instantclient-sqlplus-linuxx64.zip ]; then
       echo "[-] Error: Failed to download Oracle Instant Client packages!"
       echo "    Please check your internet connection or download manually from Oracle website."
       exit 1
   fi

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
   cat << EOF | sudo tee /etc/profile.d/oracle-instantclient.sh
    export PATH=\$PATH:${INSTANT_CLIENT_DIR}
    export SQLPATH=${INSTANT_CLIENT_DIR}
    export TNS_ADMIN=${INSTANT_CLIENT_DIR}
    export LD_LIBRARY_PATH=${INSTANT_CLIENT_DIR}:\$LD_LIBRARY_PATH
    export ORACLE_HOME=${INSTANT_CLIENT_DIR}
 EOF

   # Make the environment variables available in current session
   export PATH=$PATH:${INSTANT_CLIENT_DIR}
   export SQLPATH=${INSTANT_CLIENT_DIR}
   export TNS_ADMIN=${INSTANT_CLIENT_DIR}
   export LD_LIBRARY_PATH=${INSTANT_CLIENT_DIR}:$LD_LIBRARY_PATH
   export ORACLE_HOME=${INSTANT_CLIENT_DIR}

   echo "[*] Configuring dynamic linker..."
   # Create a configuration file for the dynamic linker
   echo "${INSTANT_CLIENT_DIR}" | sudo tee /etc/ld.so.conf.d/oracle-instantclient.conf

   # Update the dynamic linker cache
   sudo ldconfig

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

   echo "[*] Installation complete!"
   echo "[*] You can now use SQLPlus directly in this terminal."
   echo "[*] For new terminals, either:"
   echo "    1. Log out and log back in, or"
   echo "    2. Run: source /etc/profile.d/oracle-instantclient.sh"
   ```

### Notes
- The script will download and install Oracle Instant Client directly
- Environment variables are set system-wide in `/etc/profile.d/`
- The dynamic linker cache is updated to recognize Oracle libraries
- The script includes error checking and better feedback
- Environment variables are immediately available in the current terminal
- For new terminals, either log out and back in, or source the environment file

### Connecting to Oracle Database
After installation, connect to the database using:
```bash
sqlplus username/password@//hostname:1521/service_name
``` 
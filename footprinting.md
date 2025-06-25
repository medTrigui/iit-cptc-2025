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
   Create and run the following script to set up SQLPlus:
   ```bash
   #!/bin/bash

   # Update and install unzip if not already installed
   sudo apt-get update
   sudo apt-get install unzip -y

   # Download Oracle Instant Client packages
   wget https://download.oracle.com/otn_software/linux/instantclient/214000/instantclient-basic-linux.x64-21.4.0.0.0dbru.zip
   wget https://download.oracle.com/otn_software/linux/instantclient/214000/instantclient-sqlplus-linux.x64-21.4.0.0.0dbru.zip

   # Create target directory and unzip packages
   sudo mkdir -p /opt/oracle
   sudo unzip -d /opt/oracle instantclient-basic-linux.x64-21.4.0.0.0dbru.zip
   sudo unzip -d /opt/oracle instantclient-sqlplus-linux.x64-21.4.0.0.0dbru.zip

   # Set environment variables for this session
   export LD_LIBRARY_PATH=/opt/oracle/instantclient_21_4:$LD_LIBRARY_PATH
   export PATH=/opt/oracle/instantclient_21_4:$PATH

   # Optional: Persist environment variables for future sessions
   echo 'export LD_LIBRARY_PATH=/opt/oracle/instantclient_21_4:$LD_LIBRARY_PATH' >> ~/.bashrc
   echo 'export PATH=/opt/oracle/instantclient_21_4:$PATH' >> ~/.bashrc

   # Print sqlplus version to verify installation
   sqlplus -v
   ```

### Notes
- After successful credential enumeration using ODAT, use SQLPlus to connect to the database
- Make sure to save the SQLPlus setup script with executable permissions (`chmod +x script.sh`)
- The environment variables will be persisted in `.bashrc` for future sessions 
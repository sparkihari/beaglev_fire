# Install and Set Up Libero SoC 2026.1

A quick guide to installing Libero SoC, SoftConsole, and the license server.

---

## 1. Install Libero SoC

Make the installer executable and run it:

```bash
chmod +x Libero_SoC_2026.1_online_lin.sh
sudo ./Libero_SoC_2026.1_online_lin.sh
```

The installation path is:

```text
/usr/local/microchip/Libero_SoC_2026.1
```

Then run the script that installs the required packages:

```bash
cd /usr/local/microchip/Libero_SoC_2026.1
sudo ./req_to_install.sh
```

---

## 2. Install SoftConsole

Make the installer executable and run it:

```bash
chmod +x Microchip-SoftConsole-v2022.2-RISC-V-747-linux-x64-installer.run
sudo ./Microchip-SoftConsole-v2022.2-RISC-V-747-linux-x64-installer.run
```

---

## 3. Set Up the License

Create a license directory under /usr/local/microchip:

```bash
sudo mkdir -p /usr/local/microchip/License
```

Copy the generated license file into it:

```bash
sudo cp /home/<username>/Downloads/License.dat /usr/local/microchip/License/
```

Create a log file for the license server:
```bash
sudo touch /usr/local/microchip/License/license.log
```

Make the license readable by everyone but writable only by root:
```bash
sudo chmod 644 /usr/local/microchip/License/License.dat
sudo chmod 755 /usr/local/microchip/License
```

---

## 4. Check the Hostname and Update the License File

First, check your hostname:

```bash
hostname
```

The hostname in the license file must match the hostname of this machine.

Next, open `/usr/local/microchip/License/License.dat` and make sure the `SERVER` line contains your hostname and MAC ID, and that the daemon paths point to your installation:

```text
SERVER <hostname> <MAC_ID> 1702
DAEMON actlmgrd /usr/local/microchip/Libero_SoC_2026.1/Libero_SoC/Designer/bin64/actlmgrd
DAEMON saltd /usr/local/microchip/Libero_SoC_2026.1/Libero_SoC/Designer/bin64/saltd
VENDOR snpslmd /usr/local/microchip/Libero_SoC_2026.1/Libero_SoC/Designer/bin64/snpslmd
```

---

## 5. Tool Environment Variables Setup

Create a tool setup folder and script for the environment variables under /usr/local/microchip:

```bash
sudo mkdir -p /usr/local/microchip/Setup
sudo nano /usr/local/microchip/Setup/setup-microchip-tools.sh
```

Add the following lines:

```bash
#!/bin/bash

#===============================================================================
#   - SoftConsole (SC_INSTALL_DIR)
#   - Libero (LIBERO_INSTALL_DIR)
#   - Licensing daemon for Libero (LICENSE_DAEMON_DIR)
#===============================================================================
export SC_INSTALL_DIR=/usr/local/microchip/SoftConsole-v2022.2-RISC-V-747
export LIBERO_INSTALL_DIR=/usr/local/microchip/Libero_SoC_2026.1/Libero_SoC
export LICENSE_DAEMON_DIR=/usr/local/microchip/Libero_SoC_2026.1/LicenseDaemons
export LICENSE_FILE_DIR=/usr/local/microchip/License

#
# SoftConsole
#
export PATH=$PATH:$SC_INSTALL_DIR/riscv-unknown-elf-gcc/bin
export FPGENPROG=$LIBERO_INSTALL_DIR/Designer/bin64/fpgenprog

#
# Libero
#
export PATH=$PATH:$LIBERO_INSTALL_DIR/Designer/bin:$LIBERO_INSTALL_DIR/Designer/bin64
export PATH=$PATH:$LIBERO_INSTALL_DIR/Synplify/bin
export PATH=$PATH:$LIBERO_INSTALL_DIR/ModelSim_Pro/linuxacoem
export LOCALE=C
export LD_LIBRARY_PATH=/usr/lib/i386-linux-gnu:$LD_LIBRARY_PATH

#
# Libero License daemon
#
export LM_LICENSE_FILE=1702@<hostname>
export SNPSLMD_LICENSE_FILE=1702@<hostname>

pgrep -x lmgrd >/dev/null || $LICENSE_DAEMON_DIR/lmgrd -c $LICENSE_FILE_DIR/License.dat -l $LICENSE_FILE_DIR/license.log
```

> **Note:** Since this script is sourced from `~/.bashrc`, the `pgrep` check above prevents a new `lmgrd` process from being launched every time you open a terminal.

---

## 6. Configure the Setup tools script in the Environment (`~/.bashrc`)

Open the file:

```bash
nano ~/.bashrc
```

Add the following lines to the end of `~/.bashrc`:

```bash
#
# Libero Tool Setup Script 
#
if [ -f "/usr/local/microchip/Setup/setup-microchip-tools.sh" ]; then
 . "/usr/local/microchip/Setup/setup-microchip-tools.sh"
fi

```

Reload the file so the changes take effect in the current terminal:

```bash
source ~/.bashrc
```

Verify the environment variables:

```bash
echo "$PATH"
```

---

## 7. Start Libero

```bash
libero
```


---

## Recommended Directory Structure

```text
/usr/local/microchip/
├── Libero_SoC_2026.1/
├── SoftConsole-v2022.2-RISC-V-747/
├── License/
│   ├── License.dat
│   └── license.log
└── Setup/
    └── setup-microchip-tools.sh
```

---

## Useful Commands

### Check the license environment

```bash
echo $LM_LICENSE_FILE
echo $SNPSLMD_LICENSE_FILE
```

### Check the SSL environment

```bash
env | grep -iE 'ssl|cert|curl'
```

### Check the CA certificate

```bash
ls -l /etc/ssl/certs/ca-certificates.crt
```

If it is missing or broken, reinstall it:

```bash
sudo apt install --reinstall ca-certificates
sudo update-ca-certificates
```

### Test HTTPS connectivity

```bash
curl -I https://www.microchip.com
```
---

## Quick Troubleshooting

### Certificate or download error

If you see the following error:

```text
error setting certificate verify locations
CAfile: /etc/pki/tls/certs/ca-bundle.crt
```

reinstall the CA certificates:

```bash
sudo apt install --reinstall ca-certificates
sudo update-ca-certificates
```

Then verify the certificate file and environment:

```bash
ls -l /etc/ssl/certs/ca-certificates.crt
env | grep -iE 'ssl|cert|curl'
```
---
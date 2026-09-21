# Install and Set Up Libero SoC 2026.1

A quick guide to installing Libero SoC, SoftConsole, and the FlexLM license server.

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

Create a license directory:

```bash
mkdir -p ~/flexlm
```

Copy the generated license file into it:

```bash
sudo cp \
/usr/local/microchip/Libero_SoC_2026.1/license/License.dat \
~/flexlm/
```

Verify that the file is in place:

```bash
ls -l ~/flexlm/License.dat
```

---

## 4. Check the Hostname and Update the License File

First, check your hostname:

```bash
hostname
```

The hostname in the license file must match the hostname of this machine.

Next, open `~/flexlm/License.dat` and make sure the `SERVER` line contains your hostname and MAC ID, and that the daemon paths point to your installation:

```text
SERVER <hostname> <MAC_ID> 1702
DAEMON actlmgrd /usr/local/microchip/Libero_SoC_2026.1/Libero_SoC/Designer/bin64/actlmgrd
DAEMON saltd /usr/local/microchip/Libero_SoC_2026.1/Libero_SoC/Designer/bin64/saltd
VENDOR snpslmd /usr/local/microchip/Libero_SoC_2026.1/Libero_SoC/Designer/bin64/snpslmd
```

---

## 5. Start FlexLM

Start the license server:

```bash
/usr/local/microchip/Libero_SoC_2026.1/Libero_SoC/Designer/bin64/lmgrd \
-c ~/flexlm/License.dat \
-log /tmp/lmgrd.log &
```

Check that it is running:

```bash
ps -ef | grep -E 'lmgrd|actlmgrd' | grep -v grep
```

Check the log:

```bash
cat /tmp/lmgrd.log
```

To monitor the log live:

```bash
tail -f /tmp/lmgrd.log
```

---

## 6. Configure the Environment (`~/.bashrc`)

Add the following lines to the end of `~/.bashrc`. They point the licensing variables at the local FlexLM server, set the library and locale paths, and add Libero to your `PATH`:

```bash
# Libero SoC licensing
export LM_LICENSE_FILE=1702@<hostname>
export SNPSLMD_LICENSE_FILE=1702@<hostname>

# Libraries and locale
export LD_LIBRARY_PATH=/usr/lib/i386-linux-gnu/:/usr/lib/x86_64-linux-gnu/:/usr/lib/:/usr/local/lib/
export LANG=en_US.UTF-8

# Libero SoC executables
export PATH=/usr/local/microchip/Libero_SoC_2026.1/Libero_SoC/Designer/bin64:${PATH}
```

Reload the file so the changes take effect in the current terminal:

```bash
source ~/.bashrc
```

Verify the license variables:

```bash
echo $LM_LICENSE_FILE
echo $SNPSLMD_LICENSE_FILE
```

---

## 7. Start Libero

```bash
libero
```

If the `libero` command is not found, run it with the full path:

```bash
/usr/local/microchip/Libero_SoC_2026.1/Libero_SoC/Designer/bin64/libero
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

### Check listening ports

```bash
ss -lntp
```

### Check the Libero process

```bash
ps -ef | grep libero
```

### Ensure the /usr/tmp folder exists and is accessible

```bash
sudo mkdir -p /usr/tmp
sudo chmod 1777 /usr/tmp
```

---

## Quick Troubleshooting

### License problem

Run the following commands and review the output:

```bash
hostname
ls -l ~/flexlm/License.dat
ps -ef | grep -E 'lmgrd|actlmgrd'
cat /tmp/lmgrd.log
echo $LM_LICENSE_FILE
```

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

## Recommended Directory Structure

```text
/usr/local/microchip/
└── Libero_SoC_2026.1/

~/flexlm/
└── License.dat
```

Keep **projects and license files outside the Libero installation directory**.

---

## Daily Startup

Once everything is set up, these are usually the only commands you need:

```bash
# Start the license server
/usr/local/microchip/Libero_SoC_2026.1/Libero_SoC/Designer/bin64/lmgrd \
-c ~/flexlm/License.dat \
-log /tmp/lmgrd.log &

# Launch Libero
libero
```

### Useful tip

If Libero reports a license problem, **check `/tmp/lmgrd.log` first**. It usually contains the most useful information about a FlexLM failure.
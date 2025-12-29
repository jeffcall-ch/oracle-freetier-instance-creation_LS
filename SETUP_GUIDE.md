# Oracle A1 Flex Instance Creation - Setup Guide

Complete step-by-step guide for running the instance creation script on your E2.1.Micro VM.

## Prerequisites
- E2.1.Micro Ubuntu VM running in Oracle Cloud
- SSH access to the VM from your laptop
- OCI config file properly set up

---

## Step 1: Clone the Repository

SSH into your VM and clone the repo:

```bash
git clone https://github.com/jeffcall-ch/oracle-freetier-instance-creation_LS
cd oracle-freetier-instance-creation_LS
```

---

## Step 2: Make Setup Script Executable

```bash
chmod +x setup_init.sh
```

---

## Step 3: Generate SSH Key Pair

Generate an SSH key pair on your VM (this will be used to access the new A1 instance):

```bash
ssh-keygen -t rsa -b 4096
```

- Press Enter to save to default location (`/home/ubuntu/.ssh/id_rsa`)
- Press Enter twice to skip passphrase (or set one if preferred)

Verify the key was created:
```bash
cat ~/.ssh/id_rsa.pub
```

---

## Step 4: Configure oci.env

Edit the `oci.env` file with your settings:

```bash
nano oci.env
```

### Required Settings:

```bash
# Path to your OCI config file (should exist on the VM)
OCI_CONFIG=/home/ubuntu/.oci/config

# Availability Domain (AD-1, AD-2, or AD-3)
OCT_FREE_AD=AD-1

# Display name for your new instance
DISPLAY_NAME=my-arm-ubuntu-instance

# Shape (for A1 Flex)
OCI_COMPUTE_SHAPE=VM.Standard.A1.Flex

# Not creating a second E2 Micro
SECOND_MICRO_INSTANCE=False

# Wait time between retry attempts (in seconds)
REQUEST_WAIT_TIME_SECS=60

# Path to your SSH public key (UPDATE THIS!)
SSH_AUTHORIZED_KEYS_FILE=/home/ubuntu/.ssh/id_rsa.pub

# Leave empty when running from E2.1.Micro (auto-detects subnet)
OCI_SUBNET_ID=

# Leave empty to use OS settings below
OCI_IMAGE_ID=

# Operating system settings
OPERATING_SYSTEM="Canonical Ubuntu"
OS_VERSION=22.04

# Assign public IP (set to true for VPN use)
ASSIGN_PUBLIC_IP=true

# Boot volume size in GB (50-200 for free tier)
BOOT_VOLUME_SIZE=50

# Email notifications (optional)
NOTIFY_EMAIL=True
EMAIL=your-email@gmail.com
EMAIL_PASSWORD=your-16-char-app-password

# Discord webhook (optional)
DISCORD_WEBHOOK=
```

### Key Settings Explained:

- **`ASSIGN_PUBLIC_IP=true`**: Needed for VPN/Tailscale use. Free on Oracle.
- **`SSH_AUTHORIZED_KEYS_FILE`**: Must point to `/home/ubuntu/.ssh/id_rsa.pub`
- **`BOOT_VOLUME_SIZE`**: 
  - 50GB = enough for VPN only
  - 100GB = recommended for VPN + WordPress
  - Can be expanded later without recreating instance
- **`OCI_SUBNET_ID`**: Leave empty - auto-detects when running from E2.1.Micro

---

## Step 5: Set Up Gmail App Password (Optional)

To receive email notifications:

1. Go to https://myaccount.google.com/
2. Navigate to **Security** → **2-Step Verification** (must be enabled)
3. Scroll to **App passwords**
4. Generate a new app password for "Mail"
5. Copy the 16-character password (no spaces)
6. Paste into `EMAIL_PASSWORD` in `oci.env`

---

## Step 6: Run the Script in Background

Run the script so it continues after you disconnect:

```bash
nohup ./setup_init.sh > script_output.log 2>&1 &
```

---

## Step 7: Verify Script is Running

Check that both processes are running:

```bash
ps aux | grep -E "setup_init|main.py"
```

You should see:
- `python3 main.py` - creating the instance
- `bash ./setup_init.sh` - monitoring it

---

## Step 8: Monitor Progress

### View live log updates:
```bash
tail -f launch_instance.log
```
Press **Ctrl+C** to exit tail

### View all logs:
```bash
cat launch_instance.log
```

### Check script output:
```bash
tail -f script_output.log
```

---

## Step 9: Disconnect Safely

You can now close your SSH connection. The script will continue running in the background and attempting to create the A1 Flex instance.

You'll receive an email when:
- The instance is successfully created
- An error occurs

---

## Important Notes

### A1 Instance Availability
- A1 Flex instances are in **high demand** on free tier
- The script may run for **hours or days** retrying until capacity is available
- This is normal - just let it run
- You'll get an email when it succeeds

### SSH Key Pairs (Two Different Keys!)

1. **Laptop → E2.1.Micro VM**: Your laptop's key (already working)
2. **E2.1.Micro → A1 Flex**: The new key you generated on the VM

Generating the key on the VM **does not affect** your laptop's access to it.

### Storage

- **50GB**: Enough for VPN/Tailscale only
- **100GB**: Recommended for VPN + WordPress or other services
- Can expand boot volume later without recreating instance
- Free tier limit: 200GB total across all instances

### Public IP

- **2 public IPs** included free in Oracle Free Tier
- No additional cost for assigning public IP
- Recommended for VPN/exit node use case

---

## Useful Commands

### Check if script is running:
```bash
ps aux | grep -E "setup_init|main.py"
```

### Stop the script:
```bash
ps aux | grep main.py
kill <PID>
```

### View all log files:
```bash
ls -la *.log
```

### Check disk space later:
```bash
df -h
```

### Check OCI region (in config file):
```bash
grep region ~/.oci/config
```

---

## Troubleshooting

### No email notifications
- Script only sends email when instance is created or error occurs
- Check `launch_instance.log` for progress
- Verify Gmail App Password is correct (16 characters, no spaces)

### Permission denied on setup_init.sh
```bash
chmod +x setup_init.sh
```

### Script not running after disconnect
- Use `nohup` command as shown in Step 6
- Verify with `ps aux` command

### Check for errors:
```bash
cat ERROR_IN_CONFIG.log
```

---

## After Instance is Created

1. You'll receive an email with instance details
2. SSH into the new A1 instance from your E2.1.Micro VM:
   ```bash
   ssh ubuntu@<A1_PRIVATE_IP>
   ```
3. Set up Tailscale or other services as needed

---

## Region Information

The instance is created in the **same region** as specified in your OCI config file (typically where your E2.1.Micro is running).

To check/change region:
```bash
nano ~/.oci/config
# Look for: region=us-ashburn-1 (or similar)
```

---

## Re-running the Script

If you need to stop and restart:

```bash
# Stop current process
ps aux | grep main.py
kill <PID>

# Clean up and re-run (skips reinstalling packages)
./setup_init.sh rerun
```

Or run in background again:
```bash
nohup ./setup_init.sh rerun > script_output.log 2>&1 &
```

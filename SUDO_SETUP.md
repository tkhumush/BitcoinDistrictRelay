# Setup Passwordless Sudo for bitcoindistrict User

## Method 1: Using visudo (Recommended - Safest)

SSH as root or admin user:
```bash
ssh root@172.105.154.238
```

Open sudoers file safely:
```bash
sudo visudo
```

Add this line at the end of the file:
```
bitcoindistrict ALL=(ALL) NOPASSWD:ALL
```

Save and exit (Ctrl+X, then Y, then Enter in nano)

## Method 2: Create sudoers file (Alternative)

```bash
# SSH as root/admin
ssh root@172.105.154.238

# Create a sudoers file for bitcoindistrict
echo "bitcoindistrict ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/bitcoindistrict

# Set correct permissions
sudo chmod 0440 /etc/sudoers.d/bitcoindistrict
```

## Test It Works

Switch to bitcoindistrict user or SSH as bitcoindistrict:
```bash
# Switch user (if you're root)
su - bitcoindistrict

# Or SSH directly
ssh bitcoindistrict@172.105.154.238
```

Test sudo without password:
```bash
sudo whoami
# Should print "root" without asking for password

sudo apt update
# Should work without password prompt
```

## Verify Docker Access

Also make sure bitcoindistrict is in the docker group:
```bash
# As root or admin user:
sudo usermod -aG docker bitcoindistrict

# Verify
groups bitcoindistrict
# Should show: bitcoindistrict : bitcoindistrict docker
```

Log out and back in for docker group to take effect:
```bash
exit
ssh bitcoindistrict@172.105.154.238
```

Test Docker without sudo:
```bash
docker ps
# Should work without sudo
```

## Security Note

For production, you might want to limit sudo to specific commands instead of ALL:
```bash
# Instead of: bitcoindistrict ALL=(ALL) NOPASSWD:ALL
# Use specific commands:
bitcoindistrict ALL=(ALL) NOPASSWD: /usr/bin/docker, /usr/bin/docker-compose, /usr/bin/apt
```

But for ease of deployment, `NOPASSWD:ALL` is fine for a dedicated deployment user.

## Troubleshooting

If sudo still asks for password:
1. Check the sudoers file syntax: `sudo visudo -c`
2. Make sure there are no conflicting rules earlier in the file
3. The `NOPASSWD:ALL` line should be at the END of the file (order matters)
4. Log out and back in after making changes

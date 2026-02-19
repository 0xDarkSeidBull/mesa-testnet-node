# 🚀 **Mina Mesa Testnet Node - COMPLETE FIXED GUIDE** 🚀

## 📋 **FROM SCRATCH TO SYNC - VERIFIED WORKING COMMANDS**

---

## 📧 **STEP 0: GET KEYS FROM TEAM**
Team will send you email with:
- `mina-mesa-network-bp27` (private key file)
- `mina-mesa-network-bp27.pub` (public key file)
- Usually as a ZIP attachment

---

## 🔐 **STEP 1: TRANSFER KEYS TO SERVER (SFTP)**

```bash
# On your LOCAL terminal (not server)
sftp root@YOUR_SERVER_IP

# Once connected, upload the ZIP file
put mina-mesa-network-bp27.zip /root/

# Exit sftp
exit
```

**OR using SCP:**
```bash
scp mina-mesa-network-bp27.zip root@YOUR_SERVER_IP:/root/
```

---

## 🔄 **STEP 2: SSH INTO SERVER & UPDATE**

```bash
# SSH into your server
ssh root@YOUR_SERVER_IP

# Update system
sudo apt update && sudo apt upgrade -y
```

---

## 📦 **STEP 3: INSTALL DOCKER & UNZIP**

```bash
# Install Docker
sudo apt install -y docker.io unzip
sudo systemctl enable docker
sudo systemctl start docker

# Verify Docker
docker --version
```

---

## 🔑 **STEP 4: SETUP KEYS FOLDER**

```bash
# Create keys directory
mkdir -p ~/keys
cd ~/keys

# Set correct permissions
chmod 700 ~/keys
```

---

## 📁 **STEP 5: EXTRACT AND SETUP KEYS FROM EMAIL**

```bash
# IMPORTANT: ZIP file is in /root, go there first
cd /root

# Extract the ZIP file
unzip mina-mesa-network-bp27.zip

# Check extracted files
ls -la | grep mina-mesa

# Copy files to keys folder with correct names
cp mina-mesa-network-bp27 ~/keys/my-wallet
cp mina-mesa-network-bp27.pub ~/keys/my-wallet.pub

# Go to keys folder
cd ~/keys

# Set correct permissions
chmod 600 my-wallet
chmod 644 my-wallet.pub 2>/dev/null

# Verify files
ls -la ~/keys/
# Should show: my-wallet, my-wallet.pub
```

---

## 🔐 **STEP 6: GENERATE LIBP2P KEY (CRITICAL!)**

```bash
# Generate libp2p key inside temporary container
docker run -it --rm \
  -v ~/keys:/keys \
  --entrypoint /bin/bash \
  gcr.io/o1labs-192920/mina-daemon:4.0.0-preflight1-b649c79-bookworm-mesa \
  -c "export MINA_LIBP2P_PASS='MeraStrongPassword123' && mina libp2p generate-keypair -privkey-path /keys/libp2p-key && chmod 600 /keys/libp2p-key"

# Verify key was generated
ls -la ~/keys/libp2p-key
# Should show: -rw------- 1 root root 483 Feb 19 xx:xx libp2p-key
```

---

## 📂 **STEP 7: CREATE WRITABLE KEYS DIRECTORY**

```bash
# Create fresh writable directory
mkdir -p ~/keys-writable-fresh

# Copy ALL keys to writable directory
cp ~/keys/* ~/keys-writable-fresh/

# Verify my-wallet was copied
ls -la ~/keys-writable-fresh/my-wallet
# If not showing, copy manually:
cp ~/keys/my-wallet ~/keys-writable-fresh/ 2>/dev/null
cp ~/keys/my-wallet.pub ~/keys-writable-fresh/ 2>/dev/null

# Set correct permissions
chmod 700 ~/keys-writable-fresh
chmod 600 ~/keys-writable-fresh/libp2p-key
chmod 600 ~/keys-writable-fresh/my-wallet
chmod 644 ~/keys-writable-fresh/*.pub 2>/dev/null
chmod 644 ~/keys-writable-fresh/*.peerid 2>/dev/null

# Final verification
ls -la ~/keys-writable-fresh/
# MUST SHOW: my-wallet, libp2p-key, my-wallet.pub, libp2p-key.peerid
```

---

## 🚀 **STEP 8: START CONTAINER (FIXED WORKING COMMAND)**

```bash
# Remove any old container
docker stop mina-mesa-preflight 2>/dev/null
docker rm mina-mesa-preflight 2>/dev/null

# Make sure we're in the right directory
cd ~

# FINAL WORKING COMMAND - USE THIS EXACTLY
docker run --name mina-mesa-preflight -d \
  -p 8302:8302 \
  --restart=always \
  -e MINA_LIBP2P_PASS="MeraStrongPassword123" \
  -e MINA_PRIVKEY_PASS="" \
  -v $(pwd)/.mina-config:/root/.mina-config \
  -v ~/keys-writable-fresh:/keys \
  gcr.io/o1labs-192920/mina-daemon:4.0.0-preflight1-b649c79-bookworm-mesa \
  daemon \
  --peer-list-url https://storage.googleapis.com/o1labs-gitops-infrastructure/mina-mesa-network/mina-mesa-network-seeds.txt \
  --libp2p-keypair /keys/libp2p-key \
  --block-producer-key /keys/my-wallet
```

---

## ✅ **STEP 9: VERIFY CONTAINER IS RUNNING**

```bash
# Check if container is running
docker ps
# Expected: CONTAINER ID ... mina-mesa-preflight ... Up ... 0.0.0.0:8302->8302/tcp

# Wait 30 seconds then check logs
sleep 30
docker logs mina-mesa-preflight --tail 20
# Should show "Loaded genesis ledger" and no errors

# If container exited, check full logs:
docker logs mina-mesa-preflight
```

---

## 📊 **STEP 10: MONITOR SYNC STATUS**

```bash
# Basic status check (wait 2-3 minutes after start)
docker exec -it mina-mesa-preflight mina client status | grep -E "Sync status|Block height|Peers"

# Live watch (updates every 10 seconds)
watch -n 10 'docker exec mina-mesa-preflight mina client status | grep -E "Sync status|Block height|Peers"'

# Full status if needed
docker exec -it mina-mesa-preflight mina client status
```

---

## 📝 **STEP 11: VIEW BLOCK SYNC LOGS**

```bash
# See blocks being synced
docker logs -f mina-mesa-preflight | grep -E "Saw block|Updating new available work|state_hash"
```

---

## 💾 **STEP 12: CREATE BACKUP (VERY IMPORTANT!)**

```bash
# Create backup with date stamp
mkdir -p ~/mina-backup-$(date +%Y%m%d)

# Backup everything
cp -r ~/keys ~/mina-backup-$(date +%Y%m%d)/
cp -r ~/.mina-config ~/mina-backup-$(date +%Y%m%d)/
cp -r ~/keys-writable-fresh ~/mina-backup-$(date +%Y%m%d)/

# Verify backup
echo "✅ Backup created in ~/mina-backup-$(date +%Y%m%d)"
ls -la ~/mina-backup-$(date +%Y%m%d)/
```

---

## 🔍 **STEP 13: GET YOUR NODE INFORMATION**

```bash
# Your IP address
curl -s ifconfig.me

# Your Peer ID (wait for node to start)
docker exec mina-mesa-preflight mina client status 2>/dev/null | grep "Libp2p PeerID" || echo "Node starting..."

# Your Block Producer Key
docker exec mina-mesa-preflight mina client status 2>/dev/null | grep "Block producers running" || echo "Node starting..."
```

---

## 🎯 **STEP 14: UNDERSTAND SYNC PROGRESSION**

**Phase 1: Bootstrap (Initial Sync)**
```
Sync status: Bootstrap
Block height: 0 → 10,000
Peers: 3-5
```

**Phase 2: Catchup (Active Sync)**
```
Sync status: Catchup
Block height: 10,000 → 32,918
Peers: 10-20
```

**Phase 3: Synced (Complete!)**
```
Sync status: Synced
Block height: 32,918
Peers: 15-25
```

---

## 🛠️ **STEP 15: TROUBLESHOOTING (FIXED)**

| Problem | Solution |
|---------|----------|
| **`my-wallet not found`** | `cp ~/keys/my-wallet ~/keys-writable-fresh/` |
| **`port 8302 already allocated`** | `docker stop $(docker ps -q --filter publish=8302)` |
| **Container exits immediately** | `docker logs mina-mesa-preflight` (see exact error) |
| **`permissions on /keys`** | `chmod 700 ~/keys-writable-fresh` |
| **`libp2p key corrupted`** | Repeat Step 6 (generate fresh key) |
| **`genesis ledger error`** | `rm -rf ~/.mina-config/genesis && docker restart mina-mesa-preflight` |
| **Daemon not responding** | Wait 2-3 minutes, then check logs |

---

## ✅ **FINAL VERIFICATION CHECKLIST**

```bash
# Run this to verify everything
echo "=== VERIFICATION ==="
echo "1. Docker installed: $(docker --version | head -1)"
echo "2. Keys exist: $(ls -la ~/keys-writable-fresh/my-wallet 2>/dev/null | wc -l) files"
echo "3. Container status: $(docker ps --filter name=mina-mesa-preflight --format 'table {{.Status}}' | tail -1)"
echo "4. Port 8302: $(ss -tlnp | grep 8302 >/dev/null && echo 'OPEN' || echo 'CLOSED')"
echo "==================="
```

---

## 📌 **CRITICAL NOTES (READ CAREFULLY)**

1. **ZIP file location**: It's in `/root/`, NOT in `~/keys/`
2. **Always verify** `my-wallet` exists in `~/keys-writable-fresh/` before starting container
3. **Wait 2-3 minutes** after container start before checking status
4. **If container exits**, always check logs first: `docker logs mina-mesa-preflight`
5. **Password**: Use exactly `MeraStrongPassword123` for libp2p key
6. **Block producer key**: No password (empty string)

---

## 🎉 **CONGRATULATIONS!**

Your Mina Mesa Testnet Node is now successfully running! The node will take 15-30 minutes to fully sync. Once `Sync status: Synced` appears, you're ready to produce blocks!

---

**📝 GitHub repo mein yeh updated version daal do!** 🔥

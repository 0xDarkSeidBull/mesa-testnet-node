# 🚀 **Mina Mesa Testnet Node - Complete Setup Guide** 🚀

## **📋 FROM SCRATCH TO SYNC - THE EXACT COMMANDS THAT WORKED PERFECTLY**

---

## **📧 STEP 0: GET KEYS FROM TEAM**

### **Team will send you email with:**
- `mina-mesa-network-bp27` (private key file)
- `mina-mesa-network-bp27.pub` (public key file)
- Usually as a ZIP attachment

---

## **🔐 STEP 1: TRANSFER KEYS TO SERVER (SFTP)**

### **From your local machine to VPS:**

```bash
# On your LOCAL terminal (not server)
# Use SFTP to transfer files
sftp root@91.107.241.139

# Once connected, upload the ZIP file
put mina-mesa-network-bp27.zip /root/

# Exit sftp
exit
```

### **Or use SCP (simpler):**

```bash
# On your LOCAL terminal
scp mina-mesa-network-bp27.zip root@91.107.241.139:/root/
```

---

## **🔄 STEP 2: SYSTEM UPDATE (ON SERVER)**

```bash
# SSH into your server first
ssh root@91.107.241.139

# Then run these commands
sudo apt update && sudo apt upgrade -y
```

---

## **📦 STEP 3: INSTALL DOCKER**

```bash
# Install Docker
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker

# Verify Docker
docker --version
```

---

## **🔑 STEP 4: SETUP KEYS FOLDER**

```bash
# Create keys directory
mkdir -p ~/keys
cd ~/keys

# Set correct permissions
chmod 700 ~/keys
```

---

## **📁 STEP 5: EXTRACT AND SETUP KEYS FROM EMAIL**

```bash
# Go to root directory where ZIP was uploaded
cd /root

# Extract the ZIP file
unzip mina-mesa-network-bp27.zip

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

## **🔐 STEP 6: GENERATE LIBP2P KEY (CRITICAL!)**

```bash
# Note: Mina is NOT installed on host, so we generate inside container

# Run temporary container to generate libp2p key
docker run -it --rm \
  -v ~/keys:/keys \
  --entrypoint /bin/bash \
  gcr.io/o1labs-192920/mina-daemon:4.0.0-preflight1-b649c79-bookworm-mesa \
  -c "export MINA_LIBP2P_PASS='MeraStrongPassword123' && mina libp2p generate-keypair -privkey-path /keys/libp2p-key && chmod 600 /keys/libp2p-key"

# Verify key was generated
ls -la ~/keys/libp2p-key
# Should show: -rw------- 1 root root 483 Feb 19 10:47 libp2p-key
```

---

## **📂 STEP 7: CREATE WRITABLE KEYS DIRECTORY**

```bash
# Create fresh writable directory (container will use this)
mkdir -p ~/keys-writable-fresh

# Copy all keys to writable directory
cp ~/keys/* ~/keys-writable-fresh/

# Set correct permissions
chmod 700 ~/keys-writable-fresh
chmod 600 ~/keys-writable-fresh/libp2p-key
chmod 600 ~/keys-writable-fresh/my-wallet
chmod 644 ~/keys-writable-fresh/*.pub 2>/dev/null
chmod 644 ~/keys-writable-fresh/*.peerid 2>/dev/null

# Verify
ls -la ~/keys-writable-fresh/
# Should show all key files with correct permissions
```

---

## **🚀 STEP 8: START CONTAINER (THE PERFECT WORKING COMMAND)**

```bash
# Remove any old container if exists
docker stop mina-mesa-preflight 2>/dev/null
docker rm mina-mesa-preflight 2>/dev/null

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

## **✅ STEP 9: VERIFY CONTAINER IS RUNNING**

```bash
# Check if container is running
docker ps
# Expected output: CONTAINER ID ... mina-mesa-preflight ... Up ... 0.0.0.0:8302->8302/tcp

# Check logs (wait 2-3 minutes)
docker logs -f mina-mesa-preflight
# Press Ctrl+C to stop following logs (container keeps running)
```

---

## **📊 STEP 10: MONITOR SYNC STATUS**

```bash
# Basic status check (use this command frequently)
docker exec -it mina-mesa-preflight mina client status | grep -E "Sync status|Block height|Peers"

# Live watch (updates every 10 seconds)
watch -n 10 'docker exec mina-mesa-preflight mina client status | grep -E "Sync status|Block height|Peers"'

# Full status if needed
docker exec -it mina-mesa-preflight mina client status
```

---

## **📝 STEP 11: VIEW BLOCK SYNC LOGS**

```bash
# See blocks being synced in real-time
docker logs -f mina-mesa-preflight | grep -E "Saw block|Updating new available work|state_hash"
```

---

## **💾 STEP 12: CREATE BACKUP (VERY IMPORTANT!)**

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

## **🔍 STEP 13: GET YOUR NODE INFORMATION**

```bash
# Your IP address
curl -s ifconfig.me
# Output: 91.107.241.139

# Your Peer ID
docker exec mina-mesa-preflight mina client status | grep "Libp2p PeerID"
# Output: 12D3KooWMBiwV7xmU1owHy51rtCw5SFYC4LY9E6V7bwckTqaRaMr

# Your Block Producer Key
docker exec mina-mesa-preflight mina client status | grep "Block producers running"
# Output: B62qmgZQkjAFBZxGjDY6A7ZsGHaAGT8foiBah9bSptDjoXDbChryQAe
```

---

## **🎯 STEP 14: UNDERSTAND SYNC PROGRESSION**

### **Phase 1: Bootstrap (Initial Sync)**
```
Sync status: Bootstrap
Block height: 0 → 10,000
Peers: 3-5
```

### **Phase 2: Catchup (Active Sync)**
```
Sync status: Catchup
Block height: 10,000 → 32,918
Peers: 10-20
```

### **Phase 3: Synced (Complete!)**
```
Sync status: Synced
Block height: 32,918
Peers: 15-25
```

---

## **🛠️ STEP 15: USEFUL MANAGEMENT COMMANDS**

```bash
# Quick status summary
echo "=== NODE STATUS ===" && \
echo "IP: $(curl -s ifconfig.me)" && \
echo "PeerID: $(docker exec mina-mesa-preflight mina client status | grep 'Libp2p PeerID' | awk '{print $NF}')" && \
docker exec mina-mesa-preflight mina client status | grep -E "Sync status|Block height|Peers"

# Restart container
docker restart mina-mesa-preflight

# Stop container
docker stop mina-mesa-preflight

# Start container
docker start mina-mesa-preflight

# Remove container (if something goes wrong)
docker rm -f mina-mesa-preflight

# Check logs after restart
docker logs -f mina-mesa-preflight
```

---

## **⚠️ STEP 16: TROUBLESHOOTING COMMON ISSUES**

| Problem | Solution |
|---------|----------|
| `port 8302 already allocated` | `docker stop $(docker ps -q --filter publish=8302)` |
| `permissions on /keys` | `chmod 700 ~/keys-writable-fresh` |
| `libp2p key corrupted/password error` | Repeat Step 6 (generate fresh key) |
| `genesis ledger error` | `rm -rf ~/.mina-config/genesis` and restart container |
| `container exits immediately` | `docker logs mina-mesa-preflight` to see exact error |
| `cannot access /keys/libp2p-key` | Check if file exists: `ls -la ~/keys-writable-fresh/libp2p-key` |

---

## **✅ FINAL CHECKLIST**

- [✅] Docker installed
- [✅] Keys folder created
- [✅] Email keys copied (my-wallet, my-wallet.pub)
- [✅] Libp2p key generated
- [✅] Writable keys directory ready
- [✅] Container running
- [✅] Port 8302 open
- [✅] Peers connected (13+)
- [✅] Sync status: Bootstrap/Catchup/Synced
- [✅] Backup created

---


## **📌 IMPORTANT NOTES**

1. **Keys from Email** - Team sends `mina-mesa-network-bp27` and `.pub` files
2. **Transfer via SFTP/SCP** - Upload to server before starting
3. **Libp2p key** - Must be generated (not from email)
4. **Password** - Use `MeraStrongPassword123` for libp2p key
5. **Block producer key** - No password (empty string)
6. **Container auto-restarts** - Even after system reboot

---

## **🎉 CONGRATULATIONS!**

**Your Mina Mesa Testnet Node is now successfully running!** 

The node will take 15-30 minutes to fully sync. Once `Sync status: Synced` appears, you're ready to produce blocks!

---


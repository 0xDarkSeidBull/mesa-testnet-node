# 🚀 **Mina Mesa Testnet Node** 🚀
## 🌟 **Welcome to the Mesa Testnet!**

<img width="1567" height="601" alt="image" src="https://github.com/user-attachments/assets/450ee6e9-2fc4-4003-b16d-8a9e3fd4c428" />

---

The **Mesa Upgrade** is Mina Protocol's next major hard fork, bringing significant improvements to the network including:
- ✅ **Reduced slot time** (faster blocks)
- ✅ **Expanded zkApp account update limits**
- ✅ **Enhanced protocol performance**
- ✅ **Greater developer flexibility**

### 🎯 **Why Run a Mesa Testnet Node?**

The Mesa Testnet is now **LIVE** and this is your chance to:
- 🔧 **Test the upcoming hard fork** before mainnet
- 🛠️ **Help validate protocol changes** and surface bugs early
- 🌐 **Contribute to Mina's decentralization**
- 🎁 **Position yourself for upcoming opportunities**

---

## ⭐ **IMPORTANT - READ THIS FIRST!**

### 🔥 **What's Coming Next:**

| Phase | Status | Description |
|-------|--------|-------------|
| **Mesa Testnet** | 🟢 **LIVE NOW** | Current phase - Pre-flight testing |
| **Mesa Trail** | 🔜 Coming Soon | Extended testing environment |
| **Trailblazers Program** | 🔜 Coming Soon | **Incentivized testing program** 🏆 |
| **Post-Testnet Validation** | 🔜 Planned | Final validation phase |
| **Devnet Upgrade** | 🔜 Planned | Pre-mainnet upgrade testing |

### 🎁 **Why You Should Run a Node NOW:**

✅ **Discord Role**: Running a node? Join the [#mesa-network-testing](https://discord.gg/minaprotocol) channel and get a special role!  
✅ **Trailblazers Program**: Node operators and developers who participate now will have **higher chances of selection** for the upcoming incentivized program  
✅ **Early Experience**: Be among the first to test Mesa features  
✅ **Community Contribution**: Help shape Mina's future

> 💡 **Pro Tip**: The earlier you join and actively test, the better your chances for upcoming opportunities!

---

## 📌 **Quick Links:**
- 📢 **Discord**: [#mesa-testnet channel](https://discord.gg/minaprotocol)
- 📚 **Official Blog**: [Mesa Testnet Live Announcement](https://minaprotocol.com/blog/mesa-testnet-live-the-next-step-in-minas-upcoming-hard-fork)
- 📋 **Mesa Testing Plan**: [Full Breakdown](https://minaprotocol.com/blog/reaching-mesa-how-were-testing-minas-next-major-upgrade)

---

## 💻 **MINIMUM SYSTEM REQUIREMENTS**

Your node can run successfully on the following minimum specifications:

| Component | Minimum Requirement | Recommended |
|-----------|-------------------|-------------|
| **CPU** | 2 cores | 4+ cores |
| **RAM** | 4 GB | 8+ GB |
| **Storage** | 40 GB | 100+ GB |
| **OS** | Ubuntu 22.04 LTS | Ubuntu 22.04/24.04 LTS |
| **Network** | 1 Mbps | 10+ Mbps |
| **Docker** | Latest version | Latest version |

### ✅ **Works on:**
- Ubuntu 22.04 LTS (Jammy) ✓
- Ubuntu 24.04 LTS (Noble) ✓
- Debian 11/12 ✓
- Any Linux with Docker support ✓

---

## 🎉 **Ready? Let's Get Started!**

Follow the steps below to get your Mesa Testnet node up and running in **15-20 minutes**. Once synced, you'll be actively contributing to Mina's next major upgrade!

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
# Install Docker and unzip
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

# 🟢 FIX: Remove ZIP file after extraction to clean up
rm -f /root/mina-mesa-network-bp27.zip

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

# 🟢 FIX: Force copy ALL keys to writable directory (suppress errors)
cp -f ~/keys/* ~/keys-writable-fresh/ 2>/dev/null

# Double-check my-wallet was copied
if [ ! -f ~/keys-writable-fresh/my-wallet ]; then
    cp ~/keys/my-wallet ~/keys-writable-fresh/
fi

# Set correct permissions
chmod 700 ~/keys-writable-fresh
chmod 600 ~/keys-writable-fresh/libp2p-key
chmod 600 ~/keys-writable-fresh/my-wallet
chmod 644 ~/keys-writable-fresh/*.pub 2>/dev/null
chmod 644 ~/keys-writable-fresh/*.peerid 2>/dev/null

# Final verification
echo "=== Keys in writable directory ==="
ls -la ~/keys-writable-fresh/
# MUST SHOW: my-wallet, libp2p-key, my-wallet.pub, libp2p-key.peerid
```

---

## 🚀 **STEP 8: START CONTAINER IN BACKGROUND**

```bash
# IMPORTANT: Make sure you're in /root directory
cd /root

# Remove any old container if exists
docker stop mina-mesa-preflight 2>/dev/null
docker rm mina-mesa-preflight 2>/dev/null

# Verify keys exist before starting
echo "Checking keys before container start..."
ls -la ~/keys-writable-fresh/my-wallet || { echo "❌ ERROR: my-wallet missing!"; exit 1; }
ls -la ~/keys-writable-fresh/libp2p-key || { echo "❌ ERROR: libp2p-key missing!"; exit 1; }

# 🟢 FINAL WORKING COMMAND - RUNS IN BACKGROUND (detached mode)
docker run --name mina-mesa-preflight -d \
  -p 8302:8302 \
  --restart=always \
  -e MINA_LIBP2P_PASS="MeraStrongPassword123" \
  -e MINA_PRIVKEY_PASS="" \
  -v /root/.mina-config:/root/.mina-config \
  -v ~/keys-writable-fresh:/keys \
  gcr.io/o1labs-192920/mina-daemon:4.0.0-preflight1-b649c79-bookworm-mesa \
  daemon \
  --peer-list-url https://storage.googleapis.com/o1labs-gitops-infrastructure/mina-mesa-network/mina-mesa-network-seeds.txt \
  --libp2p-keypair /keys/libp2p-key \
  --block-producer-key /keys/my-wallet

# Check if container started
sleep 5
if docker ps | grep -q mina-mesa-preflight; then
    echo "✅ Container started successfully in BACKGROUND mode!"
    echo "You can close this terminal - node will keep running"
else
    echo "❌ Container failed to start. Checking logs..."
    docker logs mina-mesa-preflight --tail 20
fi
```

---

## ✅ **STEP 9: VERIFY CONTAINER IS RUNNING IN BACKGROUND**

```bash
# Check if container is running
docker ps
# Expected: CONTAINER ID ... mina-mesa-preflight ... Up ... 0.0.0.0:8302->8302/tcp

# Wait 30 seconds then check logs
echo "Waiting 30 seconds for daemon to initialize..."
sleep 30
docker logs mina-mesa-preflight --tail 20
# Should show "Loaded genesis ledger" and no errors

# If container exited, check full logs:
if ! docker ps | grep -q mina-mesa-preflight; then
    echo "❌ Container exited. Full logs:"
    docker logs mina-mesa-preflight
fi
```

---

## 📊 **STEP 10: MONITOR SYNC STATUS**

```bash
# 🟢 FIX: Wait 2 minutes before first status check (important!)
echo "Waiting 2 minutes for node to initialize..."
sleep 120

# Basic status check
docker exec -it mina-mesa-preflight mina client status | grep -E "Sync status|Block height|Peers"

# Live watch (updates every 10 seconds) - runs in foreground
echo "Starting live monitor (Ctrl+C to stop)"
watch -n 10 'docker exec mina-mesa-preflight mina client status | grep -E "Sync status|Block height|Peers"'

# Full status if needed
docker exec -it mina-mesa-preflight mina client status
```

---

## 📝 **STEP 11: VIEW BLOCK SYNC LOGS**

```bash
# See blocks being synced in real-time
docker logs -f mina-mesa-preflight | grep -E "Saw block|Updating new available work|state_hash"
```

---

## 💾 **STEP 12: CREATE BACKUP (VERY IMPORTANT!)**

```bash
# Create backup with date stamp
BACKUP_DIR=~/mina-backup-$(date +%Y%m%d-%H%M%S)
mkdir -p $BACKUP_DIR

# Backup everything
cp -r ~/keys $BACKUP_DIR/
cp -r ~/.mina-config $BACKUP_DIR/
cp -r ~/keys-writable-fresh $BACKUP_DIR/

# Verify backup
echo "✅ Backup created in $BACKUP_DIR"
ls -la $BACKUP_DIR/
```

---

## 🔍 **STEP 13: GET YOUR NODE INFORMATION**

```bash
# Your IP address
echo "Your IP: $(curl -s ifconfig.me)"

# 🟢 NOTE: Run this AFTER Step 9 shows "Daemon ready" in logs
echo "Waiting for node to be ready..."
sleep 30

# Your Peer ID
PEER_ID=$(docker exec mina-mesa-preflight mina client status 2>/dev/null | grep "Libp2p PeerID" | awk '{print $NF}')
if [ -n "$PEER_ID" ]; then
    echo "✅ Your Peer ID: $PEER_ID"
else
    echo "⏳ Node still starting... Check back in 2 minutes"
    echo "Run: docker logs mina-mesa-preflight | grep 'Daemon ready'"
fi

# Your Block Producer Key
BP_KEY=$(docker exec mina-mesa-preflight mina client status 2>/dev/null | grep "Block producers running" | grep -o 'B62[a-zA-Z0-9]\+')
if [ -n "$BP_KEY" ]; then
    echo "✅ Your Block Producer Key: $BP_KEY"
fi
```

---

## 🎯 **STEP 14: UNDERSTAND SYNC PROGRESSION**

**Phase 1: Bootstrap (Initial Sync)**
```
Sync status: Bootstrap
Block height: 0 → 10,000
Peers: 3-5
Time: First 5-10 minutes
```

**Phase 2: Catchup (Active Sync)**
```
Sync status: Catchup
Block height: 10,000 → 32,918
Peers: 10-20
Time: 10-20 minutes
```

**Phase 3: Synced (Complete!)**
```
Sync status: Synced
Block height: 32,918
Peers: 15-25
Time: 20-30 minutes total
```

---

## 🛠️ **STEP 15: TROUBLESHOOTING (FIXED)**

| Problem | Solution |
|---------|----------|
| **`my-wallet not found`** | `cp ~/keys/my-wallet ~/keys-writable-fresh/ && chmod 600 ~/keys-writable-fresh/my-wallet` |
| **`port 8302 already allocated`** | `docker stop $(docker ps -q --filter publish=8302) 2>/dev/null` |
| **Container exits immediately** | `docker logs mina-mesa-preflight --tail 50` to see exact error |
| **`permissions on /keys`** | `chmod 700 ~/keys-writable-fresh && chmod 600 ~/keys-writable-fresh/*` |
| **`libp2p key corrupted`** | `rm -f ~/keys/libp2p-key*` and repeat Step 6 |
| **`genesis ledger error`** | 🟢 FIX: `rm -rf ~/.mina-config/* && docker restart mina-mesa-preflight` |
| **`Cannot open file: /keys/my-wallet`** | `docker exec mina-mesa-preflight ls -la /keys/` to check |
| **Daemon not responding** | `docker logs mina-mesa-preflight --tail 20` and wait 2-3 minutes |
| **Container name already in use** | `docker rm -f mina-mesa-preflight` then repeat Step 8 |

---

## ✅ **FINAL VERIFICATION CHECKLIST**

```bash
# Run this to verify everything
echo "=== NODE VERIFICATION ==="
echo "1. Docker: $(docker --version | head -1)"
echo "2. Keys in writable dir: $(ls -la ~/keys-writable-fresh/my-wallet 2>/dev/null | awk '{print $5" "$9}')"
echo "3. Container status: $(docker ps --filter name=mina-mesa-preflight --format 'table {{.Status}}' | tail -1)"
echo "4. Port 8302: $(netstat -tlnp 2>/dev/null | grep 8302 >/dev/null && echo 'OPEN' || echo 'CLOSED')"
echo "5. Container logs (last 2 lines):"
docker logs mina-mesa-preflight --tail 2 2>/dev/null || echo "No logs yet"
echo "========================"
```

---

## 📌 **CRITICAL NOTES (READ CAREFULLY)**

1. **ZIP file location**: It's in `/root/`, NOT in `~/keys/`
2. **Always verify** `my-wallet` exists in `~/keys-writable-fresh/` before starting container
3. **Wait 2-3 minutes** after container start before checking status (Step 10 does this automatically)
4. **If container exits**, always check logs first: `docker logs mina-mesa-preflight`
5. **Password**: Use exactly `MeraStrongPassword123` for libp2p key
6. **Block producer key**: No password (empty string)
7. **Container runs in BACKGROUND** (`-d` flag): Terminal band kar sakte ho, node chalega
8. **Auto-restart**: Server reboot ke baad bhi container automatically start hoga (`--restart=always`)

---

## 🚀 **QUICK START (IF SOMETHING GOES WRONG)**

```bash
# If node stops working, run these commands:
docker stop mina-mesa-preflight
docker rm mina-mesa-preflight
rm -rf ~/.mina-config/*
# Then repeat STEP 8 only
```

---

## 🎉 **CONGRATULATIONS!**

Your Mina Mesa Testnet Node is now successfully running in **BACKGROUND mode**! The node will take 15-30 minutes to fully sync. Once `Sync status: Synced` appears, you're ready to produce blocks!

For any issues, check the Troubleshooting section above or run:
```bash
docker logs mina-mesa-preflight --tail 50
```

## 🎯 **AFTER YOUR NODE IS SYNCED:**

```bash
# Join Discord and share:
echo "✅ Mesa Testnet Node Running!"
echo "PeerID: $(docker exec mina-mesa-preflight mina client status | grep 'Libp2p PeerID' | awk '{print $NF}')"
echo "IP: $(curl -s ifconfig.me)"
```

Head over to **[Discord #mesa-network-testing](https://discord.gg/minaprotocol)** and:
1. Share your node status
2. Get your **Discord mesa-network-tester role** 🏷️
3. <img width="354" height="287" alt="image" src="https://github.com/user-attachments/assets/a8a48f11-3d42-45c1-ad5c-b46c29317463" />
4. Stay updated on **Trailblazers Program** launch
5. Connect with other node operators


---


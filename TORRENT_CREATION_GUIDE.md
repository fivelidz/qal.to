# How to Create and Submit Model Torrents

This guide explains how to create torrents for quantized AI models and submit them to Qal.to.

## Prerequisites

**You will need:**
- A quantized GGUF model file (downloaded from HuggingFace or self-quantized)
- A torrent client that can create torrents (transmission, qBittorrent, etc.)
- Command-line tools (optional but recommended)

## Step 1: Download/Prepare the Model

### From HuggingFace:

```bash
# Install huggingface-cli
pip install huggingface-hub

# Download a model
huggingface-cli download \
  bartowski/Llama-3.3-70B-Instruct-GGUF \
  Llama-3.3-70B-Instruct-Q4_K_M.gguf \
  --local-dir ./models
```

### Verify the file:

```bash
# Get SHA256 checksum
sha256sum Llama-3.3-70B-Instruct-Q4_K_M.gguf

# Compare with HuggingFace checksum
# (found on the Files tab of the model page)
```

## Step 2: Create the Torrent

### Option A: Using transmission-cli (Recommended)

```bash
# Install transmission-cli
sudo pacman -S transmission-cli  # Arch/CachyOS
# or
sudo apt install transmission-cli  # Debian/Ubuntu

# Create torrent with multiple trackers
transmission-create \
  --tracker udp://tracker.opentrackr.org:1337/announce \
  --tracker udp://open.stealth.si:80/announce \
  --tracker udp://tracker.torrent.eu.org:451/announce \
  --tracker udp://tracker.moeking.me:6969/announce \
  --comment "Llama 3.3 70B Instruct Q4_K_M - Quantized for local inference" \
  --output llama-3.3-70b-instruct-q4.torrent \
  Llama-3.3-70B-Instruct-Q4_K_M.gguf
```

### Option B: Using mktorrent

```bash
# Install mktorrent
sudo pacman -S mktorrent

# Create torrent
mktorrent \
  -a udp://tracker.opentrackr.org:1337/announce \
  -a udp://open.stealth.si:80/announce \
  -c "Llama 3.3 70B Instruct Q4_K_M" \
  -o llama-3.3-70b-instruct-q4.torrent \
  Llama-3.3-70B-Instruct-Q4_K_M.gguf
```

### Option C: Using qBittorrent GUI

1. Open qBittorrent
2. Tools → Torrent Creator
3. Select file: `Llama-3.3-70B-Instruct-Q4_K_M.gguf`
4. Add trackers (one per line):
   ```
   udp://tracker.opentrackr.org:1337/announce
   udp://open.stealth.si:80/announce
   udp://tracker.torrent.eu.org:451/announce
   udp://tracker.moeking.me:6969/announce
   ```
5. Set comment: "Llama 3.3 70B Instruct Q4_K_M - Quantized for local inference"
6. Click "Create and save"

## Step 3: Extract Info Hash and Magnet Link

### Using transmission-show:

```bash
# View torrent info
transmission-show llama-3.3-70b-instruct-q4.torrent

# Extract magnet link
transmission-show llama-3.3-70b-instruct-q4.torrent | grep "Magnet:"

# Extract info hash
transmission-show llama-3.3-70b-instruct-q4.torrent | grep "Hash:"
```

### Using Python:

```python
import bencode
import hashlib

# Read torrent file
with open('llama-3.3-70b-instruct-q4.torrent', 'rb') as f:
    torrent_data = bencode.decode(f.read())

# Calculate info hash
info_hash = hashlib.sha1(bencode.encode(torrent_data[b'info'])).hexdigest()
print(f"Info Hash: {info_hash}")

# Build magnet link
name = torrent_data[b'info'][b'name'].decode('utf-8')
magnet = f"magnet:?xt=urn:btih:{info_hash}&dn={name}"
print(f"Magnet Link: {magnet}")
```

## Step 4: Start Seeding

### Using transmission:

```bash
# Add torrent to transmission
transmission-remote -a llama-3.3-70b-instruct-q4.torrent

# Verify it's seeding
transmission-remote -l
```

### Using qBittorrent:

1. File → Add Torrent File
2. Select your .torrent file
3. Verify the file path matches your model
4. Click "OK" - it will start seeding

**Important:** Keep your torrent client running! You committed to seed for 30+ days.

## Step 5: Submit to Qal.to

### Option A: Web Form (Easiest)

1. Go to https://qal.to/upload.html
2. Fill out the form with:
   - Model name
   - Magnet link
   - SHA256 checksum
   - Source URL (HuggingFace link)
   - Your info
3. Click "Submit for Review"

### Option B: GitHub Issues (Direct)

1. Go to https://github.com/fivelidz/qal.to/issues/new/choose
2. Select "Model Torrent Submission"
3. Fill out the template
4. Submit issue

## Step 6: Wait for Approval

- **New uploaders:** 24-48 hour review
- **Trusted uploaders:** Auto-published
- We verify checksum and test download
- Once approved, your model appears on Qal.to!

---

## Recommended Public Trackers

Add multiple trackers for better peer discovery:

```
udp://tracker.opentrackr.org:1337/announce
udp://open.stealth.si:80/announce
udp://tracker.torrent.eu.org:451/announce
udp://tracker.moeking.me:6969/announce
udp://exodus.desync.com:6969/announce
udp://tracker.openbittorrent.com:6969/announce
udp://tracker.tiny-vps.com:6969/announce
```

## Best Practices

✅ **DO:**
- Use multiple trackers
- Verify checksums match source
- Include model description in comment
- Seed for 30+ days minimum
- Test download before submitting

❌ **DON'T:**
- Upload models you don't have rights to
- Use only one tracker
- Stop seeding immediately
- Submit corrupted files
- Include personal info in torrent

## Troubleshooting

### "No peers found"
- Add more trackers to your torrent
- Enable DHT in your client
- Check firewall isn't blocking ports

### "Checksum mismatch"
- Re-download from source
- Verify file isn't corrupted
- Check you're using correct hash algorithm (SHA256)

### "Torrent won't seed"
- Verify file path is correct
- Check client is running
- Ensure port forwarding is enabled

## Need Help?

- GitHub Issues: https://github.com/fivelidz/qal.to/issues
- Join discussions: Check our GitHub Discussions tab
- Email: team@qalarc.com (for verification issues)

---

**Thank you for contributing to open, decentralized AI!** 🚀

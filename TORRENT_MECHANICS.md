# Torrent Mechanics & Trust System for Qal.to

## How Torrent Sites Work

### 1. **Torrent File Creation**
When someone wants to share a file via torrent:

```bash
# Using standard torrent tools
transmission-create --tracker udp://tracker.example.com:1337/announce model.gguf
# or
mktorrent -a udp://tracker.example.com:1337/announce -o model.torrent model.gguf
```

This creates a `.torrent` file containing:
- **Info hash** - SHA1 hash of the file content (unique identifier)
- **Tracker URLs** - Servers that coordinate peers
- **Piece hashes** - Chunks of file for verification
- **File metadata** - Name, size, creation date
- **Creator info** - Optional uploader details

### 2. **Magnet Link Generation**
A magnet link is a URL that contains the info hash:

```
magnet:?xt=urn:btih:INFOHASH&dn=FILENAME&tr=TRACKER_URL
```

Example:
```
magnet:?xt=urn:btih:c12fe1c06bba254a9dc9f519b335aa7c1367a88a
       &dn=llama-3.3-70b-q4_k_m.gguf
       &tr=udp://tracker.opentrackr.org:1337/announce
       &tr=udp://open.stealth.si:80/announce
```

**Parts:**
- `xt=urn:btih:` - BitTorrent Info Hash
- `dn=` - Display Name (file name)
- `tr=` - Tracker URL (can have multiple)
- `xl=` - Exact Length (file size in bytes)
- `ws=` - Web Seed (HTTP fallback)

### 3. **Tracker Role**
Trackers don't host files - they coordinate peers:
- Track who has the file (seeders)
- Track who's downloading (leechers)
- Introduce peers to each other
- Return peer lists when queried

**Trackerless (DHT):** Modern torrents use Distributed Hash Table - no central tracker needed.

### 4. **Upload Process for Torrent Sites**

**Traditional Flow:**
1. User creates `.torrent` file locally
2. User uploads `.torrent` to site (small file, few KB)
3. Site stores torrent file
4. Site displays metadata in listings
5. User starts seeding (keeps torrent client running with file)
6. Others download via torrent client
7. Site tracks seeders/leechers via tracker or DHT

**Magnet-Only Flow (Modern):**
1. User creates `.torrent` file locally
2. User extracts info hash from torrent
3. User submits info hash + metadata to site (no file upload needed)
4. Site generates magnet link from info hash
5. User starts seeding
6. Others download via magnet link
7. DHT handles peer discovery (no tracker needed)

---

## Trust & Verification System for Qal.to

### Problem: How do users know a model is legitimate?

**Challenges:**
- Malicious files (malware disguised as models)
- Incorrect quantizations (labeled Q4 but actually Q8)
- Corrupted files
- Outdated versions
- Fake uploaders

### Solution: Multi-Layer Trust System

#### **Layer 1: Uploader Verification Tiers**

**🔴 Unverified (Red Badge)**
- New uploaders
- No track record
- Downloads at own risk
- Requires manual review before seeding

**🟡 Community (Yellow Badge)**
- 5+ successful uploads
- Positive community ratings
- No reported issues
- Basic verification

**🟢 Trusted (Green Badge)**
- 20+ successful uploads
- Verified identity (email, GitHub)
- Consistent quality
- Community endorsements

**⭐ Official (Gold Star)**
- Model creators (Meta, Mistral, etc.)
- Qalarc verified accounts
- Direct from HuggingFace mirrors
- Cryptographically signed

#### **Layer 2: Checksum Verification**

Every model includes:
```json
{
  "model": "llama-3.3-70b-q4_k_m.gguf",
  "sha256": "a1b2c3d4e5f6...",
  "md5": "1a2b3c4d...",
  "size_bytes": 42949672960,
  "verified_by": ["huggingface", "qalarc_team"]
}
```

**Verification Process:**
1. User downloads model via torrent
2. User runs checksum: `sha256sum llama-3.3-70b-q4_k_m.gguf`
3. Compares against published checksum on Qal.to
4. ✅ Match = authentic | ❌ Mismatch = corrupted/tampered

**Auto-verification in browser (WebTorrent):**
```javascript
// Calculate SHA256 as download progresses
const hash = await crypto.subtle.digest('SHA-256', fileBuffer);
const hashHex = Array.from(new Uint8Array(hash))
  .map(b => b.toString(16).padStart(2, '0')).join('');

if (hashHex === expectedHash) {
  showSuccess("✅ Model verified authentic");
} else {
  showError("⚠️ Checksum mismatch - file may be corrupted");
}
```

#### **Layer 3: Source Provenance**

Track model origins:
```
🔗 Source Chain:
HuggingFace (meta-llama/Llama-3.3-70B-Instruct)
  ↓ Quantized by: TheBloke
  ↓ Mirrored to: Qal.to
  ↓ Uploaded by: @qalarc_official ⭐
  ↓ Verified: 2025-11-02
```

**Source Types:**
- **Official HuggingFace** - Direct mirror from HF
- **Community Quant** - Trusted quantizers (TheBloke, etc.)
- **Qalarc Verified** - Tested on Qalarc hardware
- **User Upload** - Community contribution (verify before use)

#### **Layer 4: Community Ratings & Reports**

**Rating System:**
- ⭐⭐⭐⭐⭐ Quality (5 stars)
- 👍 Works as described
- 💬 Comments (12)
- 🚩 Reports (0)

**Report Categories:**
- Malware detected
- Wrong quantization
- Corrupted file
- Misleading description
- Copyright violation

**Auto-suspension:** 3+ reports = hidden until review

#### **Layer 5: File Signature (Advanced)**

For official uploads, use GPG signatures:

```bash
# Qalarc signs model file
gpg --sign --detach-sign llama-3.3-70b-q4.gguf
# Creates llama-3.3-70b-q4.gguf.sig

# Users verify signature
gpg --verify llama-3.3-70b-q4.gguf.sig llama-3.3-70b-q4.gguf
# Output: Good signature from "Qalarc Team <team@qalarc.com>"
```

Include `.sig` files in torrent for verification.

---

## Qal.to Upload Workflow

### **For Users Uploading Models:**

1. **Create Torrent Locally**
```bash
# Using transmission-cli or mktorrent
transmission-create \
  --tracker udp://tracker.opentrackr.org:1337/announce \
  --tracker udp://open.stealth.si:80/announce \
  --comment "Llama 3.3 70B Q4_K_M - Optimized for Qalarc" \
  --output llama-3.3-70b-q4.torrent \
  llama-3.3-70b-q4_k_m.gguf
```

2. **Extract Info Hash**
```bash
# From torrent file
transmission-show llama-3.3-70b-q4.torrent | grep "Hash:"
# Output: Hash: c12fe1c06bba254a9dc9f519b335aa7c1367a88a
```

3. **Calculate Checksums**
```bash
sha256sum llama-3.3-70b-q4_k_m.gguf
md5sum llama-3.3-70b-q4_k_m.gguf
```

4. **Submit to Qal.to**
Fill out web form:
- Model name
- Info hash
- File size
- SHA256/MD5 checksums
- Source (HuggingFace URL, etc.)
- Description/README
- Tags (chat, code, vision, etc.)
- Hardware requirements
- Benchmarks (optional)

5. **Start Seeding**
Keep torrent client running with file loaded

6. **Qal.to Processes**
- Validates info hash format
- Checks for duplicates
- Queues for verification (if new uploader)
- Publishes to site once approved

### **Verification Queue (For New Uploaders)**

Before going live, Qal.to team/trusted users:
1. Downloads via torrent
2. Verifies checksum matches
3. Tests model loads in llama.cpp
4. Confirms quantization level
5. Runs basic inference test
6. Approves or rejects

**Trusted uploaders:** Auto-approved, verification optional

---

## Technical Implementation for Qal.to

### **Backend (Optional - Can be Fully Static)**

Option A: **Fully Static (GitHub Pages)**
- Models catalog in `models.json`
- Manual updates via git commits
- Community submits via GitHub issues
- Qalarc team reviews and merges

Option B: **Minimal Backend (Serverless)**
- AWS Lambda + S3 for submissions
- DynamoDB for model catalog
- CloudFront CDN for static assets
- GitHub Actions for deployments

Option C: **Simple Server (Node.js/Python)**
- Form submissions to database
- Admin panel for approvals
- Automated checksum verification
- Tracker integration (optional)

### **Recommended: Static + GitHub Workflow**

**Pros:**
- Zero hosting costs (GitHub Pages)
- Version controlled catalog
- Community can submit PRs
- Fully transparent
- No database to maintain

**Workflow:**
1. User opens GitHub issue: "Add Model: Llama 3.3 70B Q4"
2. Fills template with info hash, checksums, metadata
3. Qalarc team verifies
4. Merges PR that adds entry to `models.json`
5. GitHub Actions rebuilds site
6. Model appears on Qal.to

---

## Trust Indicators in UI

### **Model Listing Badge Examples:**

```
⭐ Official - Meta Llama 3.3 70B Q4
   Verified by: Qalarc Team | HuggingFace Mirror
   🟢 234 seeders | 12,438 downloads | SHA256 ✓

🟢 Trusted - DeepSeek-R1 671B Q4
   By: @TheBloke | Community Verified
   🟢 89 seeders | 3,291 downloads | SHA256 ✓

🟡 Community - Mixtral-8x22B Q5
   By: @new_uploader | Pending Verification
   🟡 12 seeders | 156 downloads | ⚠️ Verify checksum

🔴 Unverified - Custom-Llama-Merge
   By: @unknown_user | Not Verified
   🟡 3 seeders | 23 downloads | ⚠️ Download at own risk
```

### **Model Detail Page Trust Section:**

```
╔════════════════════════════════════════╗
║  🛡️ TRUST & VERIFICATION              ║
╠════════════════════════════════════════╣
║  Uploader: @qalarc_official ⭐         ║
║  Status: VERIFIED OFFICIAL             ║
║  Source: HuggingFace (meta-llama)      ║
║  Uploaded: 2025-11-02                  ║
║                                        ║
║  Checksums:                            ║
║  SHA256: a1b2c3d4e5f6... ✓            ║
║  MD5: 1a2b3c4d... ✓                   ║
║                                        ║
║  Community:                            ║
║  Rating: ⭐⭐⭐⭐⭐ (234 reviews)          ║
║  Reports: 0 🚩                         ║
║                                        ║
║  Verified by:                          ║
║  ✓ Qalarc Team (2025-11-02)           ║
║  ✓ HuggingFace Mirror                 ║
║  ✓ Community Testing (156 users)      ║
╚════════════════════════════════════════╝
```

---

## Next Steps

1. ✅ Understand torrent mechanics
2. ✅ Design trust/verification system
3. ⏳ Build Evangelion-inspired static UI
4. ⏳ Create `models.json` schema
5. ⏳ Implement upload submission form
6. ⏳ Add checksum verification tools
7. ⏳ Integrate trust badges in listings

Ready to start building the actual site with this trust system in place?

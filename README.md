# Qal.to - Decentralized AI Model Marketplace

**Domain:** qal.to
**Purpose:** Torrent-based marketplace for quantized local AI models
**Integration:** Links to qalarc.com for compatible hardware

## Vision

Qal.to is a community-driven torrent marketplace specifically for quantized LLMs and AI agents. Users can discover, download, and share optimized models via P2P distribution, with direct hardware compatibility information linking to Qalarc systems.

## Core Features (Planned)

- **Model Discovery**: Browse quantized models by size, use-case, and quantization level
- **Torrent Distribution**: Magnet links and WebTorrent for decentralized downloads
- **Hardware Pairing**: Direct compatibility indicators for Qalarc 256GB/512GB systems
- **Community Ratings**: User reviews and benchmarks
- **Verification**: Checksums and signatures for model authenticity
- **Performance Metrics**: Real-world benchmarks on Qalarc hardware

## Tech Stack

- Static HTML/CSS/JavaScript (for speed and simplicity)
- WebTorrent for browser-based downloading
- Magnet links for decentralized distribution
- Client-side search/filtering
- GitHub Pages or SiteGround hosting

## Repository Structure

```
qalto_version/
├── README.md          # This file
├── index.html         # Homepage
├── browse.html        # Model browsing interface
├── model.html         # Model detail template
├── about.html         # About/FAQ
├── assets/
│   ├── css/
│   ├── js/
│   └── images/
└── data/
    └── models.json    # Model catalog (generated/curated)
```

## Development Status

- [x] Repository initialized
- [ ] Design mockups
- [ ] Homepage implementation
- [ ] Browse/search functionality
- [ ] Model detail pages
- [ ] Torrent integration
- [ ] Qalarc.com integration
- [ ] Domain configuration

## Integration with Qalarc

Every model page will include:
- Minimum RAM requirements
- Recommended Qalarc system (256GB/512GB)
- Direct purchase link to qalarc.com
- Performance benchmarks on Qalarc hardware

## License

TBD

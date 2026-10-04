# StegnoVault — Image Inscription & Steganography Tool

> Hide secret messages invisibly inside PNG/BMP images using **LSB steganography** and **AES-256-GCM encryption** — 100% client-side, no server, no uploads.

🔗 **Live Demo →** [ramaneon.github.io/stegno](https://ramaneon.github.io/stegno)

---

## Features

| Feature | Details |
|---|---|
| 🧬 **LSB Steganography** | Embeds data into the 1–3 least-significant bits of each RGB pixel channel |
| 🔒 **AES-256-GCM Encryption** | Password-protected messages encrypted before embedding |
| 🔑 **PBKDF2 Key Derivation** | 100 000 iterations, random 16-byte salt + 12-byte IV per encode |
| 🖼️ **PNG & BMP Support** | Lossless formats only — JPEG compression destroys hidden data |
| ⚡ **100% Browser-Side** | No servers, no API calls, nothing ever leaves your device |
| 📊 **Capacity Meter** | Real-time indicator of how much of the image's LSB space is used |
| 🎚️ **Adjustable Bit Depth** | 1-bit (stealth), 2-bit (balanced), 3-bit (high capacity) |
| 🌙 **Dark Glassmorphism UI** | Animated gradient background, smooth micro-interactions |
| 📱 **Responsive** | Works on mobile, tablet, and desktop |

---

## How It Works

```
Message → [AES-256-GCM + PBKDF2] → Ciphertext bytes
       → [Split into N-bit chunks] 
       → [Written to LSB of RGB pixels]
       → [Downloaded as PNG]

Recipient uploads PNG → reads LSB chunks → reassembles bytes
       → [AES-256-GCM decrypt with password] → Original message
```

### Binary Header Format (32 bytes, embedded at pixel 0)

```
Bytes  Content
0–3    Magic: 0x53 0x54 0x47 0x4E  ("STGN")
4–7    Payload length (uint32 LE)
8      Bit depth (bpc: 1, 2, or 3)
9      Flags (reserved)
10–11  Version (0x01 0x00)
12–31  Reserved zeros
```

The encrypted payload (if AES is enabled) has its own 32-byte sub-header:

```
Bytes  Content
0–3    Crypto magic: 0x53 0x56 0x41 0x45  ("SVAE")
4–19   PBKDF2 salt (16 bytes, random)
20–31  AES-GCM IV (12 bytes, random)
32+    AES-GCM ciphertext
```

---

## Usage

### Encode (hide a message)
1. Open [the tool](https://ramaneon.github.io/stegno)
2. Click **Encode** tab
3. Upload a PNG or BMP image
4. Type your secret message
5. Set a password (optional but recommended — enables AES-256-GCM)
6. Click **Encode Message**
7. Download the resulting PNG and send it to your friend

### Decode (extract a message)
1. Open [the tool](https://ramaneon.github.io/stegno)
2. Click **Decode** tab
3. Upload the encoded PNG (must be the original lossless file — never re-compress to JPEG)
4. Enter the password if one was used
5. Select the same bit depth used during encoding (default: 2)
6. Click **Extract Message**

---

## Security Notes

- **AES-256-GCM** provides authenticated encryption — wrong password = failed authentication, not garbled output
- **PBKDF2** with 100 000 iterations makes brute-force attacks expensive
- Random salt and IV per encode — same message + same password = different ciphertext every time
- Without the password, the embedded bytes appear as random noise (indistinguishable from unencoded image noise at low bit depths)
- Use **1-bit depth** for maximum stealth — zsteg and similar tools won't trivially detect the payload

---

## CTF / Forensics Notes

This tool uses a custom binary header (`STGN` magic) at pixel offset 0. If you're doing forensics on a StegnoVault-encoded image:

```bash
# Check LSB with zsteg (will see encrypted noise, not plaintext)
zsteg -a file.png

# binwalk won't find anything — it's pixel-embedded, not appended
binwalk file.png

# exiftool metadata is clean
exiftool -a -u file.png
```

The payload is only recoverable with the correct password and bit depth.

---

## Local Development

No build step — it's a single HTML file:

```bash
git clone https://github.com/ramaneon/stegno
cd stegno
# Open index.html in any browser
start index.html
```

Or serve it locally:
```bash
python -m http.server 8080
# → http://localhost:8080
```

---

## Deploy to GitHub Pages

1. Go to your repo → **Settings → Pages**
2. Source: **Deploy from a branch**
3. Branch: **main**, folder: **/ (root)**
4. Save → your site is live at `https://ramaneon.github.io/stegno`

---

## Tech Stack

- **HTML5 + Vanilla JS** — zero dependencies, zero frameworks
- **Web Crypto API** — AES-256-GCM, PBKDF2 (native browser, FIPS-compliant)
- **Canvas API** — pixel-level read/write for LSB operations
- **CSS3** — glassmorphism, CSS variables, Grid, animations

---

## License

MIT License — free to use, fork, and remix.

---

*Built with ❤️ — zero servers, zero tracking, zero compromise.*

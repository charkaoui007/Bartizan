# Bartizan — Zero-Knowledge Encrypted File Server

<p align="center">
  <img src="https://img.shields.io/badge/security-zero--knowledge-blue?style=for-the-badge">
  <img src="https://img.shields.io/badge/encryption-AES--256--GCM-yellow?style=for-the-badge">
  <img src="https://img.shields.io/badge/deployment-one--command-green?style=for-the-badge">
</p>

> The most secure file hosting system ever built. Your files are unreadable even if the server gets hacked.

## Why Bartizan?

Traditional file hosts (Dropbox, Google Drive, etc.) store your files in plaintext. The server operator can read everything. If hackers breach the server, they get all your files. Third parties can snoop on your data.

**Bartizan changes that forever.**

- 🔒 **Zero-Knowledge**: Server never sees unencrypted data
- 🚀 **Self-Deployed**: Deploy to your own cloud with one command
- 🎲 **No Enumeration**: Files stored with random UUIDs — discoverable only with your API key  
- 🛡️ **Military-Grade**: AES-256-GCM encryption with per-file IV
- 🔑 **Hash-Only Keys**: API key hash stored — even if DB stolen, attackers can't use your credentials

## Security Architecture

```
┌──────────────┐     TLS      ┌──────────────┐     SAS      ┌──────────────┐
│  Flutter App │ ──────────> │  Node.js API │ ─────────> │ Azure Blobs │
│ (Bartizan)   │   (HTTPS)   │ + AES-256    │   Tokens   │ (Encrypted) │
│              │            │             │             │             │
│ PIN/Biometric│            │ Hash API Key│             │ [ENCRYPTED] │
│ + Local     │            │ + Rate Limit│             │   + UUID    │
│   Encryption             │ + Helmet     │             │    Names   │
└──────────────┘            └──────────────┘             └──────────────┘
                                   │
                                   ▼
                          ┌──────────────┐
                          │   MongoDB    │
                          │ (Metadata)  │
                          │             │
                          │ filename:   │
                          │  UUID only  │
                          │ encrypted:  │
                          │  true       │
                          └──────────────┘
```

## What Makes This Unique

| Feature | Dropbox/Google Drive | Bartizan |
|---------|---------------------|----------|
| **Who sees your files** | The provider | Nobody — not even the server |
| **Deployment** | Their servers | Your own cloud, one command |
| **Encryption** | Server-side (they hold the key) | Zero-knowledge (you hold the key) |
| **API Key** | Stored in plaintext | SHA-256 hash only |
| **Filenames** | Public/guessable | Random UUIDs |
| **Enumeration** | Anyone can list files | API key required |




## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | /api/upload | Upload encrypted file |
| GET | /api/file/:name | Get file metadata |
| GET | /api/download/:name | Download & decrypt file |
| DELETE | /api/file/:name | Delete file |
| GET | /api/files | List all files |
| GET | /api/files/recent | Recent files |
| GET | /api/search | Search files |

All endpoints require `X-API-Key` header starting with `rf_live_`.

## Security Details

- **Encryption**: AES-256-GCM with random IV for each file
- **API Key Format**: `rf_live_<24 random hex characters>`
- **API Key Storage**: SHA-256 hash only
- **Rate Limits**: 100 req/15min global, 20 for uploads
- **File Storage**: Azure Blob Storage with SAS tokens
- **Metadata**: MongoDB (separate attack surface)

## Tech Stack

- **Backend**: Node.js, Express, Helmet, express-rate-limit
- **Database**: MongoDB (metadata only)
- **Storage**: Azure Blob Storage (encrypted files)
- **Frontend**: Next.js
- **Mobile**: Flutter
- **Deployment**: Railway (CLI automated)

## Deployment steps

[tutorials.docx.pdf](https://github.com/user-attachments/files/26635096/tutorials.docx.pdf)


---

**Security isn't an afterthought — it's the foundation.**




# 🏰 Bartizan — Zero-Knowledge Encrypted File Server

<p align="center">
  <img src="https://img.shields.io/badge/security-zero--knowledge-blue?style=for-the-badge">
  <img src="https://img.shields.io/badge/encryption-AES--256--GCM-yellow?style=for-the-badge">
  <img src="https://img.shields.io/badge/deployment-one--command-green?style=for-the-badge">
  <img src="https://img.shields.io/badge/backend-Node.js-339933?style=for-the-badge&logo=node.js&logoColor=white">
  <img src="https://img.shields.io/badge/storage-Azure-0078D4?style=for-the-badge&logo=microsoft-azure&logoColor=white">
  <img src="https://img.shields.io/badge/database-MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white">
</p>

> **The most secure self-hosted file server ever built. Your files are unreadable — even if the server gets hacked.**

---

## 📋 Table of Contents

- [Why Bartizan?](#why-bartizan)
- [How It Compares](#how-it-compares)
- [Security Architecture](#security-architecture)
- [Security Details](#security-details)
- [Tech Stack](#tech-stack)
- [API Reference](#api-reference)
- [Deployment](#deployment)
  - [Prerequisites](#prerequisites)
  - [Part 1 — Railway API Token](#part-1--railway-api-token)
  - [Part 2 — MongoDB Atlas API Keys](#part-2--mongodb-atlas-api-keys)
  - [Part 3 — Azure Storage Account](#part-3--azure-storage-account)
  - [Credentials Summary](#credentials-summary)
- [Security Reminders](#security-reminders)

---

## Why Bartizan?

Traditional file hosts (Dropbox, Google Drive, etc.) store your files in plaintext. The server operator can read everything. If hackers breach the server, they get all your files. Third parties can snoop on your data.

**Bartizan changes that.**

- 🔒 **Zero-Knowledge** — the server never sees unencrypted data. Ever.
- 🚀 **Self-Deployed** — deploy to your own cloud infrastructure with one command
- 🎲 **No Enumeration** — files stored with random UUIDs, discoverable only with your API key
- 🛡️ **Military-Grade Encryption** — AES-256-GCM with a unique IV generated per file
- 🔑 **Hash-Only Key Storage** — only a SHA-256 hash of your API key is stored; even a stolen database cannot be used to authenticate

---

## How It Compares

| Feature | Dropbox / Google Drive | **Bartizan** |
|---------|------------------------|--------------|
| **Who can read your files** | The provider + any attacker who breaches them | Nobody — not even the server operator |
| **Deployment** | Their servers, their rules | Your own cloud, fully under your control |
| **Encryption model** | Server-side (they hold the key) | Zero-knowledge (only you hold the key) |
| **API Key storage** | Plaintext or reversible | SHA-256 hash only — irreversible |
| **Filenames on disk** | Original or guessable | Random UUIDs — no information leakage |
| **File enumeration** | Anyone with access can list files | Requires valid API key |

---

## Security Architecture

```
┌──────────────┐     TLS      ┌──────────────────────┐     SAS      ┌───────────────────┐
│  Flutter App │ ──────────▶ │     Node.js API       │ ──────────▶ │   Azure Blob      │
│  (Bartizan)  │   (HTTPS)   │   + AES-256-GCM       │   Tokens    │   Storage         │
│              │             │                       │             │                   │
│ PIN/Biometric│             │  • Hash API Key       │             │  [ENCRYPTED FILE] │
│ + Local      │             │  • Rate Limiting      │             │   + UUID Name     │
│   Encryption │             │  • Helmet headers     │             │                   │
└──────────────┘             └──────────────────────┘             └───────────────────┘
                                          │
                                          ▼
                               ┌──────────────────────┐
                               │       MongoDB         │
                               │     (Metadata)        │
                               │                       │
                               │  filename:  UUID only │
                               │  encrypted: true      │
                               │  owner:     key hash  │
                               └──────────────────────┘
```

**Data flow:**
1. The Flutter client encrypts the file locally with AES-256-GCM before it ever leaves the device
2. The encrypted blob is transmitted over TLS to the Node.js API
3. The API authenticates the request using only the SHA-256 hash of the API key
4. The encrypted blob is stored in Azure with a random UUID filename — no original filename is preserved
5. Metadata (UUID, hash, timestamp) is stored in MongoDB — never the file content or original name
6. Download reverses the process; decryption happens on the client

---

## Security Details

| Property | Implementation |
|----------|----------------|
| **Encryption algorithm** | AES-256-GCM |
| **IV (Initialization Vector)** | Randomly generated per file |
| **API key format** | `rf_live_<24 random hex characters>` |
| **API key storage** | SHA-256 hash only — the raw key is never persisted |
| **Rate limiting** | 100 requests / 15 min globally · 20 requests / 15 min for uploads |
| **File naming** | Random UUID — original filename never stored |
| **Transport security** | TLS (HTTPS) enforced |
| **HTTP hardening** | Helmet.js (security headers) |
| **File storage** | Azure Blob Storage, accessed via short-lived SAS tokens |
| **Metadata storage** | MongoDB (separate attack surface from file storage) |

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| **Backend API** | Node.js, Express, Helmet, express-rate-limit |
| **Database** | MongoDB (metadata only — no file content) |
| **File Storage** | Azure Blob Storage (encrypted blobs) |
| **Frontend** | Next.js |
| **Mobile** | Flutter |
| **Deployment** | Railway (one-command automated CLI deploy) |

---

## API Reference

All endpoints require the `X-API-Key` header with a valid key in the format `rf_live_<24 hex chars>`.

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/upload` | Upload an encrypted file |
| `GET` | `/api/file/:name` | Retrieve file metadata by UUID |
| `GET` | `/api/download/:name` | Download and decrypt a file |
| `DELETE` | `/api/file/:name` | Delete a file |
| `GET` | `/api/files` | List all files for the authenticated key |
| `GET` | `/api/files/recent` | List recently uploaded files |
| `GET` | `/api/search` | Search files by metadata |

---

## Deployment

Bartizan deploys to Railway (backend), MongoDB Atlas (database), and Azure Blob Storage (file storage). Before running the deploy command, you need to collect credentials from all three services. Follow the steps below — no technical experience required.

### Prerequisites

Make sure you have active accounts on all three platforms:

- ✅ [railway.app](https://railway.app) — sign up for free
- ✅ [cloud.mongodb.com](https://cloud.mongodb.com) — sign up for free (at least one Project required)
- ✅ [portal.azure.com](https://portal.azure.com) — sign up (students can use **Azure for Students** for free)

---

### Part 1 — Railway API Token

A Railway API Token allows the deploy script to push your application automatically.

#### Step 1 · Go to Account Settings

In the bottom-left corner of the Railway dashboard, click your **account avatar or username**, then select **Account Settings**.

![Railway — Tokens page](images/img-000.png)

#### Step 2 · Click Tokens

In the left sidebar, click **Tokens**. This page lists all API tokens on your account.

#### Step 3 · Create a New Token

Click **New Token** and fill in the form:

| Field | What to Enter |
|-------|---------------|
| **Name** | e.g. `Bartizan Deploy` or `CI Token` |
| **Workspace** | Choose a workspace, or leave as `No workspace` for account-wide access |

Click **Create**.

#### Step 4 · Copy and Save Immediately

> ⚠️ Railway shows the token **once only**. Copy it immediately and store it somewhere secure (password manager, encrypted file, secrets vault). It cannot be retrieved after you leave this page.

**Save this value:**
```
RAILWAY_API_TOKEN=rf_live_xxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

---

### Part 2 — MongoDB Atlas API Keys

Atlas API Keys allow the deploy script to configure your database project. Each key pair has a **Public Key** (like a username) and a **Private Key** (like a password).

#### Step 1 · Navigate to API Keys

From your **Project Overview**, follow this path:

```
Security → Project Identity & Access → Applications → API Keys
```

![MongoDB Atlas — Project Overview](images/img-001.png)

#### Step 2 · Create a New API Key

Click **Create API Key** and fill in the form:

| Field | What to Enter |
|-------|---------------|
| **Description** | e.g. `Bartizan Deploy Key` |
| **Project Permissions** | `Project Data Access Admin` (or appropriate level) |

Click **Next**.

![MongoDB Atlas — Create API Key form](images/img-002.png)

#### Step 3 · Copy Your Keys — Once Only

After clicking Next, Atlas displays your key pair **exactly once**:

```
Public Key:  qtnwswrk
Private Key: 7c09e360-cf15-48e4-a760-1c16c345522c
```

> ⚠️ **Do this immediately:**
> 1. Click **Copy** next to the Public Key
> 2. Click **Copy** next to the Private Key
> 3. Save both in a secure location
>
> Atlas warns: *"After you leave this page, the full private key is unavailable."* If lost, you must create a new key pair.

![MongoDB Atlas — Save API Key screen](images/img-003.png)

#### Step 4 · Add IP Access List *(Recommended)*

Add your server's IP address or a CIDR range to the **IP Access List** to restrict where this key can be used from. This prevents the key from being abused if ever leaked.

#### Step 5 · Click Done

Your new key pair will appear in the API Keys table on the Applications page.

**Save these values:**
```
MONGODB_PUBLIC_KEY=qtnwswrk
MONGODB_PRIVATE_KEY=7c09e360-cf15-48e4-a760-1c16c345522c
```

---

### Part 3 — Azure Storage Account

Azure Blob Storage holds the encrypted file blobs. The deploy script needs your storage account name and an access key.

#### Step 1 · Open the Azure Portal

Navigate to [portal.azure.com](https://portal.azure.com) and click **Storage accounts** from the Azure services list at the top.

> 💡 If you don't see it, click **More services** and search for `Storage accounts`.

![Microsoft Azure — Home page](images/img-004.png)

#### Step 2 · Click + Create

You'll land on the **Storage center** page. Click **+ Create** in the toolbar.

> 💡 An empty list here is completely normal if this is your first storage account.

![Azure Storage center — Resources list](images/img-005.png)

#### Step 3 · Fill in the Basics

A **Create a storage account** form will open. Fill in only the **Basics** tab:

| Field | What to Enter | Notes |
|-------|--------------|-------|
| **Subscription** | Your subscription (e.g. `Azure for Students`) | Your Azure billing account |
| **Resource group** | Select existing or click **Create new** | Groups related Azure resources |
| **Storage account name** | e.g. `bartizanstorage2026` | Lowercase + numbers only, 3–24 chars, globally unique |
| **Region** | Region closest to your users | Closer = lower latency |
| **Performance** | Standard *(pre-selected)* | Appropriate for most deployments |
| **Redundancy** | Locally-redundant storage (LRS) | 3 copies in one location — lowest cost |

> ⚠️ The storage account name must be **globally unique** across all of Azure. If taken, append numbers (e.g. `bartizan2026b`).

![Azure — Create storage account form (Basics tab)](images/img-006.png)

#### Step 4 · Skip Other Tabs

Leave all other tabs (Advanced, Networking, Data protection, Encryption) at their defaults. Skip directly to **Review + create**.

> 💡 Leave Encryption type as **Microsoft-managed keys (MMK)** — Azure handles this automatically.

![Azure — Encryption tab (default settings)](images/img-007.png)

#### Step 5 · Review and Create

Click the **Review + create** tab. If you see **Validation passed** (green), click **Create**. Deployment takes 30–60 seconds.

> ⚠️ If you see **Validation failed** (red), return to the Basics tab and check all fields marked with `*`, especially the storage account name.

![Azure — Review + create showing validation](images/img-008.png)

#### Step 6 · Retrieve Your Access Key

After deployment, click **Go to resource**, then:

1. In the left sidebar → **Security + networking** → **Access keys**
2. Click **Show** next to the Key field under `key1`
3. Copy the **Key value** or the **Connection string**

> ⚠️ Never share access keys publicly or commit them to a repository. Anyone with your key has full access to the storage account.
>
> 💡 Azure provides two keys (`key1` / `key2`) for zero-downtime rotation — you can regenerate one while the other stays active.

![Azure — Access keys page](images/img-009.png)

**Save these values:**
```
AZURE_STORAGE_ACCOUNT_NAME=bartizanstorage2026
AZURE_STORAGE_ACCESS_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

---

### Credentials Summary

Once all three parts are complete, you should have the following. Create a `.env` file in the project root:

```env
# ── Railway ────────────────────────────────────────────
RAILWAY_API_TOKEN=

# ── MongoDB Atlas ──────────────────────────────────────
MONGODB_PUBLIC_KEY=
MONGODB_PRIVATE_KEY=

# ── Azure Blob Storage ─────────────────────────────────
AZURE_STORAGE_ACCOUNT_NAME=
AZURE_STORAGE_ACCESS_KEY=
```

Then run the deploy script:

```bash
npm run deploy
# or
./deploy.sh
```

---

## Security Reminders

| Rule | Why It Matters |
|------|----------------|
| 🚫 Never commit credentials to Git | Public repos are indexed — leaks happen within minutes |
| 🔐 Use a secrets manager or password vault | Reduces the blast radius of any single breach |
| 🔄 Rotate keys periodically | Limits the damage if a key is ever compromised |
| 🌐 Use IP Access Lists (MongoDB Atlas) | Restricts key use to known network addresses |
| 🔑 Keep `key1` and `key2` in sync (Azure) | Enables zero-downtime key rotation |
| 📦 Store `.env` files outside version control | Add `.env` to your `.gitignore` |

---

<p align="center">
  <strong>Security isn't an afterthought — it's the foundation.</strong>
</p>

# 🌿 certification.webapp

## 🧾 Overview
`certification.webapp` is a **Next.js decentralized application (dApp)** designed for **document certification and verification** on the **Hedera network**.  
It allows users to **upload certification documents**, automatically process them through **OCR (Optical Character Recognition)**, verify their plausibility, and **mint them as NFTs** to ensure authenticity and traceability.

This module is part of the GreenEarthX platform ecosystem.

---

## 🏗️ Main Purpose
- Enable document upload and validation.
- Trigger OCR extraction and plausibility checks.
- Tokenize verified certifications as NFTs on Hedera.
- Provide a user-friendly decentralized interface.

---

## 🧩 System Architecture

### Components Involved
| Component | Description | Local Port |
|------------|-------------|-------------|
| **certification.webapp** | Next.js dApp for upload, OCR trigger, NFT minting | 3002 |
| **OcrPlausibilityCheck** | Python FastAPI OCR service (Docker) | 8000 |
| **PlausibilityCheck** | Plausibility algorithm micro-service (Docker) | 8001 |
| **Hedera Network** | Handles HCS messaging and NFT minting | — |

### Workflow
1. The user uploads a certification document.
2. The file is sent to `OcrPlausibilityCheck` for OCR text extraction.
3. The extracted data is verified by the `PlausibilityCheck` service.
4. Once validated, a **non-fungible token (NFT)** is minted on **Hedera** representing the certified document.
5. Document and metadata are stored on **IPFS** for decentralized access.

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-------------|
| Frontend | **Next.js 14**, **React**, **TypeScript** |
| Blockchain | **Hedera Hashgraph SDK** |
| Storage | **IPFS (InterPlanetary File System)** |
| Styling | **Tailwind CSS** |
| APIs | **Axios**, **REST API** |
| Authentication | **HashConnect Wallet** |
| Environment | **Node.js ≥ 18**, **npm or yarn** |

---

## ⚙️ Installation & Setup

### 1. Clone the repository
```bash
git clone https://github.com/GreenEarthX/certification.webapp.git
cd certification.webapp
```
### Environment (.env at root)
```bash

```

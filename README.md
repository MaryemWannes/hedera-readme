 # Building a Chain of Trust in the Green Power-to-X market
 
A **plausibility-verification + certification platform** for green-hydrogen / Power-to-X plants in Africa and beyond.  
Built on **Hedera** (HTS + HCS) with **IPFS**, **AI OCR**, **Python micro-services**, and a **global plant geomap**.

## Overview

- **Project Title:** Building a Chain of Trust in the Green Power-to-X market
- **Hackathon Track:** Track 2: DLT of Operations 
- **Team Name:** GreenEarthX  
- **Submission Date:** October 2025

### Solution Summary
**GreenEarthX** is a **full-stack certification dApp** that lets plant operators:

1. **Upload** documents (Proof-of-Sustainability, Invoice, PPA, Term-Sheet)  
2. **Run AI OCR + Plausibility Check** (Python micro-services)  
3. **Mint immutable NFTs** (Hedera HTS) with IPFS-backed proofs  
4. **Log every step on HCS** for an **audit trail**  
5. **View all plants on a global geomap** with **" Profile History "** that pulls live **Mirror-Node** for plausibility check results.  

All verification is **on-chain, cost-effective, and instantly shareable**.

---

## Architecture Diagram
<img width="1994" height="1194" alt="image" src="https://github.com/user-attachments/assets/fa4b5210-9be0-4491-bcd3-b449d299edc8" />

## Project Structure – Multi-Repo Layout

> This is a monorepo grouping all hackathon components into a single GitHub project.
> Each folder is a self-contained module with its own README.md, scripts, and deployment instructions.

| Folder                      | Purpose                                                   | Local URL                | Key README                                  |
|-----------------------------|-----------------------------------------------------------|--------------------------|---------------------------------------------|
| certification.webapp        | Next.js dApp for document upload, OCR trigger, NFT minting | http://localhost:3002    | [certification.webapp/README.md](certification.webapp/README.md) |
| Geomap.webapp               | Interactive global plant map + live Mirror Node history   | http://localhost:3001    | [Geomap.webapp/README.md](Geomap.webapp/README.md) |
| Onboarding.app              | User onboarding                                                             | http://localhost:3000    | [Onboarding.app/README.md](Onboarding.app/README.md) |
| OcrPlausibilityCheck        | Python FastAPI OCR service (Docker)                       | http://localhost:8000    | [OcrPlausibilityCheck/README.md](OcrPlausibilityCheck/README.md) |
| PlausibilityCheck           | Plausibility algorithm micro-service (Docker)             | http://localhost:8001    | [PlausibilityCheck/README.md](PlausibilityCheck/README.md) |
| Hedera-collection-and-topics| Scripts to create NFT collections & HCS topic             | —                        | [Hedera-collection-and-topics/README.md](Hedera-collection-and-topics/README.md) |
| certification_terraform     | Terraform IaC for backend infra (optional)                | —                        | [certification_terraform/README.md](certification_terraform/README.md) |
| geomap-infrastructure       | Cloud infra for Geomap (optional)                         | —                        | [geomap-infrastructure/README.md](geomap-infrastructure/README.md) |


## Hedera Integration Summary

### Services Used

#### 1. Hedera Token Service (HTS) - NFT Collections
**Why We Chose HTS:**  
We chose HTS to create 4 NFT collections that serve as **immutable records of sustainability proofs, invoices, power purchase agreements, and term sheets**. HTS eliminates the need for smart contract deployment, reducing complexity and costs. Each NFT collection provides a tamper-proof, verifiable record that can be instantly recognized across the Hedera ecosystem.

**4 NFT Collections Created on Hedera Testnet:**

| Collection Name | Token ID | Symbol | Purpose |
|-----------------|----------|--------|--------|
| **GEX Proof of Sustainability** | `0.0.7108305` | `GEX-POS` | Immutable Proof of Sustainability (PoS) for green H2/RFNBO compliance |
| **GEX Invoices** | `0.0.7108304` | `GEX-INV` | Verified sales invoices linked to certified production |
| **GEX Power Purchase Agreement** | `0.0.7108301` | `GEX-PPA` | Tokenized long-term offtake contracts |
| **TermSheet NFT Collection** | `0.0.7131752` | `TERM-NFT` | Legal & financial term sheets for project financing |

> **Each NFT contains:**  
> - `ipfs://<CID>` → PDF document  
> - `ipfs://<OCR_CID>` → AI-extracted structured data  (optional) 
> - Timestamp

**Transaction Types:**  
- `TokenCreateTransaction` - Creating 4 NFT collections with metadata
- `TokenMintTransaction` - Issuing NFTs to the collections with metadata URIs
- `TokenTransferTransaction` - Transferring NFTs to recipients

**Key Function (`certification.webapp/src/lib/hedera.ts`):**
```ts
mintNFT(metadata: { name: string; image: string }, tokenIdString: string)
```

**Economic Justification:**  
HTS's predictable, low fees (~$0.001 per mint) make it economically viable to issue large volumes of NFTs in African markets where transaction costs directly impact adoption. The built-in royalty capabilities and standardized metadata format ensure sustainability without expensive smart contract maintenance. This is critical for our business model targeting **green hydrogen and Power-to-X developers in Africa**.

---

#### 2. Hedera Consensus Service (HCS) - Event Logging
**Why We Chose HCS:**  
We created dedicated HCS topic to log all NFT issuance and plausibility check events immutably. Each event submission creates an auditable trail of when NFTs were created, issued, and verified. This provides transparency and accountability essential for **sustainability certification and regulatory compliance**.

**Transaction Types:**  
- `TopicCreateTransaction` - Setting up HCS topic for event logging
- `TopicMessageSubmitTransaction` - Recording issuance events: who issued an NFT, when, which collection, and associated metadata

**HCS Event Payload (logged on mint):**
```json 
{
  "event": "NFT_MINTED",
  "documentType": "invoice",
  "collection": "0.0.7108304",
  "serial": "1",
  "ipfs": "ipfs://bafy...",
  "timestamp": "2025-10-29T12:00:00Z"
}
```

**Economic Justification:**  
HCS's $0.0001 per message fee ensures we can log comprehensive audit trails without incurring prohibitive costs. This is especially valuable in Africa where operational cost predictability is essential for business sustainability. Every transaction is immutably recorded and timestamped by Hedera's Byzantine Fault Tolerant consensus.

---

#### 3. IPFS Integration - File Storage
**Why We Chose IPFS:**  
We use IPFS to decentralized store actual files (documents, images, certificates) and store only the IPFS content hash (CID) within the NFT metadata. This approach provides several advantages:
- **Scalability:** NFTs remain lightweight; heavy files live off-chain
- **Permanence:** Files pinned to IPFS ensure long-term availability
- **Verifiability:** The immutable hash proves file integrity and authenticity

**Implementation:**  
- Upload files to IPFS using `uploadToPinata(buffer, filename)`
- Receive IPFS CID (content hash)
- Embed `ipfs://<CID>` in NFT metadata URI
- Store reference in Hedera NFT

**Key Function (`certification.webapp/src/lib/ipfs.ts`):**
```ts
uploadToPinata(data: ArrayBuffer | Uint8Array, filename: string): Promise<string>
```

**Economic Justification:**  
By separating file storage from blockchain transactions, we reduce on-chain costs while maintaining cryptographic proof of file authenticity. IPFS ensures files remain accessible without reliance on centralized servers vulnerable to censorship or downtime—critical for African markets with infrastructure challenges.

---

#### 4. Hedera Mirror Node - Transaction History & Verification
**Why We Chose Mirror Node:**  
We query Hedera Mirror Node Explorer to retrieve complete transaction histories, verify NFT creation events, and display immutable audit trails to users. This allows real-time verification without requiring users to run their own nodes.

**Transaction Types:**  
- Mirror Node REST API queries for:
  - Token transaction history (all mints and transfers)
  - HCS topic message retrieval (all issuance events)
  - Account activity and balance verification

**Mirror Node Queries Used:**

- `GET /tokens/{tokenId}/nfts` → All minted NFTs
- `GET /topics/{topicId}/messages` → HCS event log
- `GET /accounts/{id}/tokens` → Operator balance  

**Economic Justification:**  
Mirror Node queries are free and publicly accessible, enabling cost-free verification and transparency. Users in Africa can instantly prove authenticity and ownership without intermediaries, reducing friction and building trust in the ecosystem.

---




## Deployed Hedera Testnet IDs

| Component | ID | Purpose |
|-----------|----|---------| 
| GEX-POS | `0.0.7108305` | Proof of Sustainability |
| GEX-INV | `0.0.7108304` | Invoices |
| GEX-PPA | `0.0.7108301` | Power Purchase Agreements |
| TERM-NFT | `0.0.7131752` | Term Sheets |
| HCS Topic  | `0.0.7108913` | All issuance & plausibility events|
| Testnet Account ID | `0.0.7081162` | Main transaction signer |

> Live Verification Links:
> - [HashScan POS](https://hashscan.io/testnet/token/0.0.7108305)
> - [HashScan HCS Topic](https://hashscan.io/testnet/topic/0.0.7108913)

---

## Setup & Deployment Instructions

### Prerequisites
- Node.js (v16 or higher)
- npm or yarn
- Git
- Hedera Testnet Account (with ℏ testnet tokens)
- IPFS access (Pinata API key or local IPFS node)
- Hedera SDK
- docker for pulling python services


### Installation

1. **Clone the Repository**
   ```bash
   git clone https://github.com/your-username/your-repo.git
   cd your-repo
   ```

2. **Install Dependencies**
   ```bash
   npm install
   ```

3. **Configure Environment Variables**
   ```bash
   cp .env.example .env
   ```
   
   Update `.env` with your credentials:
   ```
   HEDERA_ACCOUNT_ID=0.0.xxxxx
   HEDERA_PRIVATE_KEY=your-private-key
   HEDERA_NETWORK=testnet
   
   IPFS_API_KEY=your-pinata-api-key
   IPFS_API_SECRET=your-pinata-api-secret
   
   NFT_COLLECTION_ID_1=0.0.xxxxx
   NFT_COLLECTION_ID_2=0.0.xxxxx
   NFT_COLLECTION_ID_3=0.0.xxxxx
   NFT_COLLECTION_ID_4=0.0.xxxxx
   
   HCS_TOPIC_ID=0.0.xxxxx
   
   MIRROR_NODE_URL=https://testnet.mirrornode.hedera.com
   
   BACKEND_PORT=5000
   FRONTEND_PORT=3000
   ```

4. **Deploy NFT Collections & Create HCS Topic**
   ```bash
   cd Hedera-collection-and-topics
   node createGexInvoices.js
   node createGexPowerPurchase.js
   node createGexSustainability.js
   node createGexTermsheet.js
   node createGexTraceTopic.js 
   ```
   This will create the 4 NFT collections and HCS topic, outputting their IDs to update in `.env`. 
   > Note: Already done for the hackathon – IDs are shared here.
   

### Running the Project Locally

**Frontend:**
we have different Two Frontends GeoMap AND Certification.webapp and the Onboarding Service so we need to run each one seperatly
```bash
npm run dev
```
The Nextjs frontend will launch on `http://localhost:3000` for OnboardingApp, on `http://localhost:3001` for the GeoMap and `http://localhost:3002` for Certification.webapp !

**Backend:**
OCR and Plausibility check services : pull docker images

```bash
docker pull medbnk/plausibility_ocrs:latest
docker pull medbnk/plausibilityalgorithm:latest
```
The backend will start on `http://localhost:8000` and `http://localhost:8001`

**Expected Running State:**
- Frontend accessible at `http://localhost:3002` with upload and NFT creation interfaces
- Backend API running at `http://localhost:8000` handling data extraction from pdf documents that gonna be uploaded to IPFS and minted to Hedera NFT Collection using HTS and tracked using HCS service.
- 4 NFT collections active on Hedera Testnet
- HCS topic listening for issuance event messages
- Mirror Node queries retrieving live transaction history viewed in the GeoMap `http://localhost:3001`
---

## Key Features & Workflows

### 1. File Upload to IPFS
- User uploads file via frontend
- Backend sends file to IPFS (Pinata)
- Backend receives IPFS CID (content hash)
- Hash stored in NFT metadata

### 2. NFT Issuance
- Backend creates NFT with metadata containing IPFS CID
- Executes `TokenMintTransaction` on selected collection
- Logs event to HCS topic with timestamp and details
- Returns transaction hash and NFT ID to frontend

### 3. Transaction History Retrieval
- Frontend queries Hedera Mirror Node via backend
- Displays all mint transactions, transfers, and HCS events
- Shows immutable audit trail with timestamps and participants
- Allows users to verify NFT authenticity and provenance

---

## Security & Credentials

### Critical Security Notice
**⚠️ DO NOT commit private keys, secrets, or sensitive credentials to this repository.**

### Configuration Example
Create a `.env.example` file showing the structure (without actual values):

```env
# Hedera Network Configuration
HEDERA_ACCOUNT_ID=0.0.xxxxx
HEDERA_PRIVATE_KEY=xxxx...xxxx
HEDERA_NETWORK=testnet

# IPFS Configuration
IPFS_API_KEY=your_pinata_api_key
IPFS_API_SECRET=your_pinata_api_secret

# NFT Collection IDs
NFT_COLLECTION_ID_1=0.0.xxxxx
NFT_COLLECTION_ID_2=0.0.xxxxx
NFT_COLLECTION_ID_3=0.0.xxxxx
NFT_COLLECTION_ID_4=0.0.xxxxx

# HCS Topic
HCS_TOPIC_ID=0.0.xxxxx

# Hedera Mirror Node
MIRROR_NODE_URL=https://testnet.mirrornode.hedera.com

# Application Configuration
BACKEND_PORT=5000
FRONTEND_PORT=3000
```

### Judge Access Instructions
Test account credentials and IPFS keys for judges will be provided **separately in the DoraHacks BUIDL submission** to ensure security. Judges should:

1. Copy the Test Hedera Account ID and Private Key from the DoraHacks submission notes
2. Copy the IPFS API credentials
3. Add these credentials to their local `.env` file
4. Run `npm start` to launch the application with test credentials

---

## Project Files & Structure


---

## Additional Resources

- [Hedera Documentation](https://docs.hedera.com/)
- [Hedera SDK (JavaScript)](https://github.com/hashgraph/hedera-sdk-js)
- [HTS Token Service Docs](https://docs.hedera.com/hedera/sdks-and-apis/token-service)
- [HCS Consensus Service Docs](https://docs.hedera.com/hedera/sdks-and-apis/hedera-consensus-service)
- [Hedera Mirror Node API](https://mainnet-public.mirrornode.hedera.com/)
- [IPFS Documentation](https://docs.ipfs.io/)
- [Pinata IPFS Pinning Service](https://www.pinata.cloud/)

---

## Support & Contact

**Team Contact:** [marwen123.c@gmail.com ]  
**GitHub Issues:** [Link to your GitHub repo issues]  

---

*Prepared for Hedera Africa Hackathon 2025*

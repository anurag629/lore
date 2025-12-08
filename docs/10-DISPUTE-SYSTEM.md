# Dispute System Guide

## Overview

The Dispute System allows users to challenge IP assets for various reasons, such as copyright infringement, trademark violations, or other IP-related concerns. The system provides a structured workflow for raising disputes, submitting evidence, and resolving conflicts.

---

## Table of Contents

1. [What is a Dispute?](#what-is-a-dispute)
2. [Dispute Workflow](#dispute-workflow)
3. [Raising a Dispute](#raising-a-dispute)
4. [Submitting Evidence](#submitting-evidence)
5. [Resolving Disputes](#resolving-disputes)
6. [API Reference](#api-reference)
7. [Examples](#examples)

---

## What is a Dispute?

A **Dispute** is a formal challenge raised against an IP asset. Disputes can be raised for various reasons:

- **Copyright Infringement**: Unauthorized use of copyrighted material
- **Trademark Violation**: Use of protected trademarks
- **Plagiarism**: Direct copying of creative work
- **Misattribution**: Incorrect attribution or ownership claims
- **Other IP Violations**: Any other intellectual property concerns

### Dispute Statuses

- **Pending**: Dispute has been raised, awaiting review
- **Under Review**: Dispute is being reviewed by administrators
- **Resolved**: Dispute has been resolved with a decision
- **Cancelled**: Dispute was cancelled by the disputer

### Dispute Results

- **Upheld**: Dispute is valid - the challenged asset violates IP rights
- **Rejected**: Dispute is invalid - no violation found
- **Settled**: Parties reached an agreement

---

## Dispute Workflow

```
1. Raise Dispute
   ↓
2. Submit Evidence (Disputer)
   ↓
3. Asset Owner Responds (Optional)
   ↓
4. Additional Evidence (Both Parties)
   ↓
5. Admin Review
   ↓
6. Resolution
```

---

## Raising a Dispute

### Step 1: Prepare Information

Before raising a dispute, gather:
- Target asset UUID
- Clear reason for the dispute
- Initial evidence (if available)
- Evidence description

### Step 2: Raise the Dispute

```http
POST /api/disputes/raise_dispute/
Authorization: Bearer <token>
Content-Type: application/json

{
  "target_asset_uuid": "...",
  "reason": "Copyright infringement - this asset uses my copyrighted material without permission",
  "evidence_description": "The asset directly copies elements from my original work",
  "evidence_url": "ipfs://QmX5Z..." // Optional
}
```

**Response:**
```json
{
  "uuid": "...",
  "target_asset": {
    "uuid": "...",
    "title": "Mystical Forest Path"
  },
  "disputer": {
    "wallet_address": "0x...",
    "username": "disputer123"
  },
  "reason": "Copyright infringement...",
  "status": "pending",
  "evidence_count": 1,
  "raise_transaction_hash": "0xabc123...",
  "raised_at": "2024-12-04T10:30:00Z"
}
```

### Requirements

- Must be authenticated
- Target asset must exist
- Reason must be provided
- Initial evidence is recommended

---

## Submitting Evidence

Both the disputer and the asset owner can submit evidence.

### Disputer Submitting Evidence

```http
POST /api/disputes/{uuid}/submit_evidence/
Authorization: Bearer <token>
Content-Type: application/json

{
  "description": "Additional evidence showing the similarity between my work and the disputed asset",
  "evidence_url": "ipfs://QmY6A..." // Optional
}
```

### Asset Owner Submitting Evidence

The asset owner can also submit evidence to defend their asset:

```http
POST /api/disputes/{uuid}/submit_evidence/
Authorization: Bearer <token>
Content-Type: application/json

{
  "description": "Evidence showing original creation process and independent development",
  "evidence_url": "ipfs://QmZ7B..." // Optional
}
```

### Viewing Evidence

```http
GET /api/disputes/{uuid}/evidence/
```

**Response:**
```json
{
  "count": 3,
  "results": [
    {
      "uuid": "...",
      "description": "Evidence description",
      "evidence_url": "ipfs://...",
      "submitted_by": {
        "wallet_address": "0x...",
        "username": "disputer123"
      },
      "submitted_at": "2024-12-04T10:35:00Z"
    }
  ]
}
```

---

## Resolving Disputes

### Admin Resolution

Only administrators can resolve disputes:

```http
POST /api/disputes/{uuid}/resolve/
Authorization: Bearer <admin_token>
Content-Type: application/json

{
  "result": "upheld", // upheld, rejected, or settled
  "resolution_notes": "After review, the dispute is valid. The asset infringes on copyright."
}
```

**Response:**
```json
{
  "success": true,
  "message": "Dispute resolved successfully",
  "dispute": {
    "uuid": "...",
    "status": "resolved",
    "result": "upheld",
    "resolution_notes": "After review...",
    "resolved_by": {
      "wallet_address": "0x...",
      "username": "admin123"
    },
    "resolve_transaction_hash": "0xghi789...",
    "resolved_at": "2024-12-04T15:00:00Z"
  }
}
```

### Cancelling a Dispute

The disputer can cancel their own dispute:

```http
POST /api/disputes/{uuid}/cancel/
Authorization: Bearer <token>
```

**Response:**
```json
{
  "success": true,
  "message": "Dispute cancelled successfully",
  "dispute": {
    "uuid": "...",
    "status": "cancelled",
    "resolved_at": "2024-12-04T15:00:00Z"
  }
}
```

---

## API Reference

### Endpoints Summary

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| GET | `/api/disputes/` | List all disputes | Yes |
| POST | `/api/disputes/raise_dispute/` | Raise new dispute | Yes |
| GET | `/api/disputes/{uuid}/` | Get dispute details | Yes |
| POST | `/api/disputes/{uuid}/submit_evidence/` | Submit evidence | Yes (disputer or owner) |
| POST | `/api/disputes/{uuid}/resolve/` | Resolve dispute | Yes (admin only) |
| POST | `/api/disputes/{uuid}/cancel/` | Cancel dispute | Yes (disputer only) |
| GET | `/api/disputes/{uuid}/evidence/` | Get evidence | Yes |
| GET | `/api/disputes/statistics/` | Get statistics | Yes |

### Models

#### Dispute
- `uuid` - Public UUID
- `story_dispute_id` - Story Protocol Dispute ID (on-chain)
- `target_asset` - IP asset being disputed
- `disputer` - User who raised the dispute
- `reason` - Reason for dispute
- `status` - pending/under_review/resolved/cancelled
- `result` - upheld/rejected/settled
- `resolution_notes` - Notes from resolution
- `resolved_by` - User/admin who resolved
- `raise_transaction_hash` - Transaction hash when raised
- `resolve_transaction_hash` - Transaction hash when resolved
- `raised_at` - When dispute was raised
- `resolved_at` - When dispute was resolved

#### DisputeEvidence
- `uuid` - Public UUID
- `dispute` - Foreign key to Dispute
- `submitted_by` - User who submitted
- `description` - Description of evidence
- `evidence_url` - URL to evidence file (IPFS or external)
- `evidence_ipfs_hash` - IPFS hash of evidence
- `transaction_hash` - Blockchain transaction hash
- `submitted_at` - When evidence was submitted

---

## Examples

### Example 1: Raise Copyright Dispute

```typescript
// Raise dispute
const raiseDispute = async (targetAssetUuid: string, reason: string) => {
  const response = await api.post('/api/disputes/raise_dispute/', {
    target_asset_uuid: targetAssetUuid,
    reason: reason,
    evidence_description: "The asset directly copies my original artwork",
    evidence_url: "ipfs://QmX5Z..." // Optional
  });
  return response.data;
};

// Submit additional evidence
const submitEvidence = async (disputeUuid: string, description: string) => {
  const response = await api.post(`/api/disputes/${disputeUuid}/submit_evidence/`, {
    description: description,
    evidence_url: "ipfs://QmY6A..." // Optional
  });
  return response.data;
};
```

### Example 2: Asset Owner Response

```typescript
// Asset owner submits defense evidence
const submitDefense = async (disputeUuid: string) => {
  const response = await api.post(`/api/disputes/${disputeUuid}/submit_evidence/`, {
    description: "Evidence showing original creation process, timestamps, and independent development",
    evidence_url: "ipfs://QmZ7B..."
  });
  return response.data;
};
```

### Example 3: Admin Resolution

```typescript
// Admin resolves dispute
const resolveDispute = async (disputeUuid: string, result: string, notes: string) => {
  const response = await api.post(`/api/disputes/${disputeUuid}/resolve/`, {
    result: result, // "upheld", "rejected", or "settled"
    resolution_notes: notes
  });
  return response.data;
};
```

---

## Best Practices

### For Disputers

1. **Clear Documentation**: Provide clear, detailed reasons for the dispute
2. **Strong Evidence**: Submit compelling evidence (screenshots, timestamps, original files)
3. **IPFS Storage**: Upload evidence to IPFS for permanent, verifiable storage
4. **Professional Tone**: Maintain a professional and factual tone
5. **Follow Up**: Monitor the dispute and submit additional evidence if needed

### For Asset Owners

1. **Quick Response**: Respond promptly to disputes
2. **Defense Evidence**: Submit evidence showing original creation
3. **Documentation**: Keep records of creation process, timestamps, and sources
4. **Cooperation**: Work with administrators during review

### For Administrators

1. **Thorough Review**: Carefully review all evidence from both parties
2. **Fair Assessment**: Make decisions based on evidence, not assumptions
3. **Clear Communication**: Provide clear resolution notes explaining the decision
4. **Timely Resolution**: Resolve disputes in a timely manner

---

## Dispute Statistics

View platform-wide dispute statistics:

```http
GET /api/disputes/statistics/
```

**Response:**
```json
{
  "total_disputes": 25,
  "pending": 5,
  "under_review": 3,
  "resolved": 15,
  "cancelled": 2,
  "upheld": 8,
  "rejected": 5,
  "settled": 2
}
```

---

## Limitations

- Only authenticated users can raise disputes
- Only disputers can cancel their own disputes
- Only administrators can resolve disputes
- Evidence must be submitted via IPFS or external URLs
- Disputes cannot be edited after creation (only cancelled)

---

## Troubleshooting

### "Cannot raise dispute - asset does not exist"
**Solution:** Verify the asset UUID is correct and the asset exists

### "Cannot submit evidence - not disputer or owner"
**Solution:** Only the disputer or asset owner can submit evidence

### "Cannot resolve dispute - not admin"
**Solution:** Only administrators can resolve disputes

### "Dispute already resolved"
**Solution:** Resolved disputes cannot be modified

---

**Last Updated:** January 2025  
**Related:** [API Documentation](./06-API-DOCUMENTATION.md) | [Solution Architecture](./03-SOLUTION-ARCHITECTURE.md)


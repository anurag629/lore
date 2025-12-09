# API Documentation

## Overview

The **Lore API** is a RESTful API built with Django REST Framework. It provides endpoints for user authentication, IP asset management, derivative creation, and AI-powered content generation.

**Base URL:** `http://localhost:8000/api/` (Development)

**Authentication:** JWT Bearer Token (obtained via SIWE)

**Content Type:** `application/json`

---

## Table of Contents

1. [Authentication Endpoints](#authentication-endpoints)
2. [IP Asset Endpoints](#ip-asset-endpoints)
3. [Derivative Asset Endpoints](#derivative-asset-endpoints)
4. [Group IP Endpoints](#group-ip-endpoints)
5. [Dispute Endpoints](#dispute-endpoints)
6. [Permission Endpoints](#permission-endpoints)
7. [Collections Endpoints](#collections-endpoints)
8. [Favorites Endpoints](#favorites-endpoints)
9. [Comments Endpoints](#comments-endpoints)
10. [User Profile Endpoints](#user-profile-endpoints)
11. [AI Generation Endpoints](#ai-generation-endpoints)
12. [Analytics Endpoints](#analytics-endpoints)
13. [Error Responses](#error-responses)
14. [Rate Limiting](#rate-limiting)

---

## Authentication Endpoints

### 1. Generate Nonce

**Generate a nonce for SIWE (Sign-In With Ethereum) authentication.**

```http
POST /api/auth/generate-nonce/
```

**Request Body:** None

**Response:**
```json
{
  "nonce": "8f3e9c2b1a4d6f5e7c9b0d1f2a3e4b5c",
  "message": "localhost:3000 wants you to sign in with your Ethereum account:\n0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb\n\nSign in to Lore Platform\n\nURI: http://localhost:3000\nVersion: 1\nChain ID: 84532\nNonce: 8f3e9c2b1a4d6f5e7c9b0d1f2a3e4b5c\nIssued At: 2024-12-04T10:30:00Z"
}
```

**Status Codes:**
- `200 OK` - Nonce generated successfully

---

### 2. Verify Signature

**Verify SIWE signature and issue JWT tokens.**

```http
POST /api/auth/verify-signature/
```

**Request Body:**
```json
{
  "message": "localhost:3000 wants you to sign in...",
  "signature": "0xabc123def456...",
  "wallet_address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"
}
```

**Response:**
```json
{
  "access": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 1,
    "wallet_address": "0x742d35cc6634c0532925a3b844bc9e7595f0beb",
    "username": "0x742d35c"
  }
}
```

**Status Codes:**
- `200 OK` - Signature verified, tokens issued
- `400 Bad Request` - Invalid signature or expired nonce
- `422 Unprocessable Entity` - Validation error

**Error Response:**
```json
{
  "error": "Invalid signature",
  "detail": "Signature verification failed"
}
```

---

### 3. Refresh Token

**Refresh access token using refresh token.**

```http
POST /api/auth/refresh-token/
```

**Request Body:**
```json
{
  "refresh": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response:**
```json
{
  "access": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Status Codes:**
- `200 OK` - Token refreshed
- `401 Unauthorized` - Invalid or expired refresh token

---

## IP Asset Endpoints

### 4. List User's IP Assets

**Get all IP assets created by the authenticated user.**

```http
GET /api/assets/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `asset_type` (optional) - Filter by type: `digital_art`, `music`, `writing`, `photography`, `game_asset`
- `ordering` (optional) - Sort by field: `-created_at` (default), `title`, `-updated_at`
- `page` (optional) - Page number (default: 1)
- `page_size` (optional) - Items per page (default: 20, max: 100)

**Response:**
```json
{
  "count": 42,
  "next": "http://localhost:8000/api/assets/?page=2",
  "previous": null,
  "results": [
    {
      "id": 1,
      "user": 1,
      "title": "Mystical Forest Path",
      "description": "A hauntingly beautiful digital artwork depicting a winding path...",
      "asset_type": "digital_art",
      "media_url": "ipfs://QmX5Z9Y8W7V6U5T4S3R2Q1P0O9N8M7L6K5J4I3H2G1F",
      "metadata_uri": "ipfs://QmA1B2C3D4E5F6G7H8I9J0K1L2M3N4O5P6Q7R8S9T0U",
      "ip_id": "0x123abc456def789ghi012jkl345mno678pqr901stu234vwx",
      "token_id": 1,
      "created_at": "2024-12-04T10:30:00Z",
      "updated_at": "2024-12-04T10:30:00Z"
    }
  ]
}
```

**Status Codes:**
- `200 OK` - Assets retrieved successfully
- `401 Unauthorized` - Missing or invalid auth token

---

### 5. Get IP Asset Details

**Get details of a specific IP asset.**

```http
GET /api/assets/{id}/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "id": 1,
  "user": 1,
  "title": "Mystical Forest Path",
  "description": "A hauntingly beautiful digital artwork...",
  "asset_type": "digital_art",
  "media_url": "ipfs://QmX5Z...",
  "metadata_uri": "ipfs://QmA1B...",
  "ip_id": "0x123abc...",
  "token_id": 1,
  "created_at": "2024-12-04T10:30:00Z",
  "updated_at": "2024-12-04T10:30:00Z",
  "derivatives": [
    {
      "id": 2,
      "child_asset_id": 5,
      "relationship_type": "remix"
    }
  ]
}
```

**Status Codes:**
- `200 OK` - Asset retrieved successfully
- `404 Not Found` - Asset does not exist
- `401 Unauthorized` - Missing or invalid auth token

---

### 6. Upload File to IPFS

**Upload media file (image, audio, video) to IPFS via Pinata.**

```http
POST /api/assets/upload-to-ipfs/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: multipart/form-data
```

**Request Body:**
```
file: <binary file data>
```

**Response:**
```json
{
  "media_url": "ipfs://QmX5Z9Y8W7V6U5T4S3R2Q1P0O9N8M7L6K5J4I3H2G1F"
}
```

**Status Codes:**
- `200 OK` - File uploaded successfully
- `400 Bad Request` - No file provided
- `500 Internal Server Error` - Pinata upload failed

**cURL Example:**
```bash
curl -X POST http://localhost:8000/api/assets/upload-to-ipfs/ \
  -H "Authorization: Bearer <access_token>" \
  -F "file=@/path/to/image.png"
```

---

### 7. Register IP Asset on Blockchain

**Register an IP asset on Story Protocol blockchain.**

```http
POST /api/assets/register-ip/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "title": "Mystical Forest Path",
  "description": "A hauntingly beautiful digital artwork depicting a winding path through an enchanted forest with glowing mushrooms and ethereal mist.",
  "asset_type": "digital_art",
  "media_url": "ipfs://QmX5Z9Y8W7V6U5T4S3R2Q1P0O9N8M7L6K5J4I3H2G1F"
}
```

**Response:**
```json
{
  "asset": {
    "id": 1,
    "user": 1,
    "title": "Mystical Forest Path",
    "description": "A hauntingly beautiful digital artwork...",
    "asset_type": "digital_art",
    "media_url": "ipfs://QmX5Z...",
    "metadata_uri": "ipfs://QmA1B...",
    "ip_id": "0x123abc456def789ghi012jkl345mno678pqr901stu234vwx",
    "token_id": 1,
    "created_at": "2024-12-04T10:30:00Z",
    "updated_at": "2024-12-04T10:30:00Z"
  },
  "tx_hash": "0xdef456abc789ghi012jkl345mno678pqr901stu234vwx567yza890bcd"
}
```

**Status Codes:**
- `201 Created` - IP asset registered successfully
- `400 Bad Request` - Missing required fields
- `500 Internal Server Error` - Blockchain registration failed

**Process:**
1. Creates metadata JSON with title, description, image, attributes
2. Uploads metadata to IPFS (Pinata)
3. Registers IP asset on Story Protocol blockchain
4. Saves asset record to PostgreSQL database
5. Returns asset details and blockchain transaction hash

---

### 8. Update IP Asset

**Update an existing IP asset (only title and description can be updated).**

```http
PATCH /api/assets/{id}/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "title": "Updated Title",
  "description": "Updated description..."
}
```

**Response:**
```json
{
  "id": 1,
  "title": "Updated Title",
  "description": "Updated description...",
  "updated_at": "2024-12-04T11:00:00Z"
}
```

**Status Codes:**
- `200 OK` - Asset updated successfully
- `404 Not Found` - Asset does not exist
- `403 Forbidden` - User does not own this asset

**Note:** Blockchain data (ip_id, token_id, metadata_uri) cannot be modified after registration.

---

### 9. Delete IP Asset

**Delete an IP asset (soft delete - marks as inactive).**

```http
DELETE /api/assets/{id}/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "message": "Asset deleted successfully"
}
```

**Status Codes:**
- `204 No Content` - Asset deleted successfully
- `404 Not Found` - Asset does not exist
- `403 Forbidden` - User does not own this asset

**Note:** Blockchain record remains immutable. Only database record is marked inactive.

---

### 9a. Restore Archived Asset

**Restore a soft-deleted (archived) asset.**

```http
POST /api/assets/{uuid}/restore/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "success": true,
  "message": "Asset restored successfully"
}
```

**Status Codes:**
- `200 OK` - Asset restored successfully
- `400 Bad Request` - Asset is not archived
- `403 Forbidden` - User does not own this asset
- `404 Not Found` - Asset does not exist

---

### 9b. Permanently Delete Asset

**Permanently delete an archived asset from the database.**

```http
DELETE /api/assets/{uuid}/permanent_delete/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "success": true,
  "message": "Asset permanently deleted"
}
```

**Status Codes:**
- `200 OK` - Asset permanently deleted
- `400 Bad Request` - Asset must be archived before permanent deletion
- `403 Forbidden` - User does not own this asset
- `404 Not Found` - Asset does not exist

**Note:** Asset must be soft-deleted (archived) first. Blockchain registration cannot be removed (blockchain is immutable).

---

### 9c. Retry Asset Registration

**Retry Story Protocol registration for a failed asset.**

```http
POST /api/assets/{uuid}/retry_registration/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "success": true,
  "message": "Registration retry initiated",
  "asset": {
    "uuid": "...",
    "registration_status": "retrying",
    "creation_step": "story_registration"
  }
}
```

**Status Codes:**
- `200 OK` - Retry initiated
- `400 Bad Request` - Asset is not in a failed state
- `403 Forbidden` - User does not own this asset
- `404 Not Found` - Asset does not exist

---

### 9d. Retry Asset Creation

**Comprehensive retry endpoint that resumes asset creation from the failed step.**

```http
POST /api/assets/{uuid}/retry_creation/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "success": true,
  "message": "Creation retry initiated",
  "asset": {
    "uuid": "...",
    "registration_status": "retrying",
    "creation_step": "metadata_upload"
  }
}
```

**Status Codes:**
- `200 OK` - Retry initiated
- `400 Bad Request` - Asset is not in a failed state
- `403 Forbidden` - User does not own this asset
- `404 Not Found` - Asset does not exist

**Note:** This endpoint automatically resumes from the step where creation failed, using stored step data.

---

### 9e. Get Asset Derivatives

**Get all derivatives of a specific IP asset.**

```http
GET /api/assets/{uuid}/derivatives/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "count": 3,
  "results": [
    {
      "uuid": "...",
      "title": "Neon Forest Remix",
      "creator": {
        "wallet_address": "0x...",
        "username": "artist123"
      },
      "created_at": "2024-12-04T12:00:00Z"
    }
  ]
}
```

**Status Codes:**
- `200 OK` - Derivatives retrieved successfully
- `404 Not Found` - Asset does not exist

---

### 9f. Get Asset Royalty Balance

**Get current royalty balance for an IP asset.**

```http
GET /api/assets/{uuid}/royalty_balance/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "asset_uuid": "...",
  "balance_wei": "1000000000000000000",
  "balance_eth": "1.0",
  "currency": "ETH"
}
```

**Status Codes:**
- `200 OK` - Balance retrieved successfully
- `404 Not Found` - Asset does not exist

---

### 9g. Claim Royalties

**Claim accumulated royalties for an IP asset.**

```http
POST /api/assets/{uuid}/claim_royalties/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "success": true,
  "message": "Royalties claimed successfully",
  "tx_hash": "0xabc123...",
  "amount_wei": "1000000000000000000",
  "amount_eth": "1.0"
}
```

**Status Codes:**
- `200 OK` - Royalties claimed successfully
- `400 Bad Request` - No royalties to claim or insufficient balance
- `403 Forbidden` - User does not own this asset
- `404 Not Found` - Asset does not exist
- `500 Internal Server Error` - Blockchain transaction failed

---

## Derivative Asset Endpoints

### 10. List Derivatives

**Get all derivative assets (where user is parent or child).**

```http
GET /api/assets/derivatives/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "count": 5,
  "results": [
    {
      "id": 1,
      "child_asset": {
        "id": 5,
        "title": "Neon Forest Remix",
        "media_url": "ipfs://..."
      },
      "parent_asset": {
        "id": 1,
        "title": "Mystical Forest Path",
        "media_url": "ipfs://..."
      },
      "relationship_type": "remix",
      "derivative_ip_id": "0x456def...",
      "tx_hash": "0x789ghi...",
      "created_at": "2024-12-04T12:00:00Z"
    }
  ]
}
```

**Status Codes:**
- `200 OK` - Derivatives retrieved successfully

---

### 11. Create Derivative Asset

**Create a derivative asset linked to parent(s).**

```http
POST /api/assets/derivatives/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "parent_asset_id": 1,
  "child_title": "Neon Forest Remix",
  "child_description": "A cyberpunk-inspired remix of the mystical forest, featuring neon colors and futuristic elements.",
  "child_media_url": "ipfs://QmY6A1B2C3D4E5F6G7H8I9J0K1L2M3N4O5P6Q7R8S9T0U",
  "relationship_type": "remix",
  "royalty_percentage": 10
}
```

**Response:**
```json
{
  "derivative": {
    "id": 1,
    "child_asset_id": 5,
    "parent_asset_id": 1,
    "relationship_type": "remix",
    "derivative_ip_id": "0x456def789ghi012jkl345mno678pqr901stu234vwx567yza",
    "tx_hash": "0x890bcd123efg456hij789klm012nop345qrs678tuv901wxy",
    "created_at": "2024-12-04T12:00:00Z"
  },
  "child_asset": {
    "id": 5,
    "title": "Neon Forest Remix",
    "description": "A cyberpunk-inspired remix...",
    "ip_id": "0x456def...",
    "token_id": 5
  },
  "tx_hash": "0x890bcd..."
}
```

**Status Codes:**
- `201 Created` - Derivative created successfully
- `400 Bad Request` - Invalid parent asset or missing fields
- `500 Internal Server Error` - Blockchain registration failed

**Process:**
1. Registers child as separate IP asset on Story Protocol
2. Links child to parent(s) on blockchain (license terms)
3. Smart contract enforces royalty percentage automatically
4. Saves derivative relationship to database

---

### 11a. Create Multi-Parent Derivative

**Create a derivative asset with multiple parent assets (1-10 parents).**

```http
POST /api/assets/create_multi_parent_derivative/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "parent_assets": [
    {
      "asset_uuid": "uuid-1",
      "attribution_percentage": 60
    },
    {
      "asset_uuid": "uuid-2",
      "attribution_percentage": 40
    }
  ],
  "child_title": "Multi-Parent Remix",
  "child_description": "A remix combining elements from multiple parent assets.",
  "child_media_url": "ipfs://QmY6A1B2C3D4E5F6G7H8I9J0K1L2M3N4O5P6Q7R8S9T0U",
  "relationship_type": "remix",
  "royalty_percentage": 10
}
```

**Response:**
```json
{
  "derivative": {
    "uuid": "...",
    "child_asset": {
      "uuid": "...",
      "title": "Multi-Parent Remix",
      "story_ip_id": "0x456def..."
    },
    "parent_assets": [
      {
        "uuid": "uuid-1",
        "attribution_percentage": 60
      },
      {
        "uuid": "uuid-2",
        "attribution_percentage": 40
      }
    ],
    "relationship_type": "remix",
    "created_at": "2024-12-04T12:00:00Z"
  },
  "tx_hash": "0x890bcd..."
}
```

**Status Codes:**
- `201 Created` - Multi-parent derivative created successfully
- `400 Bad Request` - Invalid parent assets, attribution percentages don't sum to 100%, or missing fields
- `500 Internal Server Error` - Blockchain registration failed

**Validation Rules:**
- Must have 1-10 parent assets
- Attribution percentages must sum to exactly 100%
- All parent assets must allow derivatives
- All parent assets must be registered on Story Protocol

---

### 12. Get Royalty Payments

**Get royalty payments received by the authenticated user.**

```http
GET /api/assets/royalties/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "count": 3,
  "total_earned_wei": "1000000000000000000",
  "total_earned_eth": "1.0",
  "results": [
    {
      "id": 1,
      "asset": {
        "id": 1,
        "title": "Mystical Forest Path"
      },
      "recipient": 1,
      "amount_wei": "100000000000000000",
      "amount_eth": "0.1",
      "tx_hash": "0xabc123...",
      "status": "confirmed",
      "created_at": "2024-12-04T13:00:00Z"
    }
  ]
}
```

**Status Codes:**
- `200 OK` - Royalty payments retrieved successfully

---

### 12a. Get Royalty Payment Details

**Get details of a specific royalty payment.**

```http
GET /api/royalties/{uuid}/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "uuid": "...",
  "asset": {
    "uuid": "...",
    "title": "Mystical Forest Path"
  },
  "recipient": {
    "wallet_address": "0x...",
    "username": "creator123"
  },
  "amount_wei": "100000000000000000",
  "amount_eth": "0.1",
  "tx_hash": "0xabc123...",
  "status": "confirmed",
  "created_at": "2024-12-04T13:00:00Z"
}
```

**Status Codes:**
- `200 OK` - Royalty payment retrieved successfully
- `404 Not Found` - Payment does not exist

---

## Group IP Endpoints

Group IPs allow multiple IP assets to be pooled together for collective royalty distribution.

### 13. List Group IPs

**Get all Group IPs.**

```http
GET /api/groups/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `creator` (optional) - Filter by creator wallet address
- `is_active` (optional) - Filter by active status (true/false)
- `registration_status` (optional) - Filter by registration status (pending/registered/failed)

**Response:**
```json
{
  "count": 5,
  "results": [
    {
      "uuid": "...",
      "name": "My Collection",
      "description": "A collection of related IP assets",
      "creator": {
        "wallet_address": "0x...",
        "username": "creator123"
      },
      "total_royalty_percentage": 10,
      "member_count": 3,
      "registration_status": "registered",
      "story_group_id": "0xabc123...",
      "is_active": true,
      "created_at": "2024-12-04T10:30:00Z"
    }
  ]
}
```

**Status Codes:**
- `200 OK` - Groups retrieved successfully

---

### 14. Create Group IP

**Create a new Group IP.**

```http
POST /api/groups/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "My Collection",
  "description": "A collection of related IP assets",
  "total_royalty_percentage": 10
}
```

**Response:**
```json
{
  "uuid": "...",
  "name": "My Collection",
  "description": "A collection of related IP assets",
  "creator": {
    "wallet_address": "0x...",
    "username": "creator123"
  },
  "total_royalty_percentage": 10,
  "registration_status": "pending",
  "is_active": true,
  "created_at": "2024-12-04T10:30:00Z"
}
```

**Status Codes:**
- `201 Created` - Group created successfully
- `400 Bad Request` - Validation error

---

### 15. Get Group IP Details

**Get details of a specific Group IP.**

```http
GET /api/groups/{uuid}/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "uuid": "...",
  "name": "My Collection",
  "description": "A collection of related IP assets",
  "creator": {
    "wallet_address": "0x...",
    "username": "creator123"
  },
  "total_royalty_percentage": 10,
  "member_count": 3,
  "registration_status": "registered",
  "story_group_id": "0xabc123...",
  "royalty_pool_address": "0xdef456...",
  "members": [
    {
      "uuid": "...",
      "asset": {
        "uuid": "...",
        "title": "Asset 1"
      },
      "revenue_share_percentage": 40,
      "added_at": "2024-12-04T11:00:00Z"
    }
  ],
  "is_active": true,
  "created_at": "2024-12-04T10:30:00Z"
}
```

**Status Codes:**
- `200 OK` - Group retrieved successfully
- `404 Not Found` - Group does not exist

---

### 16. Register Group IP on Blockchain

**Register a Group IP on Story Protocol blockchain.**

```http
POST /api/groups/{uuid}/register/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "creator_address": "0x..." // Optional, defaults to authenticated user's address
}
```

**Response:**
```json
{
  "success": true,
  "message": "Group registered successfully",
  "story_group_id": "0xabc123...",
  "royalty_pool_address": "0xdef456...",
  "tx_hash": "0x789ghi..."
}
```

**Status Codes:**
- `200 OK` - Group registered successfully
- `400 Bad Request` - Group has no members or already registered
- `403 Forbidden` - Only the group creator can register it
- `404 Not Found` - Group does not exist
- `500 Internal Server Error` - Blockchain registration failed

---

### 17. Add Member to Group IP

**Add an IP asset as a member to a Group IP.**

```http
POST /api/groups/{uuid}/add_member/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "asset_uuid": "...",
  "revenue_share_percentage": 40
}
```

**Response:**
```json
{
  "success": true,
  "message": "Member added successfully",
  "membership": {
    "uuid": "...",
    "asset": {
      "uuid": "...",
      "title": "Asset 1"
    },
    "revenue_share_percentage": 40,
    "added_at": "2024-12-04T11:00:00Z"
  },
  "tx_hash": "0xabc123..."
}
```

**Status Codes:**
- `200 OK` - Member added successfully
- `400 Bad Request` - Asset already in group, invalid percentage, or total exceeds 100%
- `403 Forbidden` - Only the group creator can add members
- `404 Not Found` - Group or asset does not exist
- `500 Internal Server Error` - Blockchain transaction failed

**Validation:**
- Revenue share percentages must sum to ≤ 100% after adding
- Asset must be registered on Story Protocol
- Asset must not already be a member

---

### 18. Remove Member from Group IP

**Remove an IP asset from a Group IP.**

```http
POST /api/groups/{uuid}/remove_member/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "asset_uuid": "..."
}
```

**Response:**
```json
{
  "success": true,
  "message": "Member removed successfully",
  "tx_hash": "0xabc123..."
}
```

**Status Codes:**
- `200 OK` - Member removed successfully
- `400 Bad Request` - Asset is not a member of the group
- `403 Forbidden` - Only the group creator can remove members
- `404 Not Found` - Group or asset does not exist
- `500 Internal Server Error` - Blockchain transaction failed

---

### 19. Get Group IP Statistics

**Get statistics for a Group IP.**

```http
GET /api/groups/{uuid}/statistics/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "group_uuid": "...",
  "member_count": 3,
  "total_revenue_share": 100,
  "total_royalties_received_wei": "5000000000000000000",
  "total_royalties_received_eth": "5.0",
  "distributions_count": 12,
  "last_distribution_at": "2024-12-04T15:00:00Z"
}
```

**Status Codes:**
- `200 OK` - Statistics retrieved successfully
- `404 Not Found` - Group does not exist

---

### 20. Get Group IP Distributions

**Get royalty distributions for a Group IP.**

```http
GET /api/groups/{uuid}/distributions/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "count": 12,
  "results": [
    {
      "uuid": "...",
      "membership": {
        "asset": {
          "title": "Asset 1"
        },
        "revenue_share_percentage": 40
      },
      "amount_wei": "2000000000000000000",
      "amount_eth": "2.0",
      "tx_hash": "0xabc123...",
      "distributed_at": "2024-12-04T15:00:00Z"
    }
  ]
}
```

**Status Codes:**
- `200 OK` - Distributions retrieved successfully
- `404 Not Found` - Group does not exist

---

### 21. Update Group IP

**Update a Group IP (name, description, royalty percentage).**

```http
PUT /api/groups/{uuid}/
PATCH /api/groups/{uuid}/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "Updated Collection Name",
  "description": "Updated description",
  "total_royalty_percentage": 15
}
```

**Status Codes:**
- `200 OK` - Group updated successfully
- `400 Bad Request` - Validation error
- `403 Forbidden` - Only the group creator can update it
- `404 Not Found` - Group does not exist

---

### 22. Deactivate Group IP

**Deactivate a Group IP (soft delete).**

```http
DELETE /api/groups/{uuid}/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "success": true,
  "message": "Group deactivated successfully"
}
```

**Status Codes:**
- `200 OK` - Group deactivated successfully
- `403 Forbidden` - Only the group creator can deactivate it
- `404 Not Found` - Group does not exist

**Note:** This is a soft delete. The group is marked as inactive but not removed from the database.

---

## Dispute Endpoints

Disputes allow users to challenge IP assets for various reasons (copyright infringement, etc.).

### 23. List Disputes

**Get all disputes.**

```http
GET /api/disputes/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `target_asset` (optional) - Filter by target asset UUID
- `disputer` (optional) - Filter by disputer wallet address
- `status` (optional) - Filter by status (pending/under_review/resolved/cancelled)
- `result` (optional) - Filter by result (upheld/rejected/settled)

**Response:**
```json
{
  "count": 5,
  "results": [
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
      "reason": "Copyright infringement",
      "status": "pending",
      "result": null,
      "evidence_count": 2,
      "raised_at": "2024-12-04T10:30:00Z"
    }
  ]
}
```

**Status Codes:**
- `200 OK` - Disputes retrieved successfully

---

### 24. Raise Dispute

**Raise a new dispute against an IP asset.**

```http
POST /api/disputes/raise_dispute/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "target_asset_uuid": "...",
  "reason": "Copyright infringement - this asset uses my copyrighted material without permission",
  "evidence_description": "Initial evidence description",
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

**Status Codes:**
- `201 Created` - Dispute raised successfully
- `400 Bad Request` - Validation error or asset does not exist
- `500 Internal Server Error` - Blockchain transaction failed

---

### 25. Get Dispute Details

**Get details of a specific dispute.**

```http
GET /api/disputes/{uuid}/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "uuid": "...",
  "target_asset": {
    "uuid": "...",
    "title": "Mystical Forest Path",
    "creator": {
      "wallet_address": "0x...",
      "username": "creator123"
    }
  },
  "disputer": {
    "wallet_address": "0x...",
    "username": "disputer123"
  },
  "reason": "Copyright infringement...",
  "status": "under_review",
  "result": null,
  "resolution_notes": null,
  "resolved_by": null,
  "evidence": [
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
  ],
  "raise_transaction_hash": "0xabc123...",
  "raised_at": "2024-12-04T10:30:00Z",
  "resolved_at": null
}
```

**Status Codes:**
- `200 OK` - Dispute retrieved successfully
- `404 Not Found` - Dispute does not exist

---

### 26. Submit Evidence

**Submit evidence for a dispute (disputer or asset owner).**

```http
POST /api/disputes/{uuid}/submit_evidence/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "description": "Additional evidence description",
  "evidence_url": "ipfs://QmY6A..." // Optional
}
```

**Response:**
```json
{
  "uuid": "...",
  "dispute": "...",
  "description": "Additional evidence description",
  "evidence_url": "ipfs://QmY6A...",
  "evidence_ipfs_hash": "QmY6A...",
  "submitted_by": {
    "wallet_address": "0x...",
    "username": "disputer123"
  },
  "transaction_hash": "0xdef456...",
  "submitted_at": "2024-12-04T10:40:00Z"
}
```

**Status Codes:**
- `201 Created` - Evidence submitted successfully
- `400 Bad Request` - Validation error
- `403 Forbidden` - Only disputer or asset owner can submit evidence
- `404 Not Found` - Dispute does not exist
- `500 Internal Server Error` - Blockchain transaction failed

---

### 27. Resolve Dispute (Admin)

**Resolve a dispute (admin only).**

```http
POST /api/disputes/{uuid}/resolve/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
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

**Status Codes:**
- `200 OK` - Dispute resolved successfully
- `400 Bad Request` - Validation error or dispute already resolved
- `403 Forbidden` - Only admins can resolve disputes
- `404 Not Found` - Dispute does not exist
- `500 Internal Server Error` - Blockchain transaction failed

---

### 28. Cancel Dispute

**Cancel a dispute (disputer only).**

```http
POST /api/disputes/{uuid}/cancel/
```

**Headers:**
```
Authorization: Bearer <access_token>
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

**Status Codes:**
- `200 OK` - Dispute cancelled successfully
- `400 Bad Request` - Dispute already resolved or cancelled
- `403 Forbidden` - Only the disputer can cancel the dispute
- `404 Not Found` - Dispute does not exist

---

### 29. Get Dispute Evidence

**Get all evidence for a dispute.**

```http
GET /api/disputes/{uuid}/evidence/
```

**Headers:**
```
Authorization: Bearer <access_token>
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

**Status Codes:**
- `200 OK` - Evidence retrieved successfully
- `404 Not Found` - Dispute does not exist

---

### 30. Get Dispute Statistics

**Get dispute statistics.**

```http
GET /api/disputes/statistics/
```

**Headers:**
```
Authorization: Bearer <access_token>
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

**Status Codes:**
- `200 OK` - Statistics retrieved successfully

---

## Permission Endpoints

IP Account Permissions allow granting specific permissions to addresses for IP assets.

### 31. List Permissions

**List all permissions for user's assets.**

```http
GET /api/permissions/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `asset_uuid` (optional) - Filter by asset UUID
- `permissioned_address` (optional) - Filter by permissioned address
- `permission_type` (optional) - Filter by permission type (execute/transfer_erc20/set_metadata/attach_license/register_derivative/collect_royalty)
- `is_active` (optional) - Filter by active status (true/false)

**Response:**
```json
{
  "count": 10,
  "results": [
    {
      "uuid": "...",
      "asset": {
        "uuid": "...",
        "title": "Mystical Forest Path"
      },
      "permissioned_address": "0xabc123...",
      "permission_type": "execute",
      "is_granted": true,
      "expires_at": null,
      "granted_by": {
        "wallet_address": "0x...",
        "username": "creator123"
      },
      "created_at": "2024-12-04T10:30:00Z"
    }
  ]
}
```

**Status Codes:**
- `200 OK` - Permissions retrieved successfully

---

### 32. Get Permission Details

**Get details of a specific permission.**

```http
GET /api/permissions/{uuid}/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "uuid": "...",
  "asset": {
    "uuid": "...",
    "title": "Mystical Forest Path",
    "story_ip_id": "0x..."
  },
  "permissioned_address": "0xabc123...",
  "permission_type": "execute",
  "is_granted": true,
  "expires_at": null,
  "granted_by": {
    "wallet_address": "0x...",
    "username": "creator123"
  },
  "transaction_hash": "0xdef456...",
  "created_at": "2024-12-04T10:30:00Z"
}
```

**Status Codes:**
- `200 OK` - Permission retrieved successfully
- `404 Not Found` - Permission does not exist

---

### 33. Set Permission

**Set a single permission for an asset.**

```http
POST /api/permissions/set_permission/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "asset_uuid": "...",
  "permissioned_address": "0xabc123...",
  "permission_type": "execute",
  "expires_at": "2025-12-31T23:59:59Z" // optional - null for no expiration
}
```

**Response:**
```json
{
  "success": true,
  "message": "Permission set successfully",
  "permission": {
    "uuid": "...",
    "asset": {
      "uuid": "...",
      "title": "Mystical Forest Path"
    },
    "permissioned_address": "0xabc123...",
    "permission_type": "execute",
    "is_granted": true,
    "expires_at": "2025-12-31T23:59:59Z",
    "created_at": "2024-12-04T10:30:00Z"
  },
  "tx_hash": "0xdef456..."
}
```

**Status Codes:**
- `200 OK` - Permission set successfully
- `400 Bad Request` - Validation error
- `403 Forbidden` - Only the asset creator can set permissions
- `404 Not Found` - Asset does not exist
- `500 Internal Server Error` - Blockchain transaction failed

**Permission Types:**
- `execute` - Can execute transactions on behalf of the IP
- `transfer_erc20` - Can transfer ERC20 tokens held by the IP account
- `set_metadata` - Can update IP metadata
- `attach_license` - Can attach license terms
- `register_derivative` - Can register derivatives
- `collect_royalty` - Can collect royalties

---

### 34. Set All Permissions

**Set all permissions for an asset to the same state.**

```http
POST /api/permissions/set_all_permissions/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "asset_uuid": "...",
  "permissioned_address": "0xabc123...",
  "permissions": {
    "execute": true,
    "transfer_erc20": false,
    "set_metadata": false,
    "attach_license": false,
    "register_derivative": true,
    "collect_royalty": false
  }
}
```

**Response:**
```json
{
  "success": true,
  "message": "All permissions set successfully",
  "permissions": [
    {
      "permission_type": "execute",
      "is_granted": true
    },
    {
      "permission_type": "transfer_erc20",
      "is_granted": false
    },
    {
      "permission_type": "set_metadata",
      "is_granted": false
    },
    {
      "permission_type": "attach_license",
      "is_granted": false
    },
    {
      "permission_type": "register_derivative",
      "is_granted": true
    },
    {
      "permission_type": "collect_royalty",
      "is_granted": false
    }
  ],
  "tx_hash": "0xdef456..."
}
```

**Status Codes:**
- `200 OK` - Permissions set successfully
- `400 Bad Request` - Validation error
- `403 Forbidden` - Only the asset creator can set permissions
- `404 Not Found` - Asset does not exist
- `500 Internal Server Error` - Blockchain transaction failed

---

### 35. Revoke Permission

**Revoke a specific permission.**

```http
POST /api/permissions/revoke_permission/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "asset_uuid": "...",
  "permissioned_address": "0xabc123...",
  "permission_type": "execute"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Permission revoked successfully",
  "permission": {
    "uuid": "...",
    "is_granted": false,
    "updated_at": "2024-12-04T15:00:00Z"
  },
  "tx_hash": "0xdef456..."
}
```

**Status Codes:**
- `200 OK` - Permission revoked successfully
- `400 Bad Request` - Permission not found or already revoked
- `403 Forbidden` - Only the asset creator can revoke permissions
- `404 Not Found` - Asset does not exist
- `500 Internal Server Error` - Blockchain transaction failed

---

### 36. Revoke All Permissions

**Revoke all permissions for an address on an asset.**

```http
POST /api/permissions/revoke_all_permissions/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "asset_uuid": "...",
  "permissioned_address": "0xabc123..."
}
```

**Response:**
```json
{
  "success": true,
  "message": "All permissions revoked successfully",
  "revoked_count": 6,
  "tx_hash": "0xdef456..."
}
```

**Status Codes:**
- `200 OK` - Permissions revoked successfully
- `400 Bad Request` - No permissions found
- `403 Forbidden` - Only the asset creator can revoke permissions
- `404 Not Found` - Asset does not exist
- `500 Internal Server Error` - Blockchain transaction failed

---

### 37. Get Permissions for Asset

**Get all permissions for a specific asset.**

```http
GET /api/permissions/for_asset/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `asset_uuid` (required) - Asset UUID

**Response:**
```json
{
  "count": 5,
  "results": [
    {
      "uuid": "...",
      "permissioned_address": "0xabc123...",
      "permission_type": "execute",
      "is_granted": true,
      "expires_at": null,
      "created_at": "2024-12-04T10:30:00Z"
    }
  ]
}
```

**Status Codes:**
- `200 OK` - Permissions retrieved successfully
- `400 Bad Request` - asset_uuid is required

---

### 38. Check Permission

**Check if an address has a specific permission.**

```http
GET /api/permissions/check_permission/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `asset_uuid` (required) - Asset UUID
- `permissioned_address` (required) - Address to check
- `permission_type` (required) - Permission type to check

**Response:**
```json
{
  "has_permission": true,
  "permission": {
    "uuid": "...",
    "permission_type": "execute",
    "is_granted": true,
    "expires_at": null,
    "created_at": "2024-12-04T10:30:00Z"
  }
}
```

**Status Codes:**
- `200 OK` - Permission check completed
- `400 Bad Request` - Missing required parameters

---

### 39. Get Permission Summary

**Get a summary of all permissions for an address on an asset.**

```http
GET /api/permissions/permission_summary/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `asset_uuid` (required) - Asset UUID
- `permissioned_address` (required) - Address to check

**Response:**
```json
{
  "asset_uuid": "...",
  "permissioned_address": "0xabc123...",
  "permissions": {
    "execute": true,
    "transfer_erc20": false,
    "set_metadata": false,
    "attach_license": false,
    "register_derivative": true,
    "collect_royalty": false
  },
  "active_count": 2,
  "total_count": 6
}
```

**Status Codes:**
- `200 OK` - Summary retrieved successfully
- `400 Bad Request` - Missing required parameters

---

## Collections Endpoints

### 20. List Collections

**Get all collections created by the authenticated user.**

```http
GET /api/assets/collections/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `creator` (optional) - Filter by creator wallet address
- `is_public` (optional) - Filter by public/private (true/false)

**Response:**
```json
{
  "count": 5,
  "results": [
    {
      "id": 1,
      "creator": {
        "id": 1,
        "wallet_address": "0x742d35...",
        "username": "creator123"
      },
      "name": "My Digital Art Collection",
      "description": "A collection of my favorite digital artworks",
      "is_public": true,
      "asset_count": 12,
      "created_at": "2024-12-04T10:30:00Z"
    }
  ]
}
```

**Status Codes:**
- `200 OK` - Collections retrieved successfully

---

### 21. Create Collection

**Create a new collection.**

```http
POST /api/assets/collections/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "My Digital Art Collection",
  "description": "A collection of my favorite digital artworks",
  "is_public": true
}
```

**Response:**
```json
{
  "id": 1,
  "creator": 1,
  "name": "My Digital Art Collection",
  "description": "A collection of my favorite digital artworks",
  "is_public": true,
  "created_at": "2024-12-04T10:30:00Z"
}
```

**Status Codes:**
- `201 Created` - Collection created successfully
- `400 Bad Request` - Validation error

---

### 22. Add Asset to Collection

**Add an asset to a collection.**

```http
POST /api/assets/collections/{collection_id}/assets/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "asset_id": 5
}
```

**Response:**
```json
{
  "id": 1,
  "collection": 1,
  "asset": {
    "id": 5,
    "title": "Mystical Forest Path"
  },
  "added_at": "2024-12-04T11:00:00Z"
}
```

**Status Codes:**
- `201 Created` - Asset added successfully
- `400 Bad Request` - Asset already in collection or invalid

---

## Favorites Endpoints

### 23. List Favorites

**Get all assets favorited by the authenticated user.**

```http
GET /api/assets/favorites/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "count": 8,
  "results": [
    {
      "id": 1,
      "asset": {
        "id": 5,
        "title": "Mystical Forest Path",
        "media_url": "ipfs://...",
        "creator": {
          "username": "artist123"
        }
      },
      "created_at": "2024-12-04T10:30:00Z"
    }
  ]
}
```

**Status Codes:**
- `200 OK` - Favorites retrieved successfully

---

### 24. Favorite an Asset

**Add an asset to favorites.**

```http
POST /api/assets/favorites/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "asset_id": 5
}
```

**Response:**
```json
{
  "id": 1,
  "user": 1,
  "asset": 5,
  "created_at": "2024-12-04T10:30:00Z"
}
```

**Status Codes:**
- `201 Created` - Asset favorited successfully
- `400 Bad Request` - Asset already favorited

---

### 25. Unfavorite an Asset

**Remove an asset from favorites.**

```http
DELETE /api/assets/favorites/{id}/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "message": "Asset removed from favorites"
}
```

**Status Codes:**
- `204 No Content` - Successfully unfavorited

---

## Comments Endpoints

### 26. List Comments

**Get all comments for an asset.**

```http
GET /api/comments/?asset_id={asset_id}
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "count": 12,
  "results": [
    {
      "id": 1,
      "asset": 5,
      "user": {
        "id": 2,
        "username": "commenter123",
        "avatar_url": "ipfs://..."
      },
      "content": "This is amazing!",
      "likes_count": 5,
      "replies": [
        {
          "id": 2,
          "user": {
            "id": 1,
            "username": "creator123"
          },
          "content": "Thank you!",
          "likes_count": 2,
          "created_at": "2024-12-04T11:00:00Z"
        }
      ],
      "created_at": "2024-12-04T10:30:00Z"
    }
  ]
}
```

**Status Codes:**
- `200 OK` - Comments retrieved successfully

---

### 27. Create Comment

**Create a new comment on an asset.**

```http
POST /api/comments/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "asset_id": 5,
  "content": "This is amazing!",
  "parent_id": null
}
```

**Response:**
```json
{
  "id": 1,
  "asset": 5,
  "user": 1,
  "content": "This is amazing!",
  "likes_count": 0,
  "parent": null,
  "created_at": "2024-12-04T10:30:00Z"
}
```

**Status Codes:**
- `201 Created` - Comment created successfully
- `400 Bad Request` - Validation error

---

### 28. Like/Unlike Comment

**Like or unlike a comment.**

```http
POST /api/comments/{id}/like/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "liked": true,
  "likes_count": 6
}
```

**Status Codes:**
- `200 OK` - Like status updated

---

## User Profile Endpoints

### 29. Update Profile

**Update user profile information.**

```http
POST /api/auth/profile/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "username": "newusername",
  "bio": "Updated bio text"
}
```

**Response:**
```json
{
  "id": 1,
  "wallet_address": "0x742d35...",
  "username": "newusername",
  "bio": "Updated bio text",
  "avatar_url": "ipfs://...",
  "banner_url": "ipfs://..."
}
```

**Status Codes:**
- `200 OK` - Profile updated successfully

---

### 30. Upload Avatar

**Upload and crop avatar image.**

```http
POST /api/auth/upload-avatar/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: multipart/form-data
```

**Request Body:**
```
file: <binary image data>
cropData: {"x": 0, "y": 0, "width": 200, "height": 200}
```

**Response:**
```json
{
  "avatar_url": "ipfs://QmX5Z..."
}
```

**Status Codes:**
- `200 OK` - Avatar uploaded successfully
- `400 Bad Request` - Invalid file or crop data

---

### 31. Upload Banner

**Upload and crop banner image.**

```http
POST /api/auth/upload-banner/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: multipart/form-data
```

**Request Body:**
```
file: <binary image data>
cropData: {"x": 0, "y": 0, "width": 1200, "height": 400}
```

**Response:**
```json
{
  "banner_url": "ipfs://QmY6A..."
}
```

**Status Codes:**
- `200 OK` - Banner uploaded successfully
- `400 Bad Request` - Invalid file or crop data

---

## AI Generation Endpoints

### 13. Generate Title Suggestions

**Generate 4 AI-powered title suggestions based on description.**

```http
POST /api/assets/ai/generate-title/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "description": "a mystical forest with glowing mushrooms and ethereal mist",
  "asset_type": "digital_art"
}
```

**Response:**
```json
{
  "titles": [
    "Mystical Forest Path",
    "Enchanted Woodland Trail",
    "Ethereal Mushroom Glade",
    "Luminescent Forest Sanctuary"
  ],
  "model_used": "google/gemini-flash-1.5",
  "log_id": 123
}
```

**Status Codes:**
- `200 OK` - Titles generated successfully
- `400 Bad Request` - Missing description
- `500 Internal Server Error` - AI generation failed

**Performance:**
- **Uncached:** 2-5 seconds
- **Cached:** <50ms (1-hour cache TTL)

---

### 14. Enhance Description

**Enhance a short description into a detailed, engaging version (100-150 words).**

```http
POST /api/assets/ai/enhance-description/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "description": "mystical forest",
  "title": "Mystical Forest Path",
  "asset_type": "digital_art"
}
```

**Response:**
```json
{
  "enhanced_description": "A hauntingly beautiful digital artwork depicting a winding path through an enchanted forest. The scene is illuminated by bioluminescent mushrooms that cast an ethereal glow across the misty forest floor. Ancient trees tower overhead, their gnarled branches creating a natural canopy that filters soft moonlight. Wisps of magical fog dance between the trunks, creating an atmosphere of mystery and wonder. The path itself seems to beckon viewers deeper into this mystical realm, promising secrets and adventures yet to be discovered. Perfect for fantasy enthusiasts and lovers of digital art that transports viewers to otherworldly landscapes.",
  "model_used": "google/gemini-flash-1.5",
  "log_id": 124
}
```

**Status Codes:**
- `200 OK` - Description enhanced successfully
- `400 Bad Request` - Missing description
- `500 Internal Server Error` - AI generation failed

---

### 15. Analyze Content

**Extract metadata (category, tags, style, theme, genre) from title and description.**

```http
POST /api/assets/ai/analyze-content/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "title": "Mystical Forest Path",
  "description": "A hauntingly beautiful digital artwork depicting a winding path...",
  "media_url": "ipfs://QmX5Z..."
}
```

**Response:**
```json
{
  "category": "digital_art",
  "tags": ["fantasy", "forest", "nature", "mystical", "ethereal", "bioluminescent"],
  "art_style": "digital painting",
  "theme": "nature and magic",
  "genre": "fantasy",
  "model_used": "google/gemini-flash-1.5",
  "log_id": 125
}
```

**Status Codes:**
- `200 OK` - Content analyzed successfully
- `400 Bad Request` - Missing title or description
- `500 Internal Server Error` - AI analysis failed

---

### 16. Suggest License Terms

**Recommend optimal license terms based on asset type and intended use.**

```http
POST /api/assets/ai/suggest-license/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "asset_type": "digital_art",
  "description": "A hauntingly beautiful digital artwork...",
  "intended_use": "Allow remixes and commercial use"
}
```

**Response:**
```json
{
  "license_type": "CC BY-SA 4.0",
  "commercial_use": true,
  "derivative_allowed": true,
  "attribution_required": true,
  "share_alike": true,
  "royalty_percentage": 10,
  "explanation": "This Creative Commons license allows both commercial use and remixes while requiring attribution and that derivatives use the same license. A 10% royalty ensures you benefit from commercial adaptations.",
  "model_used": "google/gemini-flash-1.5",
  "log_id": 126
}
```

**Status Codes:**
- `200 OK` - License suggested successfully
- `400 Bad Request` - Missing asset_type or description
- `500 Internal Server Error` - AI suggestion failed

---

### 17. Analyze Derivative Similarity

**Analyze similarity between parent and child assets for derivative validation.**

```http
POST /api/assets/ai/analyze-derivative/
```

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "parent_description": "A mystical forest with glowing mushrooms and ethereal mist",
  "child_description": "A cyberpunk-inspired remix featuring neon colors and futuristic elements",
  "parent_media_url": "ipfs://QmX5Z...",
  "child_media_url": "ipfs://QmY6A..."
}
```

**Response:**
```json
{
  "similarity_score": 75,
  "similar_elements": [
    "forest setting",
    "atmospheric mood",
    "glowing light sources"
  ],
  "differences": [
    "color palette (mystical vs neon)",
    "art style (fantasy vs cyberpunk)",
    "time period (timeless vs futuristic)"
  ],
  "is_derivative": true,
  "suggested_royalty_percentage": 10,
  "explanation": "The child asset clearly derives from the parent's forest concept and atmospheric lighting, but transforms it into a cyberpunk aesthetic. This qualifies as a transformative derivative work.",
  "model_used": "google/gemini-flash-1.5",
  "log_id": 127
}
```

**Status Codes:**
- `200 OK` - Derivative analyzed successfully
- `400 Bad Request` - Missing descriptions
- `500 Internal Server Error` - AI analysis failed

---

## Analytics Endpoints

### 18. Get User AI Usage Stats

**Get AI usage statistics for the authenticated user.**

```http
GET /api/assets/ai/usage-stats/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `start_date` (optional) - Filter from date (YYYY-MM-DD)
- `end_date` (optional) - Filter to date (YYYY-MM-DD)

**Response:**
```json
{
  "total_requests": 47,
  "successful_requests": 45,
  "failed_requests": 2,
  "cache_hits": 12,
  "cache_hit_rate": 25.53,
  "avg_response_time_ms": 2847,
  "total_tokens_used": 23500,
  "requests_by_operation": {
    "title": 15,
    "description": 18,
    "analysis": 7,
    "license": 5,
    "derivative": 2
  },
  "requests_by_model": {
    "google/gemini-flash-1.5": 40,
    "meta-llama/llama-3.2-3b-instruct": 5,
    "mistralai/mistral-7b-instruct": 2
  },
  "recent_requests": [
    {
      "id": 127,
      "operation_type": "derivative",
      "status": "success",
      "model_used": "google/gemini-flash-1.5",
      "response_time_ms": 3245,
      "tokens_used": 487,
      "cache_hit": false,
      "created_at": "2024-12-04T14:30:00Z"
    }
  ]
}
```

**Status Codes:**
- `200 OK` - Stats retrieved successfully

---

### 19. Get Platform AI Stats

**Get platform-wide AI usage statistics (admin only).**

```http
GET /api/assets/ai/platform-stats/
```

**Headers:**
```
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `days` (optional) - Last N days (default: 30)

**Response:**
```json
{
  "total_users": 127,
  "total_requests": 5432,
  "successful_requests": 5387,
  "failed_requests": 45,
  "cache_hits": 5163,
  "cache_hit_rate": 95.05,
  "avg_response_time_ms": 891,
  "total_tokens_used": 2710500,
  "requests_by_operation": {
    "title": 1821,
    "description": 2034,
    "analysis": 892,
    "license": 541,
    "derivative": 144
  },
  "requests_by_model": {
    "google/gemini-flash-1.5": 5012,
    "meta-llama/llama-3.2-3b-instruct": 312,
    "mistralai/mistral-7b-instruct": 108
  },
  "daily_stats": [
    {
      "date": "2024-12-04",
      "total_requests": 234,
      "cache_hits": 221,
      "unique_users": 45
    }
  ]
}
```

**Status Codes:**
- `200 OK` - Stats retrieved successfully
- `403 Forbidden` - User is not admin

---

## Error Responses

### Standard Error Format

All error responses follow this format:

```json
{
  "error": "Brief error message",
  "detail": "Detailed explanation of what went wrong",
  "field_errors": {
    "field_name": ["Validation error for this field"]
  }
}
```

### Common HTTP Status Codes

| Code | Meaning | Description |
|------|---------|-------------|
| 200 | OK | Request succeeded |
| 201 | Created | Resource created successfully |
| 204 | No Content | Resource deleted successfully |
| 400 | Bad Request | Invalid request data |
| 401 | Unauthorized | Missing or invalid auth token |
| 403 | Forbidden | Authenticated but not authorized |
| 404 | Not Found | Resource does not exist |
| 422 | Unprocessable Entity | Validation failed |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server-side error |

### Example Error Responses

**401 Unauthorized:**
```json
{
  "detail": "Authentication credentials were not provided."
}
```

**400 Bad Request:**
```json
{
  "error": "Validation failed",
  "field_errors": {
    "description": ["This field is required."],
    "asset_type": ["Invalid choice. Must be one of: digital_art, music, writing, photography, game_asset."]
  }
}
```

**500 Internal Server Error:**
```json
{
  "error": "Failed to register IP asset",
  "detail": "Story Protocol blockchain transaction failed: insufficient gas"
}
```

---

## Rate Limiting

### Default Limits

| Endpoint Group | Rate Limit |
|----------------|------------|
| Authentication | 10 requests / minute |
| IP Assets (Read) | 100 requests / minute |
| IP Assets (Write) | 20 requests / minute |
| AI Generation | 60 requests / minute |
| Analytics | 30 requests / minute |

### Rate Limit Headers

Every response includes rate limit headers:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1701691200
```

### Rate Limit Exceeded Response

```http
HTTP/1.1 429 Too Many Requests
```

```json
{
  "error": "Rate limit exceeded",
  "detail": "You have exceeded the rate limit of 100 requests per minute. Please try again in 23 seconds.",
  "retry_after": 23
}
```

---

## Authentication Flow

### Complete SIWE Authentication Example

**Step 1: Generate Nonce**
```http
POST /api/auth/generate-nonce/
```

Response:
```json
{
  "nonce": "abc123",
  "message": "localhost:3000 wants you to sign in..."
}
```

**Step 2: Sign Message with Wallet**
```javascript
// Frontend (using Wagmi)
const { signMessageAsync } = useSignMessage();
const signature = await signMessageAsync({ message });
```

**Step 3: Verify Signature**
```http
POST /api/auth/verify-signature/

{
  "message": "localhost:3000 wants you to sign in...",
  "signature": "0xabc123...",
  "wallet_address": "0x742d35..."
}
```

Response:
```json
{
  "access": "eyJhbGc...",
  "refresh": "eyJhbGc...",
  "user": { "id": 1, "wallet_address": "0x742d35..." }
}
```

**Step 4: Use Access Token**
```http
GET /api/assets/
Authorization: Bearer eyJhbGc...
```

**Step 5: Refresh Token (when access expires)**
```http
POST /api/auth/refresh-token/

{
  "refresh": "eyJhbGc..."
}
```

---

## Complete Asset Creation Flow

### End-to-End Example

**Step 1: Upload Media to IPFS**
```http
POST /api/assets/upload-to-ipfs/
Authorization: Bearer <token>
Content-Type: multipart/form-data

file: <binary image data>
```

Response:
```json
{
  "media_url": "ipfs://QmX5Z..."
}
```

**Step 2: Generate AI Title (Optional)**
```http
POST /api/assets/ai/generate-title/
Authorization: Bearer <token>

{
  "description": "mystical forest",
  "asset_type": "digital_art"
}
```

Response:
```json
{
  "titles": ["Mystical Forest Path", "Enchanted Woodland", ...]
}
```

**Step 3: Enhance Description with AI (Optional)**
```http
POST /api/assets/ai/enhance-description/
Authorization: Bearer <token>

{
  "description": "mystical forest",
  "title": "Mystical Forest Path"
}
```

Response:
```json
{
  "enhanced_description": "A hauntingly beautiful digital artwork..."
}
```

**Step 4: Register IP Asset on Blockchain**
```http
POST /api/assets/register-ip/
Authorization: Bearer <token>

{
  "title": "Mystical Forest Path",
  "description": "A hauntingly beautiful digital artwork...",
  "asset_type": "digital_art",
  "media_url": "ipfs://QmX5Z..."
}
```

Response:
```json
{
  "asset": {
    "id": 1,
    "ip_id": "0x123abc...",
    "token_id": 1,
    ...
  },
  "tx_hash": "0xdef456..."
}
```

---

## SDK / Client Libraries

### TypeScript/JavaScript Client (Frontend)

```typescript
// lib/api.ts
import axios from 'axios';

const api = axios.create({
  baseURL: 'http://localhost:8000',
});

// Add auth token
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('access_token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// API functions
export const aiAPI = {
  generateTitle: (data) => api.post('/api/assets/ai/generate-title/', data),
  enhanceDescription: (data) => api.post('/api/assets/ai/enhance-description/', data),
  // ...
};
```

### React Hooks

```typescript
// hooks/useAI.ts
import { useMutation } from '@tanstack/react-query';
import { aiAPI } from '@/lib/api';

export function useGenerateTitle() {
  return useMutation({
    mutationFn: aiAPI.generateTitle,
  });
}

// Usage
const generateTitle = useGenerateTitle();
const result = await generateTitle.mutateAsync({
  description: 'mystical forest',
  asset_type: 'digital_art'
});
```

---

## Testing with cURL

### Generate Title
```bash
curl -X POST http://localhost:8000/api/assets/ai/generate-title/ \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "description": "mystical forest with glowing mushrooms",
    "asset_type": "digital_art"
  }'
```

### Upload File
```bash
curl -X POST http://localhost:8000/api/assets/upload-to-ipfs/ \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "file=@/path/to/image.png"
```

### List Assets
```bash
curl -X GET "http://localhost:8000/api/assets/?asset_type=digital_art&page=1" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

---

## API Changelog

### v1.2.0 (January 2025)
- ✅ Group IP endpoints (create, register, manage members, distributions)
- ✅ Dispute endpoints (raise, submit evidence, resolve, cancel)
- ✅ Permission endpoints (set, revoke, check permissions)
- ✅ Multi-parent derivative creation (1-10 parents)
- ✅ Asset restore and permanent delete endpoints
- ✅ Asset retry registration and creation endpoints
- ✅ Asset derivatives listing endpoint
- ✅ Asset royalty balance and claim endpoints
- ✅ Royalty payment details endpoint
- ✅ Enhanced asset creation workflow with step tracking

### v1.1.0 (December 2024)
- ✅ Collections endpoints (CRUD operations)
- ✅ Favorites endpoints (add/remove favorites)
- ✅ Comments endpoints (create, update, delete, like, reply)
- ✅ User profile endpoints (update profile, upload avatar/banner)
- ✅ Asset update and delete endpoints
- ✅ Enhanced filtering and search capabilities

### v1.0.0 (December 2024)
- ✅ Initial release
- ✅ Authentication endpoints (SIWE)
- ✅ IP asset CRUD operations
- ✅ Derivative asset creation
- ✅ 5 AI generation endpoints
- ✅ 2 analytics endpoints
- ✅ Rate limiting
- ✅ JWT authentication

---

**Document Version:** 3.0
**Last Updated:** January 2025
**API Version:** v1.2.0
**Next:** [07-DEPLOYMENT-GUIDE.md](./07-DEPLOYMENT-GUIDE.md)

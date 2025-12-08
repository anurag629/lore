# Group IP Guide

## Overview

Group IPs allow multiple IP assets to be pooled together for collective royalty distribution. This feature enables creators to form collections or groups where royalties are automatically distributed among members based on their revenue share percentages.

---

## Table of Contents

1. [What is a Group IP?](#what-is-a-group-ip)
2. [Use Cases](#use-cases)
3. [Creating a Group IP](#creating-a-group-ip)
4. [Managing Members](#managing-members)
5. [Royalty Distribution](#royalty-distribution)
6. [API Reference](#api-reference)
7. [Examples](#examples)

---

## What is a Group IP?

A **Group IP** is a Story Protocol feature that allows multiple IP assets to be grouped together. When royalties are earned from derivatives of any member asset, they are automatically distributed to all group members according to their configured revenue share percentages.

### Key Concepts

- **Group IP**: A collection of IP assets that share royalties
- **Members**: IP assets that belong to the group
- **Revenue Share Percentage**: The percentage of group royalties allocated to each member
- **Royalty Pool**: On-chain contract that manages royalty distribution
- **Total Royalty Percentage**: The overall royalty percentage for the group (applied to derivatives)

---

## Use Cases

### 1. Artist Collections
An artist creates a collection of related artworks. When someone creates a derivative of any artwork in the collection, royalties are shared among all artworks.

**Example:**
- Group: "Fantasy Series"
- Members: 5 fantasy artworks
- Each member gets 20% of royalties

### 2. Collaborative Projects
Multiple creators collaborate on a project. All assets are grouped together, and royalties are distributed based on contribution.

**Example:**
- Group: "Collaborative Album"
- Members: 3 music tracks
- Revenue shares: 40%, 35%, 25%

### 3. Franchise Management
A creator manages a franchise with multiple related IP assets. Group IP ensures consistent royalty distribution across the franchise.

---

## Creating a Group IP

### Step 1: Create the Group

```http
POST /api/groups/
Authorization: Bearer <token>
Content-Type: application/json

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
  "total_royalty_percentage": 10,
  "registration_status": "pending",
  "is_active": true
}
```

### Step 2: Add Members

Add IP assets to the group with their revenue share percentages:

```http
POST /api/groups/{uuid}/add_member/
Authorization: Bearer <token>
Content-Type: application/json

{
  "asset_uuid": "asset-uuid-1",
  "revenue_share_percentage": 40
}
```

**Important:** Revenue share percentages must sum to ≤ 100%.

### Step 3: Register on Blockchain

Once you've added all members, register the group on Story Protocol:

```http
POST /api/groups/{uuid}/register/
Authorization: Bearer <token>
Content-Type: application/json

{
  "creator_address": "0x..." // Optional
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

---

## Managing Members

### Adding Members

Members can be added before or after blockchain registration:

```http
POST /api/groups/{uuid}/add_member/
{
  "asset_uuid": "...",
  "revenue_share_percentage": 30
}
```

**Requirements:**
- Asset must be registered on Story Protocol
- Asset must not already be a member
- Total revenue share must not exceed 100% after adding

### Removing Members

```http
POST /api/groups/{uuid}/remove_member/
{
  "asset_uuid": "..."
}
```

**Note:** Removing members requires a blockchain transaction.

### Viewing Members

```http
GET /api/groups/{uuid}/
```

The response includes all members with their revenue share percentages.

---

## Royalty Distribution

### How It Works

1. **Derivative Created**: Someone creates a derivative of a group member asset
2. **Royalty Triggered**: The derivative pays royalties to the group
3. **Automatic Distribution**: Royalties are distributed to all members based on their revenue share percentages
4. **On-Chain**: All distributions are recorded on the blockchain

### Viewing Distributions

```http
GET /api/groups/{uuid}/distributions/
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

### Statistics

Get group statistics:

```http
GET /api/groups/{uuid}/statistics/
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

---

## API Reference

### Endpoints Summary

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/groups/` | List all groups |
| POST | `/api/groups/` | Create new group |
| GET | `/api/groups/{uuid}/` | Get group details |
| PUT/PATCH | `/api/groups/{uuid}/` | Update group |
| DELETE | `/api/groups/{uuid}/` | Deactivate group |
| POST | `/api/groups/{uuid}/register/` | Register on blockchain |
| POST | `/api/groups/{uuid}/add_member/` | Add member |
| POST | `/api/groups/{uuid}/remove_member/` | Remove member |
| GET | `/api/groups/{uuid}/statistics/` | Get statistics |
| GET | `/api/groups/{uuid}/distributions/` | Get distributions |

### Models

#### GroupIP
- `uuid` - Public UUID
- `story_group_id` - Story Protocol Group IP ID (on-chain)
- `name` - Group name
- `description` - Group description
- `creator` - User who created the group
- `royalty_pool_address` - On-chain royalty pool contract address
- `total_royalty_percentage` - Total royalty percentage (0-100)
- `registration_status` - pending/registered/failed
- `is_active` - Active status

#### GroupIPMembership
- `uuid` - Public UUID
- `group` - Foreign key to GroupIP
- `asset` - Foreign key to IPAsset
- `revenue_share_percentage` - Revenue share (0-100)
- `is_active` - Active status
- `added_at` - When member was added

---

## Examples

### Example 1: Simple Collection

Create a collection of 3 artworks with equal revenue sharing:

```python
# 1. Create group
group = create_group(
    name="Fantasy Art Collection",
    description="A collection of fantasy artworks",
    total_royalty_percentage=10
)

# 2. Add members (equal shares)
add_member(group, asset1, revenue_share=33.33)
add_member(group, asset2, revenue_share=33.33)
add_member(group, asset3, revenue_share=33.34)

# 3. Register on blockchain
register_group(group)
```

### Example 2: Collaborative Project

Create a collaborative project with unequal revenue sharing:

```python
# 1. Create group
group = create_group(
    name="Collaborative Album",
    description="A collaborative music project",
    total_royalty_percentage=15
)

# 2. Add members (based on contribution)
add_member(group, track1, revenue_share=40)  # Main contributor
add_member(group, track2, revenue_share=35)  # Secondary contributor
add_member(group, track3, revenue_share=25)  # Supporting contributor

# 3. Register on blockchain
register_group(group)
```

### Example 3: Frontend Integration

```typescript
// Create group
const createGroup = async (name: string, description: string) => {
  const response = await api.post('/api/groups/', {
    name,
    description,
    total_royalty_percentage: 10
  });
  return response.data;
};

// Add member
const addMember = async (groupUuid: string, assetUuid: string, share: number) => {
  const response = await api.post(`/api/groups/${groupUuid}/add_member/`, {
    asset_uuid: assetUuid,
    revenue_share_percentage: share
  });
  return response.data;
};

// Register group
const registerGroup = async (groupUuid: string) => {
  const response = await api.post(`/api/groups/${groupUuid}/register/`);
  return response.data;
};
```

---

## Best Practices

1. **Revenue Share Planning**: Plan revenue shares before adding members to ensure they sum to 100%
2. **Member Selection**: Only add assets that are registered on Story Protocol
3. **Group Size**: Consider the number of members - more members mean smaller individual shares
4. **Royalty Percentage**: Set an appropriate total royalty percentage (typically 5-20%)
5. **Documentation**: Provide clear descriptions for your groups

---

## Limitations

- Maximum revenue share: 100% total across all members
- Members must be registered IP assets
- Group registration requires at least one member
- Removing members requires blockchain transactions
- Revenue shares cannot be changed after registration (new members can be added)

---

## Troubleshooting

### "Revenue share percentages exceed 100%"
**Solution:** Reduce individual percentages or remove members until total ≤ 100%

### "Asset is not registered on Story Protocol"
**Solution:** Register the asset first before adding to group

### "Group has no members"
**Solution:** Add at least one member before registering

### "Blockchain registration failed"
**Solution:** Check gas balance, network connection, and try again

---

**Last Updated:** January 2025  
**Related:** [API Documentation](./06-API-DOCUMENTATION.md) | [Solution Architecture](./03-SOLUTION-ARCHITECTURE.md)


# IP Account Permissions Guide

## Overview

IP Account Permissions allow asset creators to grant specific permissions to other addresses for their IP assets. This enables collaboration, delegation, and flexible IP management on Story Protocol.

---

## Table of Contents

1. [What are IP Account Permissions?](#what-are-ip-account-permissions)
2. [Permission Types](#permission-types)
3. [Granting Permissions](#granting-permissions)
4. [Revoking Permissions](#revoking-permissions)
5. [Checking Permissions](#checking-permissions)
6. [API Reference](#api-reference)
7. [Use Cases](#use-cases)
8. [Examples](#examples)

---

## What are IP Account Permissions?

**IP Account Permissions** are on-chain permissions that allow specific addresses to perform certain actions on behalf of an IP asset. This is part of Story Protocol's IP Account system, which enables decentralized IP management.

### Key Concepts

- **Permissioned Address**: The Ethereum address that receives the permission
- **Permission Type**: The specific action allowed (execute, register_derivative, collect_royalty, etc.)
- **Asset Creator**: The original creator who can grant/revoke permissions
- **On-Chain**: All permissions are stored on the blockchain via Story Protocol

---

## Permission Types

### 1. Execute (`execute`)

Allows the address to **execute transactions** on behalf of the IP asset.

**Use Cases:**
- Delegate transaction authority to a manager or agent
- Enable automated systems to execute transactions
- Allow collaborators to act on your behalf

### 2. Transfer ERC20 (`transfer_erc20`)

Allows the address to **transfer ERC20 tokens** held by the IP account.

**Use Cases:**
- Enable managers to move tokens for royalty distribution
- Allow automated systems to transfer earned royalties
- Delegate token management to a trusted party

### 3. Set Metadata (`set_metadata`)

Allows the address to **update metadata** for the IP asset.

**Use Cases:**
- Allow collaborators to update asset information
- Enable managers to maintain asset metadata
- Delegate content updates to team members

### 4. Attach License (`attach_license`)

Allows the address to **attach license terms** to the IP asset.

**Use Cases:**
- Allow partners to set licensing terms
- Enable automated licensing systems
- Delegate license management to legal teams

### 5. Register Derivative (`register_derivative`)

Allows the address to **register derivatives** of the IP asset.

**Use Cases:**
- Allow collaborators to create official derivatives
- Enable community members to create authorized remixes
- Delegate derivative creation to a partner

### 6. Collect Royalty (`collect_royalty`)

Allows the address to **collect royalties** earned by the IP asset.

**Use Cases:**
- Allow managers to collect and distribute royalties
- Enable automated royalty collection systems
- Delegate financial operations to trusted parties

---

## Granting Permissions

### Set a Single Permission

Grant one specific permission to an address:

```http
POST /api/permissions/set_permission/
Authorization: Bearer <token>
Content-Type: application/json

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
  "message": "Permission set successfully",
  "permission": {
    "uuid": "...",
    "asset": {
      "uuid": "...",
      "title": "Mystical Forest Path"
    },
    "permissioned_address": "0xabc123...",
    "permission_type": "execute",
    "is_active": true,
    "granted_at": "2024-12-04T10:30:00Z"
  },
  "tx_hash": "0xdef456..."
}
```

### Set All Permissions

Grant multiple permissions at once:

```http
POST /api/permissions/set_all_permissions/
Authorization: Bearer <token>
Content-Type: application/json

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
      "is_active": true
    },
    {
      "permission_type": "transfer_erc20",
      "is_active": false
    },
    {
      "permission_type": "set_metadata",
      "is_active": false
    },
    {
      "permission_type": "attach_license",
      "is_active": false
    },
    {
      "permission_type": "register_derivative",
      "is_active": true
    },
    {
      "permission_type": "collect_royalty",
      "is_active": false
    }
  ],
  "tx_hash": "0xdef456..."
}
```

---

## Revoking Permissions

### Revoke a Single Permission

```http
POST /api/permissions/revoke_permission/
Authorization: Bearer <token>
Content-Type: application/json

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
    "is_active": false,
    "revoked_at": "2024-12-04T15:00:00Z"
  },
  "tx_hash": "0xdef456..."
}
```

### Revoke All Permissions

Revoke all permissions for an address on an asset:

```http
POST /api/permissions/revoke_all_permissions/
Authorization: Bearer <token>
Content-Type: application/json

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
  "revoked_count": 3,
  "tx_hash": "0xdef456..."
}
```

---

## Checking Permissions

### Check a Specific Permission

```http
GET /api/permissions/check_permission/
Authorization: Bearer <token>
```

**Query Parameters:**
- `asset_uuid` (required)
- `permissioned_address` (required)
- `permission_type` (required)

**Response:**
```json
{
  "has_permission": true,
  "permission": {
    "uuid": "...",
    "permission_type": "execute",
    "is_active": true,
    "granted_at": "2024-12-04T10:30:00Z"
  }
}
```

### Get Permission Summary

Get all permissions for an address on an asset:

```http
GET /api/permissions/permission_summary/
Authorization: Bearer <token>
```

**Query Parameters:**
- `asset_uuid` (required)
- `permissioned_address` (required)

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

### List All Permissions for Asset

```http
GET /api/permissions/for_asset/
Authorization: Bearer <token>
```

**Query Parameters:**
- `asset_uuid` (required)

**Response:**
```json
{
  "count": 5,
  "results": [
    {
      "uuid": "...",
      "permissioned_address": "0xabc123...",
      "permission_type": "execute",
      "is_active": true,
      "granted_at": "2024-12-04T10:30:00Z"
    }
  ]
}
```

---

## API Reference

### Endpoints Summary

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/permissions/` | List permissions for user's assets |
| GET | `/api/permissions/{uuid}/` | Get permission details |
| POST | `/api/permissions/set_permission/` | Set a single permission |
| POST | `/api/permissions/set_all_permissions/` | Set all permissions |
| POST | `/api/permissions/revoke_permission/` | Revoke a permission |
| POST | `/api/permissions/revoke_all_permissions/` | Revoke all permissions |
| GET | `/api/permissions/for_asset/` | Get permissions for asset |
| GET | `/api/permissions/check_permission/` | Check if address has permission |
| GET | `/api/permissions/permission_summary/` | Get permission summary |

### Models

#### IPAccountPermission
- `uuid` - Public UUID
- `asset` - Foreign key to IPAsset
- `grantee_address` - Address with permission
- `permission_type` - execute/transfer_erc20/set_metadata/attach_license/register_derivative/collect_royalty
- `is_granted` - Whether permission is currently granted
- `expires_at` - When permission expires (null = no expiration)
- `granted_by` - User who granted permission
- `transaction_hash` - Blockchain transaction hash
- `created_at` - When permission was created
- `updated_at` - When permission was last updated

---

## Use Cases

### 1. Collaboration

**Scenario:** Two artists collaborate on a project. They grant each other derivative creation permissions.

```typescript
// Artist A grants permission to Artist B
await setPermission(assetUuid, artistBAddress, 'register_derivative');

// Artist B can now create derivatives of Artist A's asset
```

### 2. Delegation

**Scenario:** An artist delegates transaction authority to their manager.

```typescript
// Grant execute permission to manager
await setPermission(assetUuid, managerAddress, 'execute');

// Manager can now execute transactions on behalf of the artist
```

### 3. Community Remixes

**Scenario:** An artist allows community members to create authorized remixes.

```typescript
// Grant derivative permission to community member
await setPermission(assetUuid, communityMemberAddress, 'register_derivative');

// Community member can create remixes
```

### 4. Royalty Collection

**Scenario:** An artist allows their manager to collect royalties.

```typescript
// Grant collect_royalty permission to manager
await setPermission(assetUuid, managerAddress, 'collect_royalty');

// Manager can collect royalties on behalf of the artist
```

### 5. Automated Systems

**Scenario:** An artist uses an automated system to manage their IP assets.

```typescript
// Grant execute permission to automated system
await setPermission(assetUuid, automationAddress, 'execute');

// Automated system can execute transactions automatically
```

---

## Examples

### Example 1: Grant Single Permission

```typescript
const grantPermission = async (
  assetUuid: string,
  address: string,
  permissionType: string
) => {
  const response = await api.post('/api/permissions/set_permission/', {
    asset_uuid: assetUuid,
    permissioned_address: address,
    permission_type: permissionType
  });
  return response.data;
};

// Usage
await grantPermission(assetUuid, '0xabc123...', 'execute');
```

### Example 2: Grant Multiple Permissions

```typescript
const grantAllPermissions = async (
  assetUuid: string,
  address: string,
  permissions: {
    execute: boolean;
    transfer_erc20: boolean;
    set_metadata: boolean;
    attach_license: boolean;
    register_derivative: boolean;
    collect_royalty: boolean;
  }
) => {
  const response = await api.post('/api/permissions/set_all_permissions/', {
    asset_uuid: assetUuid,
    permissioned_address: address,
    permissions: permissions
  });
  return response.data;
};

// Usage
await grantAllPermissions(assetUuid, '0xabc123...', {
  execute: true,
  transfer_erc20: false,
  set_metadata: false,
  attach_license: false,
  register_derivative: true,
  collect_royalty: false
});
```

### Example 3: Check Permission

```typescript
const checkPermission = async (
  assetUuid: string,
  address: string,
  permissionType: string
) => {
  const response = await api.get('/api/permissions/check_permission/', {
    params: {
      asset_uuid: assetUuid,
      permissioned_address: address,
      permission_type: permissionType
    }
  });
  return response.data.has_permission;
};

// Usage
const hasPermission = await checkPermission(assetUuid, '0xabc123...', 'execute');
```

### Example 4: Revoke Permission

```typescript
const revokePermission = async (
  assetUuid: string,
  address: string,
  permissionType: string
) => {
  const response = await api.post('/api/permissions/revoke_permission/', {
    asset_uuid: assetUuid,
    permissioned_address: address,
    permission_type: permissionType
  });
  return response.data;
};

// Usage
await revokePermission(assetUuid, '0xabc123...', 'execute');
```

---

## Best Practices

1. **Verify Addresses**: Always verify the address before granting permissions
2. **Least Privilege**: Grant only the minimum permissions necessary
3. **Regular Review**: Periodically review and revoke unnecessary permissions
4. **Documentation**: Keep records of why permissions were granted
5. **Security**: Be cautious when granting execute or collect_royalty permissions
6. **Revocation**: Have a plan for revoking permissions if needed
7. **Expiration**: Consider setting expiration dates for temporary permissions

---

## Limitations

- Only the asset creator can grant/revoke permissions
- Permissions are stored on-chain (immutable after grant)
- Revoking permissions requires a blockchain transaction
- Each permission type must be set separately (or use set_all_permissions)
- Permissioned addresses must be valid Ethereum addresses

---

## Troubleshooting

### "Only the asset creator can set permissions"
**Solution:** Ensure you are authenticated as the asset creator

### "Asset does not exist"
**Solution:** Verify the asset UUID is correct and the asset exists

### "Invalid permission type"
**Solution:** Use one of: `execute`, `transfer_erc20`, `set_metadata`, `attach_license`, `register_derivative`, `collect_royalty`

### "Blockchain transaction failed"
**Solution:** Check gas balance, network connection, and try again

---

**Last Updated:** December 2025
**Related:** [API Documentation](./06-API-DOCUMENTATION.md) | [Solution Architecture](./03-SOLUTION-ARCHITECTURE.md)


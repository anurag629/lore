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
- **Permission Type**: The specific action allowed (signer, register_derivative, etc.)
- **Asset Creator**: The original creator who can grant/revoke permissions
- **On-Chain**: All permissions are stored on the blockchain via Story Protocol

---

## Permission Types

### 1. Signer (`signer`)

Allows the address to **sign transactions** on behalf of the IP asset.

**Use Cases:**
- Delegate signing authority to a manager or agent
- Enable automated systems to sign transactions
- Allow collaborators to sign on your behalf

### 2. Register Derivative (`register_derivative`)

Allows the address to **register derivatives** of the IP asset.

**Use Cases:**
- Allow collaborators to create official derivatives
- Enable community members to create authorized remixes
- Delegate derivative creation to a partner

### 3. Register Derivative with Attribution (`register_derivative_with_attribution`)

Allows the address to **register derivatives with attribution** to the original asset.

**Use Cases:**
- Enable derivative creation with proper attribution
- Allow partners to create derivatives while maintaining attribution
- Support collaborative derivative projects

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
  "permission_type": "signer"
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
    "permission_type": "signer",
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
    "signer": true,
    "register_derivative": true,
    "register_derivative_with_attribution": false
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
      "permission_type": "signer",
      "is_active": true
    },
    {
      "permission_type": "register_derivative",
      "is_active": true
    },
    {
      "permission_type": "register_derivative_with_attribution",
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
  "permission_type": "signer"
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
    "permission_type": "signer",
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
    "signer": true,
    "register_derivative": true,
    "register_derivative_with_attribution": false
  },
  "active_count": 2,
  "total_count": 3
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
      "permission_type": "signer",
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
- `permissioned_address` - Address with permission
- `permission_type` - signer/register_derivative/register_derivative_with_attribution
- `is_active` - Active status
- `granted_by` - User who granted permission
- `granted_at` - When permission was granted
- `revoked_at` - When permission was revoked (if revoked)

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

**Scenario:** An artist delegates signing authority to their manager.

```typescript
// Grant signer permission to manager
await setPermission(assetUuid, managerAddress, 'signer');

// Manager can now sign transactions on behalf of the artist
```

### 3. Community Remixes

**Scenario:** An artist allows community members to create authorized remixes.

```typescript
// Grant derivative permission to community member
await setPermission(assetUuid, communityMemberAddress, 'register_derivative_with_attribution');

// Community member can create remixes with proper attribution
```

### 4. Automated Systems

**Scenario:** An artist uses an automated system to manage their IP assets.

```typescript
// Grant signer permission to automated system
await setPermission(assetUuid, automationAddress, 'signer');

// Automated system can sign transactions automatically
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
await grantPermission(assetUuid, '0xabc123...', 'signer');
```

### Example 2: Grant Multiple Permissions

```typescript
const grantAllPermissions = async (
  assetUuid: string,
  address: string,
  permissions: {
    signer: boolean;
    register_derivative: boolean;
    register_derivative_with_attribution: boolean;
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
  signer: true,
  register_derivative: true,
  register_derivative_with_attribution: false
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
const hasPermission = await checkPermission(assetUuid, '0xabc123...', 'signer');
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
await revokePermission(assetUuid, '0xabc123...', 'signer');
```

---

## Best Practices

1. **Verify Addresses**: Always verify the address before granting permissions
2. **Least Privilege**: Grant only the minimum permissions necessary
3. **Regular Review**: Periodically review and revoke unnecessary permissions
4. **Documentation**: Keep records of why permissions were granted
5. **Security**: Be cautious when granting signer permissions
6. **Revocation**: Have a plan for revoking permissions if needed

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
**Solution:** Use one of: `signer`, `register_derivative`, `register_derivative_with_attribution`

### "Blockchain transaction failed"
**Solution:** Check gas balance, network connection, and try again

---

**Last Updated:** January 2025  
**Related:** [API Documentation](./06-API-DOCUMENTATION.md) | [Solution Architecture](./03-SOLUTION-ARCHITECTURE.md)


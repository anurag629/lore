# Asset Creation Workflow Guide

## Overview

The Asset Creation Workflow is a multi-step process that ensures reliable IP asset registration on Story Protocol. The system tracks each step and allows retrying from the point of failure, making the process resilient to network issues and service interruptions.

---

## Table of Contents

1. [Creation Steps](#creation-steps)
2. [Step-by-Step Process](#step-by-step-process)
3. [Retry Mechanism](#retry-mechanism)
4. [Error Handling](#error-handling)
5. [API Reference](#api-reference)
6. [Frontend Integration](#frontend-integration)
7. [Examples](#examples)

---

## Creation Steps

The asset creation process consists of 6 sequential steps:

1. **Media Upload to IPFS** (`media_upload`)
   - Upload media file to Pinata IPFS
   - Store IPFS hash for retry purposes

2. **Database Save** (`db_save`)
   - Save asset record to PostgreSQL
   - Initialize with `registration_status='pending'`

3. **Metadata Upload to IPFS** (`metadata_upload`)
   - Create metadata JSON
   - Upload to Pinata IPFS
   - Calculate keccak256 hash

4. **Story Protocol Registration** (`story_registration`)
   - Register IP asset on Story Protocol blockchain
   - Mint NFT and register IP ID
   - Attach PIL (Programmable IP License) terms

5. **License Terms Attachment** (`license_attachment`)
   - License terms are typically attached during registration
   - Fallback attachment if needed

6. **Completed** (`completed`)
   - All steps completed successfully
   - Asset is fully registered

---

## Step-by-Step Process

### Step 1: Media Upload to IPFS

**Purpose:** Upload the media file (image, audio, video) to IPFS via Pinata.

**Process:**
1. Receive media file or media URL
2. Upload to Pinata IPFS
3. Store IPFS hash in `media_ipfs_hash` field
4. Store IPFS URL in `media_url` field
5. Update `creation_step` to `metadata_upload`
6. Store results in `step_data['media_upload']`

**Idempotency:** If `media_ipfs_hash` already exists, skip upload and use existing hash.

**Error Handling:** If upload fails, set `registration_status='failed'` and `failed_at_step='media_upload'`.

---

### Step 2: Database Save

**Purpose:** Save asset record to database before external service calls.

**Process:**
1. Create asset record with minimal data
2. Set `registration_status='pending'`
3. Set `creation_step='media_upload'`
4. Initialize `step_data={}`

**Why First?** Ensures we have a database record to track progress even if external services fail.

---

### Step 3: Metadata Upload to IPFS

**Purpose:** Create and upload metadata JSON to IPFS.

**Process:**
1. Build metadata JSON with:
   - Title
   - Description
   - Media URL
   - Creator address
   - License terms
2. Upload to Pinata IPFS
3. Calculate keccak256 hash of metadata JSON
4. Store metadata URI and hash
5. Update `creation_step` to `story_registration`
6. Store results in `step_data['metadata_upload']`

**Metadata Structure:**
```json
{
  "name": "Asset Title",
  "description": "Asset description",
  "image": "ipfs://QmX5Z...",
  "attributes": [
    {
      "trait_type": "Creator",
      "value": "0x..."
    }
  ]
}
```

**Hash Calculation:**
```python
metadata_json = json.dumps(metadata, sort_keys=True)
metadata_hash = Web3.keccak(text=metadata_json).hex()[2:]  # Remove 0x prefix
```

---

### Step 4: Story Protocol Registration

**Purpose:** Register IP asset on Story Protocol blockchain.

**Process:**
1. Build IP metadata with camelCase keys:
   - `ipMetadataURI`: IPFS URI
   - `ipMetadataHash`: keccak256 hash (with 0x prefix)
2. Call `mint_and_register_ip_asset_with_pil_terms`
3. Store `story_ip_id` from response
4. Store `license_terms_id` from response
5. Update `creation_step` to `license_attachment` or `completed`
6. Store results in `step_data['story_registration']`

**PIL Terms:**
- `allow_derivatives`: Boolean
- `commercial_use`: Boolean
- `royalty_percentage`: Integer (0-100)

**Note:** License terms are automatically attached during registration via `mint_and_register_ip_asset_with_pil_terms`.

---

### Step 5: License Terms Attachment

**Purpose:** Attach license terms (usually done in Step 4, this is a fallback).

**Process:**
1. Check if `license_terms_id` exists from registration
2. If missing, attach license terms separately
3. Update `creation_step` to `completed`
4. Store results in `step_data['license_attachment']`

**Note:** This step is typically skipped as license terms are attached during registration.

---

### Step 6: Completed

**Purpose:** Mark asset as fully registered.

**Process:**
1. Set `registration_status='registered'`
2. Set `creation_step='completed'`
3. Clear `failed_at_step` and `registration_error`
4. Asset is now fully functional

---

## Retry Mechanism

### Automatic Retry

The system supports automatic retry from the point of failure:

1. **Identify Failed Step**: Check `failed_at_step` field
2. **Resume from Step**: Start from the failed step
3. **Use Stored Data**: Retrieve intermediate results from `step_data`
4. **Continue Process**: Complete remaining steps

### Manual Retry Endpoints

#### Retry Registration

Retry only the Story Protocol registration step:

```http
POST /api/assets/{uuid}/retry_registration/
Authorization: Bearer <token>
```

**Use Case:** Registration failed but metadata upload succeeded.

#### Retry Creation

Comprehensive retry from the failed step:

```http
POST /api/assets/{uuid}/retry_creation/
Authorization: Bearer <token>
```

**Use Case:** Any step failed, resume from that point.

### Retry Logic

```python
# Pseudo-code
if asset.failed_at_step == 'media_upload':
    # Retry media upload
    upload_media()
    
elif asset.failed_at_step == 'metadata_upload':
    # Use existing media, retry metadata upload
    upload_metadata()
    
elif asset.failed_at_step == 'story_registration':
    # Use existing metadata, retry registration
    register_on_blockchain()
```

---

## Error Handling

### Error States

- **`registration_status='pending'`**: Creation in progress
- **`registration_status='failed'`**: Creation failed at a step
- **`registration_status='retrying'`**: Retry in progress
- **`registration_status='registered'`**: Successfully registered

### Error Fields

- **`failed_at_step`**: Which step failed
- **`registration_error`**: Error message
- **`registration_attempts`**: Number of registration attempts

### Error Recovery

1. **Check Status**: Query asset to see current status
2. **Review Error**: Check `registration_error` for details
3. **Retry**: Use retry endpoints to resume
4. **Monitor**: Track progress through `creation_step`

---

## API Reference

### Create Asset

```http
POST /api/assets/
Authorization: Bearer <token>
Content-Type: application/json

{
  "title": "My Asset",
  "description": "Asset description",
  "media_url": "ipfs://..." // or media_file in multipart
}
```

**Response:**
```json
{
  "uuid": "...",
  "title": "My Asset",
  "registration_status": "registered", // or "failed", "pending"
  "creation_step": "completed", // or current step
  "failed_at_step": null, // or step name if failed
  "story_ip_id": "0x...",
  "metadata_hash": "0x...",
  "step_data": {
    "media_upload": {
      "ipfs_hash": "QmX5Z...",
      "ipfs_url": "ipfs://QmX5Z..."
    },
    "metadata_upload": {
      "ipfs_hash": "QmY6A...",
      "ipfs_url": "ipfs://QmY6A...",
      "metadata_hash": "0x..."
    },
    "story_registration": {
      "story_ip_id": "0x...",
      "license_terms_id": "...",
      "tx_hash": "0x..."
    }
  }
}
```

### Retry Endpoints

See [API Documentation](./06-API-DOCUMENTATION.md) for detailed endpoint documentation.

---

## Frontend Integration

### Creation Status Card

The frontend displays creation progress using the `CreationStatusCard` component:

```typescript
<CreationStatusCard
  asset={asset}
  onRetry={handleRetry}
  isRetrying={isRetrying}
/>
```

**Features:**
- Visual progress indicator
- Step-by-step status display
- Retry button for failed steps
- Error state display

### Polling for Status

Poll asset status during creation:

```typescript
const pollAssetStatus = async (assetUuid: string) => {
  const interval = setInterval(async () => {
    const asset = await fetchAsset(assetUuid);
    
    if (asset.registration_status === 'registered') {
      clearInterval(interval);
      // Success!
    } else if (asset.registration_status === 'failed') {
      clearInterval(interval);
      // Show error and retry option
    }
  }, 2000); // Poll every 2 seconds
};
```

### Retry Handler

```typescript
const handleRetry = async (assetUuid: string) => {
  try {
    await api.post(`/api/assets/${assetUuid}/retry_creation/`);
    // Start polling again
    pollAssetStatus(assetUuid);
  } catch (error) {
    // Show error
  }
};
```

---

## Examples

### Example 1: Successful Creation

```typescript
// 1. Create asset
const asset = await createAsset({
  title: "My Asset",
  description: "Description",
  media_file: file
});

// 2. Poll for completion
const checkStatus = async () => {
  const updated = await getAsset(asset.uuid);
  
  if (updated.registration_status === 'registered') {
    console.log('Asset registered!', updated.story_ip_id);
  } else if (updated.registration_status === 'failed') {
    console.error('Failed:', updated.registration_error);
    // Show retry button
  } else {
    // Still processing, check again
    setTimeout(checkStatus, 2000);
  }
};

checkStatus();
```

### Example 2: Retry After Failure

```typescript
// Asset failed at metadata_upload step
const asset = {
  uuid: "...",
  registration_status: "failed",
  failed_at_step: "metadata_upload",
  registration_error: "IPFS upload failed"
};

// Retry creation
const retry = async () => {
  await api.post(`/api/assets/${asset.uuid}/retry_creation/`);
  
  // Poll for completion
  pollAssetStatus(asset.uuid);
};
```

### Example 3: Check Step Data

```typescript
// Access intermediate results
const stepData = asset.step_data;

// Media upload results
const mediaHash = stepData.media_upload?.ipfs_hash;

// Metadata upload results
const metadataHash = stepData.metadata_upload?.metadata_hash;

// Registration results
const storyIpId = stepData.story_registration?.story_ip_id;
const licenseTermsId = stepData.story_registration?.license_terms_id;
```

---

## Best Practices

1. **Monitor Progress**: Poll asset status during creation
2. **Handle Errors**: Display clear error messages to users
3. **Retry Logic**: Provide retry buttons for failed assets
4. **Status Display**: Show current step to users
5. **Idempotency**: Use `idempotency_key` to prevent duplicate creations
6. **Timeout Handling**: Set reasonable timeouts for each step
7. **User Feedback**: Keep users informed of progress

---

## Troubleshooting

### "Asset stuck in pending"
**Solution:** Check if external services (IPFS, blockchain) are available. Retry the creation.

### "Media upload failed"
**Solution:** Check file size, format, and Pinata service status. Retry with smaller file or different format.

### "Metadata upload failed"
**Solution:** Check metadata JSON validity. Ensure all required fields are present.

### "Story Protocol registration failed"
**Solution:** Check gas balance, network connection, and contract addresses. Verify Story Protocol service is configured correctly.

### "License terms not attached"
**Solution:** This is usually automatic. If missing, the system will attempt separate attachment.

---

## Step Data Structure

The `step_data` JSONField stores intermediate results:

```json
{
  "media_upload": {
    "ipfs_hash": "QmX5Z...",
    "ipfs_url": "ipfs://QmX5Z...",
    "uploaded_at": "2024-12-04T10:30:00Z"
  },
  "metadata_upload": {
    "ipfs_hash": "QmY6A...",
    "ipfs_url": "ipfs://QmY6A...",
    "metadata_hash": "0xabc123...",
    "uploaded_at": "2024-12-04T10:31:00Z"
  },
  "story_registration": {
    "story_ip_id": "0xdef456...",
    "license_terms_id": "...",
    "tx_hash": "0x789ghi...",
    "registered_at": "2024-12-04T10:32:00Z"
  },
  "license_attachment": {
    "attached": true,
    "license_terms_id": "...",
    "attached_at": "2024-12-04T10:32:00Z"
  }
}
```

---

**Last Updated:** January 2025  
**Related:** [API Documentation](./06-API-DOCUMENTATION.md) | [Implementation Guide](./04-IMPLEMENTATION.md)


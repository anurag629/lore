# Frontend Components Guide

## Overview

This document provides comprehensive documentation for all frontend components in the Lore platform. Components are organized by category and include usage examples, props interfaces, and implementation details.

---

## Table of Contents

1. [Asset Components](#asset-components)
2. [Mint Components](#mint-components)
3. [Auth Components](#auth-components)
4. [UI Components](#ui-components)
5. [Validation Components](#validation-components)
6. [Social Components](#social-components)
7. [Profile Components](#profile-components)
8. [Layout Components](#layout-components)
9. [Keyboard Shortcuts](#keyboard-shortcuts)

---

## Asset Components

### ArchiveActions

**Location:** `components/assets/ArchiveActions.tsx`

**Purpose:** Provides restore and permanent delete actions for archived assets.

**Props:**
```typescript
interface ArchiveActionsProps {
  asset: IPAssetListItem;
  onSuccess?: () => void;
}
```

**Features:**
- Restore archived assets
- Permanently delete assets (with confirmation)
- Error handling and display
- Loading states

**Usage:**
```typescript
<ArchiveActions
  asset={archivedAsset}
  onSuccess={() => refetchAssets()}
/>
```

---

### CreationStatusCard

**Location:** `components/assets/CreationStatusCard.tsx`

**Purpose:** Displays multi-step asset creation progress with retry functionality.

**Props:**
```typescript
interface CreationStatusCardProps {
  asset: IPAsset;
  onRetry?: () => void;
  isRetrying?: boolean;
}
```

**Features:**
- Visual progress indicator for all 6 creation steps
- Step-by-step status display (completed, in_progress, failed, pending)
- Retry button for failed steps
- Error state display
- Animated progress indicators

**Steps Displayed:**
1. Media Upload to IPFS
2. Database Save
3. Metadata Upload to IPFS
4. Story Protocol Registration
5. License Terms Attachment
6. Completed

**Usage:**
```typescript
<CreationStatusCard
  asset={asset}
  onRetry={handleRetry}
  isRetrying={isRetrying}
/>
```

---

## Mint Components

### MintModal

**Location:** `components/mint/MintModal.tsx`

**Purpose:** Main modal for creating new IP assets with AI integration.

**Props:**
```typescript
interface MintModalProps {
  isOpen: boolean;
  onClose: () => void;
  onSuccess?: () => void;
}
```

**Features:**
- Multi-step form (media upload, title, description, license terms)
- AI-powered title generation
- AI-powered description enhancement
- File upload with preview
- License terms configuration
- Real-time validation
- Progress tracking

**Usage:**
```typescript
<MintModal
  isOpen={isMintModalOpen}
  onClose={() => setIsMintModalOpen(false)}
  onSuccess={() => {
    refetchAssets();
    setIsMintModalOpen(false);
  }}
/>
```

---

### RemixModal

**Location:** `components/mint/RemixModal.tsx`

**Purpose:** Modal for creating derivative assets (remixes) with multi-parent support.

**Props:**
```typescript
interface RemixModalProps {
  isOpen: boolean;
  onClose: () => void;
  parentAssets: IPAsset[];
  onSuccess?: () => void;
}
```

**Features:**
- Single or multi-parent derivative creation
- Attribution percentage configuration
- Parent asset selection
- AI-powered derivative analysis
- License terms validation
- Real-time attribution calculation

**Usage:**
```typescript
<RemixModal
  isOpen={isRemixModalOpen}
  onClose={() => setIsRemixModalOpen(false)}
  parentAssets={selectedAssets}
  onSuccess={() => refetchAssets()}
/>
```

---

### EditAssetModal

**Location:** `components/mint/EditAssetModal.tsx`

**Purpose:** Modal for editing asset metadata (title, description) with AI enhancement.

**Props:**
```typescript
interface EditAssetModalProps {
  isOpen: boolean;
  onClose: () => void;
  onSuccess?: () => void;
  asset: IPAsset | null;
  redirectTo?: string;
}
```

**Features:**
- Edit title and description
- AI-powered description enhancement
- Delete asset functionality
- Confirmation dialogs
- Redirect after deletion

**Usage:**
```typescript
<EditAssetModal
  isOpen={isEditModalOpen}
  onClose={() => setIsEditModalOpen(false)}
  asset={selectedAsset}
  onSuccess={() => refetchAssets()}
/>
```

---

## Auth Components

### WalletConnect

**Location:** `components/auth/WalletConnect.tsx`

**Purpose:** Wallet connection and authentication component with SIWE integration.

**Features:**
- Wallet connection via Reown AppKit
- SIWE (Sign-In With Ethereum) authentication
- User menu with profile link
- Logout functionality
- Loading states
- Error handling

**Usage:**
```typescript
<WalletConnect />
```

**Auto-login:** Automatically triggers SIWE login when wallet connects.

---

## UI Components

### Button

**Location:** `components/ui/Button.tsx`

**Purpose:** Reusable button component with variants.

**Props:**
```typescript
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'outline' | 'ghost' | 'danger';
  className?: string;
}
```

**Variants:**
- `primary` - Main action button
- `secondary` - Secondary action
- `outline` - Outlined button
- `ghost` - Minimal button
- `danger` - Destructive action

**Usage:**
```typescript
<Button variant="primary" onClick={handleClick}>
  Create Asset
</Button>
```

---

### Toast

**Location:** `components/ui/Toast.tsx`

**Purpose:** Toast notification system for user feedback.

**Hook:**
```typescript
const { showToast } = useToast();

showToast('Asset created successfully', 'success');
showToast('Error creating asset', 'error');
showToast('Processing...', 'info');
```

**Types:**
- `success` - Green toast
- `error` - Red toast
- `info` - Blue toast
- `warning` - Yellow toast

---

### OptimizedImage

**Location:** `components/ui/OptimizedImage.tsx`

**Purpose:** Optimized image loading with lazy loading and placeholder support.

**Props:**
```typescript
interface OptimizedImageProps {
  src: string;
  alt: string;
  className?: string;
  width?: number;
  height?: number;
  placeholder?: string;
}
```

**Features:**
- Lazy loading
- Placeholder support
- Error handling
- Responsive sizing

**Usage:**
```typescript
<OptimizedImage
  src={asset.media_url}
  alt={asset.title}
  placeholder="/placeholder.png"
/>
```

---

### ScrollProgress

**Location:** `components/ui/ScrollProgress.tsx`

**Purpose:** Visual scroll progress indicator.

**Features:**
- Reading progress tracking
- Smooth animation
- Customizable styling

**Usage:**
```typescript
<ScrollProgress />
```

---

### EmptyState

**Location:** `components/ui/EmptyState.tsx`

**Purpose:** Empty state component for when no data is available.

**Props:**
```typescript
interface EmptyStateProps {
  icon: React.ComponentType<{ className?: string }>;
  title: string;
  description: string;
  action?: {
    label: string;
    onClick: () => void;
  };
}
```

**Usage:**
```typescript
<EmptyState
  icon={FileText}
  title="No assets yet"
  description="Create your first IP asset to get started"
  action={{
    label: "Create Asset",
    onClick: () => openMintModal()
  }}
/>
```

---

## Validation Components

### ValidationButton

**Location:** `components/validation/ValidationButton.tsx`

**Purpose:** Button component that triggers AI-powered asset validation.

**Props:**
```typescript
interface ValidationButtonProps {
  assetUuid: string;
  onValidationComplete?: (result: any) => void;
  variant?: 'default' | 'outline' | 'ghost';
  size?: 'default' | 'sm' | 'lg';
}
```

**Features:**
- Triggers AI validation
- Opens validation result dialog
- Toast notifications
- Loading states

**Usage:**
```typescript
<ValidationButton
  assetUuid={asset.uuid}
  onValidationComplete={(result) => {
    console.log('Validation result:', result);
  }}
/>
```

---

### ValidationResult

**Location:** `components/validation/ValidationResult.tsx`

**Purpose:** Displays detailed AI validation results.

**Props:**
```typescript
interface ValidationResultProps {
  result: ValidationResult;
  compact?: boolean;
}
```

**Features:**
- Verdict display (approved/rejected/needs_review)
- Detailed feedback sections
- Score visualization
- Recommendations

**Validation Checks:**
- Content appropriateness
- Copyright compliance
- Quality assessment
- Originality check

---

## Social Components

### ShareModal

**Location:** `components/share/ShareModal.tsx`

**Purpose:** Social sharing modal for assets.

**Props:**
```typescript
interface ShareModalProps {
  isOpen: boolean;
  onClose: () => void;
  title: string;
  url: string;
  description?: string;
}
```

**Features:**
- Copy link to clipboard
- Twitter sharing
- Facebook sharing
- Email sharing
- Custom share text

**Usage:**
```typescript
<ShareModal
  isOpen={isShareModalOpen}
  onClose={() => setIsShareModalOpen(false)}
  title={asset.title}
  url={`/assets/${asset.uuid}`}
  description={asset.description}
/>
```

---

### CommentCard

**Location:** `components/comments/CommentCard.tsx`

**Purpose:** Displays a comment with replies and like functionality.

**Features:**
- Comment content display
- User avatar and info
- Like/unlike functionality
- Reply thread
- Timestamp formatting
- Delete functionality (for own comments)

---

### CommentsSection

**Location:** `components/comments/CommentsSection.tsx`

**Purpose:** Full comments section with create, list, and reply functionality.

**Features:**
- List all comments
- Create new comments
- Reply to comments
- Like comments
- Real-time updates
- Pagination

---

## Profile Components

### EditProfileModal

**Location:** `components/profile/EditProfileModal.tsx`

**Purpose:** Modal for editing user profile information.

**Features:**
- Edit username
- Edit bio
- Upload avatar
- Upload banner
- Image cropping integration

---

### ImageCropModal

**Location:** `components/profile/ImageCropModal.tsx`

**Purpose:** Custom HTML canvas-based image cropping modal.

**Props:**
```typescript
interface ImageCropModalProps {
  isOpen: boolean;
  onClose: () => void;
  image: File | string;
  aspectRatio?: number;
  onCrop: (croppedFile: File) => void;
}
```

**Features:**
- Canvas-based cropping
- Aspect ratio control
- Zoom and pan
- Preview
- Round crop for avatars
- Rectangular crop for banners

**Usage:**
```typescript
<ImageCropModal
  isOpen={isCropModalOpen}
  onClose={() => setIsCropModalOpen(false)}
  image={selectedImage}
  aspectRatio={1} // Square for avatar
  onCrop={(file) => uploadAvatar(file)}
/>
```

---

## Layout Components

### Sidebar

**Location:** `components/layout/Sidebar.tsx`

**Purpose:** Main navigation sidebar with keyboard shortcuts integration.

**Features:**
- Navigation links (Home, Explore, Dashboard, Collections, Profile)
- Mint button
- Wallet connection
- Keyboard shortcuts integration
- Active route highlighting

**Navigation Items:**
- Home (`/`)
- Explore (`/explore`)
- Dashboard (`/dashboard`)
- Collections (`/collections`)
- Profile (`/profile`)

---

## Keyboard Shortcuts

### KeyboardShortcutsProvider

**Location:** `components/keyboard/KeyboardShortcutsProvider.tsx`

**Purpose:** Global keyboard shortcuts provider.

**Shortcuts:**
- `Cmd/Ctrl + K` - Focus search input
- `Cmd/Ctrl + N` - Open mint modal
- `Escape` - Close modals

**Usage:**
```typescript
// Wrap app with provider
<KeyboardShortcutsProvider>
  <App />
</KeyboardShortcutsProvider>

// Use in components
const { openMintModal, closeMintModal } = useKeyboardShortcutsContext();
```

**Features:**
- Global keyboard event handling
- Context-based modal management
- Search focus management
- Escape key handling

---

## Collections Components

### CollectionModal

**Location:** `components/collections/CollectionModal.tsx`

**Purpose:** Modal for creating and managing collections.

**Features:**
- Create collections
- Add assets to collections
- Remove assets from collections
- Collection management
- Public/private toggle

---

## Component Patterns

### Error Boundaries

**Location:** `components/ErrorBoundary.tsx`

**Purpose:** Catches React errors and displays fallback UI.

**Usage:**
```typescript
<ErrorBoundary>
  <YourComponent />
</ErrorBoundary>
```

---

### Loading States

Most components support loading states:

```typescript
const { loading, error, data } = useAssets();

if (loading) return <Skeleton />;
if (error) return <Error message={error} />;
return <AssetList assets={data} />;
```

---

### Responsive Design

All components are built with Tailwind CSS and are responsive:

- Mobile-first approach
- Breakpoints: `sm`, `md`, `lg`, `xl`, `2xl`
- Flexible layouts
- Touch-friendly interactions

---

## Styling

### Tailwind CSS

All components use Tailwind CSS for styling:

```typescript
className="flex items-center gap-2 px-4 py-2 bg-blue-600 hover:bg-blue-700 rounded-lg"
```

### Dark Mode

Components support dark mode via Tailwind's dark mode:

```typescript
className="bg-white dark:bg-gray-900 text-gray-900 dark:text-gray-100"
```

---

## Performance Optimizations

### Code Splitting

Large components are code-split:

```typescript
const MintModal = dynamic(() => import('@/components/mint/MintModal'), {
  ssr: false
});
```

### Image Optimization

Images use Next.js Image component:

```typescript
import Image from 'next/image';

<Image
  src={asset.media_url}
  alt={asset.title}
  width={500}
  height={500}
  loading="lazy"
/>
```

### React Query Caching

Data fetching uses React Query for caching:

```typescript
const { data } = useQuery({
  queryKey: ['assets', filters],
  queryFn: () => fetchAssets(filters),
  staleTime: 5 * 60 * 1000, // 5 minutes
});
```

---

## Testing

### Component Testing

Components can be tested with React Testing Library:

```typescript
import { render, screen } from '@testing-library/react';
import Button from '@/components/ui/Button';

test('renders button', () => {
  render(<Button>Click me</Button>);
  expect(screen.getByText('Click me')).toBeInTheDocument();
});
```

---

## Best Practices

1. **Props Interface**: Always define TypeScript interfaces for props
2. **Error Handling**: Handle errors gracefully with user-friendly messages
3. **Loading States**: Show loading indicators during async operations
4. **Accessibility**: Use semantic HTML and ARIA attributes
5. **Performance**: Optimize with code splitting and lazy loading
6. **Responsive**: Ensure components work on all screen sizes
7. **Consistency**: Use shared UI components for consistency

---

**Last Updated:** January 2025  
**Related:** [API Documentation](./06-API-DOCUMENTATION.md) | [Implementation Guide](./04-IMPLEMENTATION.md)


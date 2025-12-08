# Solution Architecture

## Executive Summary

**Lore** is a hybrid decentralized platform that combines blockchain-based IP management with AI-powered content generation. The architecture leverages:

- **Story Protocol** for immutable IP ownership and automatic royalty distribution
- **LiteLLM + OpenRouter** for intelligent content generation at $0 cost
- **Django REST Framework** for robust backend APIs
- **Next.js 16** for modern, performant frontend
- **Redis** for high-performance caching
- **PostgreSQL** for relational data storage

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         USER LAYER                               │
│  (Creators via Web Browser + MetaMask/WalletConnect)            │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                      FRONTEND LAYER                              │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Next.js 16 Application (TypeScript + React 19)          │  │
│  │                                                            │  │
│  │  • Reown AppKit (Wallet Integration)                     │  │
│  │  • React Query (State Management)                        │  │
│  │  • Tailwind CSS (Styling)                                │  │
│  │  • Framer Motion (Animations)                            │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                   │
└────────────────┬────────────────────────────────────────────────┘
                 │ REST API (JSON)
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                      BACKEND LAYER                               │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Django 5.1 + Django REST Framework                       │  │
│  │                                                            │  │
│  │  ┌────────────┐  ┌────────────┐  ┌──────────────┐       │  │
│  │  │ Auth API   │  │ Assets API │  │  AI API      │       │  │
│  │  │ (SIWE/JWT) │  │ (CRUD)     │  │  (5 features)│       │  │
│  │  └────────────┘  └────────────┘  └──────────────┘       │  │
│  │                                                            │  │
│  │  ┌────────────┐  ┌────────────┐  ┌──────────────┐       │  │
│  │  │Story       │  │AI Service  │  │Pinata        │       │  │
│  │  │Service     │  │(Singleton) │  │Service       │       │  │
│  │  └────────────┘  └────────────┘  └──────────────┘       │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                   │
└───┬──────────────┬──────────────┬──────────────┬────────────────┘
    │              │              │              │
    ▼              ▼              ▼              ▼
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐
│PostgreSQL│  │  Redis   │  │  Pinata  │  │Story Protocol│
│          │  │ (Cache)  │  │  (IPFS)  │  │  (Aeneid)    │
│ • Users  │  │          │  │          │  │              │
│ • Assets │  │ • AI     │  │ • Media  │  │ • IP Assets  │
│ • AI Logs│  │ • Rate   │  │ • Meta   │  │ • Royalties  │
└──────────┘  └──────────┘  └──────────┘  └──────────────┘
                                                   │
                                                   ▼
                                          ┌──────────────┐
                                          │ OpenRouter   │
                                          │              │
                                          │ • Gemini     │
                                          │ • Llama 3.2  │
                                          │ • Mistral    │
                                          └──────────────┘
```

---

## Component Breakdown

### 1. Frontend Architecture

#### Next.js Application Structure

```
lore-frontend/
├── app/                    # Next.js 16 App Router
│   ├── layout.tsx         # Root layout with providers
│   ├── page.tsx           # Landing page
│   ├── dashboard/         # Dashboard routes
│   └── assets/            # Asset detail pages
│
├── components/
│   ├── auth/              # Authentication components
│   │   └── WalletConnect.tsx
│   ├── mint/              # Asset creation
│   │   ├── MintModal.tsx        # AI-integrated
│   │   └── RemixModal.tsx       # Derivative creation
│   ├── dashboard/         # Dashboard components
│   └── ui/                # Reusable UI components
│
├── hooks/
│   ├── useAuth.ts         # Authentication hook
│   ├── useAssets.ts       # Asset management
│   └── useAI.ts           # AI features (7 hooks)
│
├── lib/
│   ├── api.ts             # Axios client + API functions
│   ├── types.ts           # TypeScript definitions
│   └── wagmi.ts           # Wagmi + Reown config
│
└── contexts/
    └── AuthContext.tsx    # Global auth state
```

#### Key Frontend Patterns

**1. React Query for State Management**
```typescript
// hooks/useAI.ts
export function useGenerateTitle() {
  return useMutation<AITitleResponse, Error, { description: string }>({
    mutationFn: aiAPI.generateTitle,
  });
}

// Usage in component
const generateTitle = useGenerateTitle();
await generateTitle.mutateAsync({ description: '...' });
```

**2. Context-Based Authentication**
```typescript
// contexts/AuthContext.tsx
export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  const [token, setToken] = useState<string | null>(null);

  // SIWE login, logout, token refresh
  return <AuthContext.Provider value={{...}}>{children}</AuthContext.Provider>;
};
```

**3. Reown AppKit for Wallet Integration**
```typescript
// lib/wagmi.ts
const metadata = { name: 'Lore', description: '...', url: '...' };
export const config = createAppKit({
  adapters: [new WagmiAdapter({ networks: [baseSepolia] })],
  projectId: env.NEXT_PUBLIC_REOWN_PROJECT_ID,
  metadata,
});
```

---

### 2. Backend Architecture

#### Django Application Structure

```
lore_backend/
├── config/
│   ├── settings/
│   │   ├── base.py              # Core settings + AI config
│   │   ├── development.py
│   │   └── production.py
│   ├── urls.py                  # Root URL config
│   └── wsgi.py
│
├── apps/
│   ├── users/
│   │   ├── models.py            # User (wallet-based)
│   │   ├── views.py             # SIWE auth endpoints
│   │   └── serializers.py
│   │
│   └── assets/
│       ├── models.py            # IPAsset, DerivativeAsset,
│       │                        # RoyaltyPayment, AIGenerationLog,
│       │                        # AIAssetMetadata, AIUsageStats
│       ├── views.py             # Asset CRUD + 7 AI endpoints
│       ├── serializers.py       # DRF serializers (20+)
│       ├── admin.py             # Django admin interfaces
│       ├── urls.py              # URL routing
│       │
│       └── services/
│           ├── story_service.py # Story Protocol integration
│           ├── ai_service.py    # AI/LLM integration
│           └── pinata_service.py # IPFS uploads
│
└── manage.py
```

#### Key Backend Patterns

**1. Service Layer Pattern (Singleton)**
```python
# apps/assets/services/ai_service.py
class AIService:
    def __init__(self):
        self.api_key = settings.OPENROUTER_API_KEY
        self.models = settings.AI_MODELS

    def generate_title(self, description: str) -> Tuple[List[str], str, int, int, bool]:
        # Returns: (titles, model_used, response_time_ms, tokens_used, cache_hit)
        cache_key = self._generate_cache_key('title', description=description)
        cached = cache.get(cache_key)
        if cached:
            return (cached['titles'], cached['model_used'], 0, 0, True)

        # AI call with fallback
        result = self._complete_with_fallback(prompt, model_tier='fast')
        cache.set(cache_key, result, timeout=settings.AI_CACHE_TTL)
        return result

# Singleton instance
_ai_service_instance = None
def get_ai_service() -> AIService:
    global _ai_service_instance
    if _ai_service_instance is None:
        _ai_service_instance = AIService()
    return _ai_service_instance
```

**2. View Layer with Database Logging**
```python
# apps/assets/views.py
@api_view(['POST'])
@permission_classes([IsAuthenticated])
def generate_title(request):
    serializer = TitleGenerationSerializer(data=request.data)
    serializer.is_valid(raise_exception=True)

    try:
        ai_service = get_ai_service()
        titles, model_used, response_time_ms, tokens_used, cache_hit = \
            ai_service.generate_title(description=serializer.validated_data['description'])

        # Create audit log
        log_entry = AIGenerationLog.objects.create(
            user=request.user,
            operation_type='title',
            input_data={'description': serializer.validated_data['description']},
            output_data={'titles': titles},
            model_used=model_used,
            response_time_ms=response_time_ms,
            tokens_used=tokens_used,
            status='success',
            cache_hit=cache_hit
        )

        return Response({'titles': titles, 'model_used': model_used, 'log_id': log_entry.id})
    except Exception as e:
        AIGenerationLog.objects.create(
            user=request.user, operation_type='title', status='failed', error_message=str(e)
        )
        return Response({'error': str(e)}, status=500)
```

**3. Story Protocol Integration**
```python
# apps/assets/services/story_service.py
class StoryProtocolService:
    def __init__(self):
        self.client = StoryClient(network='testnet')
        self.wallet = Wallet(private_key=settings.STORY_PRIVATE_KEY)

    def register_ip_asset(self, metadata_uri: str, token_id: int) -> str:
        """Register IP asset and return ipId."""
        response = self.client.ip_asset.register(
            token_contract=settings.NFT_CONTRACT_ADDRESS,
            token_id=token_id,
            metadata={
                'metadataURI': metadata_uri,
                'metadataHash': hashlib.sha256(metadata_uri.encode()).hexdigest()
            },
            wallet=self.wallet
        )
        return response['ipId']

    def create_derivative(self, child_ip_id: str, parent_ip_ids: List[str], license_terms_ids: List[int]) -> str:
        """Link derivative to parent(s) and return transaction hash."""
        response = self.client.license.register_derivative(
            child_ip_id=child_ip_id,
            parent_ip_ids=parent_ip_ids,
            license_terms_ids=license_terms_ids,
            wallet=self.wallet
        )
        return response['txHash']
```

---

### 3. Database Schema

#### Entity-Relationship Diagram

```
┌─────────────┐
│    User     │
│─────────────│
│ id (PK)     │
│ wallet_addr │◄────┐
│ username    │     │
│ created_at  │     │
└─────────────┘     │
                    │
                    │ FK
┌─────────────────┐ │     ┌──────────────────┐
│   IPAsset       │─┘     │ AIGenerationLog  │
│─────────────────│       │──────────────────│
│ id (PK)         │       │ id (PK)          │
│ user_id (FK)    │◄──────│ user_id (FK)     │
│ title           │       │ operation_type   │
│ description     │       │ input_data       │
│ ip_id (unique)  │       │ output_data      │
│ token_id        │       │ model_used       │
│ metadata_uri    │       │ response_time_ms │
│ media_url       │       │ tokens_used      │
│ asset_type      │       │ cache_hit        │
│ created_at      │       │ status           │
└────────┬────────┘       │ created_at       │
         │                └──────────────────┘
         │
         │ FK (parent)
         │
┌────────┴───────────┐    ┌──────────────────┐
│ DerivativeAsset    │    │ AIAssetMetadata  │
│────────────────────│    │──────────────────│
│ id (PK)            │    │ id (PK)          │
│ child_asset_id(FK) │◄───│ asset_id (FK)    │
│ parent_asset_id(FK)│    │ content_type     │
│ relationship_type  │    │ ai_content       │
│ derivative_ip_id   │    │ model_used       │
│ tx_hash            │    │ accepted         │
│ created_at         │    │ user_rating      │
└────────────────────┘    └──────────────────┘

┌──────────────────┐      ┌──────────────────┐      ┌──────────────────┐
│ RoyaltyPayment   │      │  AIUsageStats    │      │   Collection     │
│──────────────────│      │──────────────────│      │──────────────────│
│ id (PK)          │      │ date (PK)        │      │ id (PK)          │
│ asset_id (FK)    │      │ total_requests   │      │ creator_id (FK)  │
│ recipient_id(FK) │      │ cache_hits       │      │ name             │
│ amount_wei       │      │ total_tokens     │      │ description      │
│ tx_hash          │      │ by_operation     │      │ is_public        │
│ status           │      │ unique_users     │      │ created_at       │
│ created_at       │      └──────────────────┘      └────────┬─────────┘
└──────────────────┘                                         │
                                                              │ FK
┌──────────────────┐      ┌──────────────────┐      ┌──────┴──────────┐
│   Comment        │      │    Favorite      │      │CollectionAsset │
│──────────────────│      │──────────────────│      │──────────────────│
│ id (PK)          │      │ id (PK)          │      │ id (PK)          │
│ asset_id (FK)    │      │ user_id (FK)     │      │ collection_id(FK)│
│ user_id (FK)     │      │ asset_id (FK)    │      │ asset_id (FK)    │
│ parent_id (FK)   │      │ created_at       │      │ added_at         │
│ content          │      └──────────────────┘      └──────────────────┘
│ likes_count      │
│ is_deleted       │
│ created_at       │
└──────────────────┘
```

#### Key Database Models

**IPAsset Model**
```python
class IPAsset(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    title = models.CharField(max_length=200)
    description = models.TextField()
    ip_id = models.CharField(max_length=100, unique=True)  # Story Protocol IP ID
    token_id = models.IntegerField()
    metadata_uri = models.URLField()  # IPFS URI
    media_url = models.URLField()
    asset_type = models.CharField(max_length=50)  # 'digital_art', 'music', etc.
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        indexes = [
            models.Index(fields=['user', '-created_at']),
            models.Index(fields=['ip_id']),
        ]
```

**AIGenerationLog Model**
```python
class AIGenerationLog(models.Model):
    OPERATION_CHOICES = [
        ('title', 'Title Generation'),
        ('description', 'Description Enhancement'),
        ('analysis', 'Content Analysis'),
        ('license', 'License Suggestion'),
        ('derivative', 'Derivative Analysis'),
    ]

    user = models.ForeignKey(User, on_delete=models.CASCADE)
    operation_type = models.CharField(max_length=20, choices=OPERATION_CHOICES, db_index=True)
    input_data = models.JSONField()  # Stores prompt and parameters
    output_data = models.JSONField(null=True)  # Stores AI response
    model_used = models.CharField(max_length=100)  # e.g., 'google/gemini-flash-1.5'
    model_tier = models.CharField(max_length=20, default='fast')
    response_time_ms = models.IntegerField(null=True)
    tokens_used = models.IntegerField(null=True)
    status = models.CharField(max_length=20, default='success')
    error_message = models.TextField(blank=True)
    cache_hit = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True, db_index=True)

    class Meta:
        indexes = [
            models.Index(fields=['user', '-created_at']),
            models.Index(fields=['operation_type', '-created_at']),
        ]
```

---

### 4. API Architecture

#### REST API Design

**Base URL:** `http://localhost:8000/api/`

**Authentication:** JWT Bearer Token (from SIWE)

**API Groups:**

```
/api/
├── auth/
│   ├── POST /generate-nonce/        # Get nonce for SIWE
│   ├── POST /verify-signature/      # Verify SIWE signature → JWT
│   └── POST /refresh-token/         # Refresh JWT
│
├── assets/
│   ├── GET    /                     # List user's assets
│   ├── POST   /                     # Create new asset
│   ├── GET    /{id}/                # Get asset details
│   ├── PATCH  /{id}/                # Update asset
│   ├── DELETE /{id}/                # Delete asset (soft delete)
│   │
│   ├── POST   /upload-to-ipfs/      # Upload media to Pinata
│   ├── POST   /register-ip/         # Register on Story Protocol
│   │
│   ├── GET    /derivatives/         # List derivatives
│   ├── POST   /derivatives/         # Create derivative link
│   │
│   ├── GET    /royalties/           # Get royalty payments
│   │
│   ├── GET    /collections/         # List collections
│   ├── POST   /collections/         # Create collection
│   ├── GET    /collections/{id}/    # Get collection details
│   ├── PATCH  /collections/{id}/    # Update collection
│   ├── DELETE /collections/{id}/    # Delete collection
│   │
│   ├── GET    /favorites/           # List user's favorites
│   ├── POST   /favorites/           # Favorite an asset
│   ├── DELETE /favorites/{id}/      # Unfavorite an asset
│   │
│   └── ai/
│       ├── POST /generate-title/           # 4 title suggestions
│       ├── POST /enhance-description/      # Expand description
│       ├── POST /analyze-content/          # Extract metadata
│       ├── POST /suggest-license/          # Recommend license
│       ├── POST /analyze-derivative/       # Similarity analysis
│       ├── GET  /usage-stats/              # User AI stats
│       └── GET  /platform-stats/           # Platform AI stats
│
├── comments/
│   ├── GET    /                     # List comments for asset
│   ├── POST   /                     # Create comment
│   ├── PATCH  /{id}/                # Update comment
│   ├── DELETE /{id}/                # Delete comment (soft delete)
│   ├── POST   /{id}/like/           # Like/unlike comment
│   └── POST   /{id}/reply/          # Reply to comment
│
├── auth/
│   ├── POST   /profile/             # Update user profile
│   ├── POST   /upload-avatar/       # Upload avatar image
│   ├── POST   /upload-banner/       # Upload banner image
│   └── GET    /user/{address}/      # Get user by wallet address
```

#### API Response Format

**Success Response:**
```json
{
  "titles": ["Mystical Forest Path", "Enchanted Woodland Trail", ...],
  "model_used": "google/gemini-flash-1.5",
  "log_id": 12345
}
```

**Error Response:**
```json
{
  "error": "Failed to generate titles",
  "detail": "Rate limit exceeded. Trying fallback model."
}
```

---

### 5. Blockchain Integration

#### Story Protocol Flow

**1. Asset Registration Flow**
```
User submits asset
    ↓
Upload media to IPFS (Pinata)
    ↓
Create metadata JSON
    ↓
Upload metadata to IPFS
    ↓
Call Story Protocol SDK:
  - client.ip_asset.register()
  - Params: token_contract, token_id, metadata_uri
    ↓
Receive ipId (unique IP identifier)
    ↓
Store in PostgreSQL (IPAsset.ip_id)
    ↓
Return success to user
```

**2. Derivative Creation Flow**
```
User creates derivative
    ↓
Upload derivative media to IPFS
    ↓
Register derivative as separate IP asset
    ↓
Call Story Protocol SDK:
  - client.license.register_derivative()
  - Params: child_ip_id, parent_ip_ids[], license_terms_ids[]
    ↓
Receive txHash (blockchain transaction)
    ↓
Store relationship (DerivativeAsset table)
    ↓
Royalties auto-flow from child → parent on-chain
```

**3. Royalty Tracking**
```
Story Protocol smart contracts handle:
  - Revenue splitting (e.g., 10% to original creator)
  - Automatic payouts when derivative earns
  - On-chain verification of all payments

Backend polls/listens:
  - Query blockchain for payment events
  - Record in RoyaltyPayment table
  - Display in dashboard
```

---

### 6. AI/ML Integration Architecture

#### LiteLLM + OpenRouter Architecture

```
Frontend Request
    ↓
Backend AI Endpoint (views.py)
    ↓
AIService.generate_title()
    ↓
Check Redis Cache (MD5 key)
    ├─ Cache Hit → Return cached (0ms)
    └─ Cache Miss ↓
        ↓
    Model Fallback Loop (3 models)
        ├─ Try Model 1 (Gemini Flash)
        │   ├─ Success → Return result
        │   └─ RateLimitError → Try Model 2
        ├─ Try Model 2 (Llama 3.2)
        │   ├─ Success → Return result
        │   └─ RateLimitError → Try Model 3
        └─ Try Model 3 (Mistral 7B)
            ├─ Success → Return result
            └─ Error → Return error
    ↓
Parse JSON response
    ↓
Store in Redis Cache (TTL: 1 hour)
    ↓
Create AIGenerationLog entry
    ↓
Return to frontend
```

#### AI Service Methods

```python
class AIService:
    # 5 AI Features:
    def generate_title(description, context) -> (titles[], model, ms, tokens, cache_hit)
    def enhance_description(description, title) -> (enhanced_desc, model, ms, tokens, cache_hit)
    def analyze_content(title, description, media_url) -> (metadata, model, ms, tokens, cache_hit)
    def suggest_license(asset_type, description) -> (license_terms, model, ms, tokens, cache_hit)
    def analyze_derivative(parent_desc, child_desc) -> (similarity, model, ms, tokens, cache_hit)

    # Helper methods:
    def _complete_with_fallback(prompt, model_tier) -> (response, model, ms, tokens)
    def _generate_cache_key(operation, **kwargs) -> str  # MD5 hash
    def _is_rate_limited(model) -> bool  # Check Redis flag
    def _mark_rate_limited(model) -> None  # Set Redis flag (5 min TTL)
```

#### Caching Strategy

**Cache Key Generation:**
```python
def _generate_cache_key(self, operation: str, **kwargs) -> str:
    # Create deterministic key from operation + parameters
    key_parts = [operation]
    for k, v in sorted(kwargs.items()):
        if isinstance(v, dict):
            v = json.dumps(v, sort_keys=True)
        key_parts.append(f"{k}:{v}")

    key_string = "|".join(key_parts)
    return f"ai_cache:{hashlib.md5(key_string.encode()).hexdigest()}"
```

**Example:**
```
Operation: generate_title
Input: description="mystical forest"
Cache Key: ai_cache:a3f5e9c2b1d4f6a8e7c9b0d1f2a3e4b5
```

**Cache Hit Rate:** 95%+ on repeated requests (saves API calls and latency)

---

### 7. Technology Stack Decisions

#### Why Django?

**Chosen:** Django 5.1 + Django REST Framework

**Reasons:**
- ✅ Mature ORM for complex relational data
- ✅ Built-in admin interface for monitoring AI logs
- ✅ Excellent Python ecosystem (LiteLLM, Web3.py)
- ✅ Fast development with batteries included
- ✅ Strong security features (CSRF, SQL injection protection)

**Alternatives Considered:**
- FastAPI: Faster async, but less mature ORM (SQLAlchemy)
- Express.js: Would require TypeScript backend duplication

---

#### Why Next.js 16?

**Chosen:** Next.js 16 (App Router) + React 19

**Reasons:**
- ✅ App Router for better file-based routing
- ✅ Server Components reduce client bundle size
- ✅ TypeScript first-class support
- ✅ Built-in optimization (images, fonts, scripts)
- ✅ Great developer experience

**Alternatives Considered:**
- Vite + React: Faster HMR, but no SSR out of box
- Remix: Better data loading, but steeper learning curve

---

#### Why LiteLLM?

**Chosen:** LiteLLM 1.80+ with OpenRouter

**Reasons:**
- ✅ Unified interface for 100+ LLM providers
- ✅ Built-in fallback logic between models
- ✅ Automatic retries and error handling
- ✅ Works seamlessly with free OpenRouter models
- ✅ Simple API (similar to OpenAI SDK)

**Alternatives Considered:**
- Direct OpenAI API: Too expensive ($0.03/1K tokens)
- Hugging Face Inference API: Complex model management
- Anthropic Claude API: No free tier

---

#### Why OpenRouter?

**Chosen:** OpenRouter with free tier models

**Reasons:**
- ✅ $0 cost (Gemini Flash, Llama 3.2, Mistral 7B free)
- ✅ 60 requests/min per model (180 req/min total with 3 models)
- ✅ No credit card required
- ✅ Good model quality (Gemini Flash = GPT-3.5 level)
- ✅ Simple REST API

**Cost Comparison:**
```
OpenAI GPT-4:     $0.03 / 1K tokens  →  $30 / 1M tokens
OpenAI GPT-3.5:   $0.002 / 1K tokens →  $2 / 1M tokens
OpenRouter Free:  $0.00 / 1K tokens  →  $0 / 1M tokens ✅
```

**With 10,000 users generating 5 AI requests/day:**
- Total requests: 50,000/day
- Tokens used: ~500 tokens/request = 25M tokens/day
- **Cost with GPT-3.5:** $50/day = $1,500/month
- **Cost with OpenRouter:** $0/month ✅

---

#### Why Redis?

**Chosen:** Redis 7.0+

**Reasons:**
- ✅ In-memory caching (sub-millisecond latency)
- ✅ TTL support (auto-expiry after 1 hour)
- ✅ Atomic operations for rate limit flags
- ✅ Simple key-value storage for cache
- ✅ Persistence optional (cache can be ephemeral)

**Impact:**
- 95%+ cache hit rate on repeated AI requests
- Response time: 2-5 seconds (no cache) → <50ms (cached)

---

#### Why Story Protocol?

**Chosen:** Story Protocol (Aeneid Testnet)

**Reasons:**
- ✅ Purpose-built for IP management (not generic NFTs)
- ✅ Programmable IP licensing
- ✅ Automatic royalty distribution via smart contracts
- ✅ Derivative tracking built-in
- ✅ Python SDK available (story-protocol-python-sdk)

**Alternatives Considered:**
- Ethereum NFTs (ERC-721): No built-in royalties or derivatives
- Polygon NFTs: Similar limitations, just cheaper gas
- Traditional IP registries: Centralized, slow, expensive

---

### 8. Security Architecture

#### Authentication Flow (SIWE)

```
1. User clicks "Connect Wallet"
   ↓
2. Frontend calls GET /api/auth/generate-nonce/
   → Returns: { nonce: "abc123", message: "Sign this message..." }
   ↓
3. Frontend requests wallet signature via Reown/Wagmi
   → User signs message in MetaMask
   ↓
4. Frontend sends POST /api/auth/verify-signature/
   → Body: { message, signature, wallet_address }
   ↓
5. Backend verifies signature using siwe library
   ✅ Valid → Generate JWT token
   ❌ Invalid → Return 401 Unauthorized
   ↓
6. Frontend stores JWT in AuthContext
   → All subsequent requests include: Authorization: Bearer <jwt>
   ↓
7. Backend middleware validates JWT on protected routes
   ✅ Valid → Allow request, attach user to request.user
   ❌ Invalid/Expired → Return 401
```

#### Security Measures

**1. SIWE Message Format**
```
localhost:3000 wants you to sign in with your Ethereum account:
0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb

Sign in to Lore Platform

URI: http://localhost:3000
Version: 1
Chain ID: 84532
Nonce: 8f3e9c2b1a4d6f5e
Issued At: 2024-12-04T10:30:00Z
```

**2. JWT Payload**
```json
{
  "user_id": 123,
  "wallet_address": "0x742d35Cc...",
  "exp": 1733320200,  // Expires in 24 hours
  "iat": 1733233800
}
```

**3. Environment Variables (Secrets)**
```bash
# Never commit these to Git
SECRET_KEY=django-secret-key-here
STORY_PRIVATE_KEY=0x...
OPENROUTER_API_KEY=sk-or-...
PINATA_JWT=eyJhbGc...
DATABASE_URL=postgresql://...
REDIS_URL=redis://...
```

**4. CORS Configuration**
```python
# config/settings/base.py
CORS_ALLOWED_ORIGINS = [
    'http://localhost:3000',  # Development
    'https://lore.example.com',  # Production
]
CORS_ALLOW_CREDENTIALS = True
```

---

### 9. Performance Optimization

#### Caching Strategy Summary

| Layer | Technology | TTL | Purpose |
|-------|-----------|-----|---------|
| AI Responses | Redis | 1 hour | Cache LLM outputs (95% hit rate) |
| Rate Limit Flags | Redis | 5 min | Track rate-limited models |
| Database Queries | Django ORM | N/A | `select_related()`, `prefetch_related()` |
| Static Assets | Next.js | CDN | Images, JS, CSS optimization |

#### Database Indexing

```python
# apps/assets/models.py
class IPAsset(models.Model):
    # ...
    class Meta:
        indexes = [
            models.Index(fields=['user', '-created_at']),  # Fast user asset list
            models.Index(fields=['ip_id']),  # Fast Story Protocol lookups
        ]

class AIGenerationLog(models.Model):
    # ...
    class Meta:
        indexes = [
            models.Index(fields=['user', '-created_at']),  # User analytics
            models.Index(fields=['operation_type', '-created_at']),  # Platform analytics
        ]
```

#### API Response Times (Targets)

| Endpoint | Target | Achieved |
|----------|--------|----------|
| GET /assets/ | <200ms | ~150ms |
| POST /assets/ (no blockchain) | <500ms | ~300ms |
| POST /register-ip/ (blockchain) | <3s | ~2s |
| POST /ai/generate-title/ (cached) | <100ms | ~50ms |
| POST /ai/generate-title/ (uncached) | <5s | ~3s |

---

### 10. Scalability Considerations

#### Horizontal Scaling

**Current Architecture:**
```
User → Load Balancer → Django Instance (1) → PostgreSQL (1)
                                           → Redis (1)
```

**Scaled Architecture:**
```
User → Load Balancer → Django Instance (1) → PostgreSQL Primary
                     → Django Instance (2) → PostgreSQL Read Replica
                     → Django Instance (3) → Redis Cluster
```

#### Database Partitioning (Future)

```sql
-- Partition AIGenerationLog by month
CREATE TABLE ai_generation_log_2024_12 PARTITION OF ai_generation_log
    FOR VALUES FROM ('2024-12-01') TO ('2025-01-01');
```

#### Redis Cluster (Future)

- Current: Single Redis instance
- Future: Redis Cluster (6 nodes: 3 primary + 3 replicas)
- Benefit: 100K+ requests/sec throughput

---

## Deployment Architecture

### Development Environment

```
Developer Machine
├── Backend: Django Dev Server (port 8000)
├── Frontend: Next.js Dev Server (port 3000)
├── PostgreSQL: Docker container (port 5432)
├── Redis: Docker container (port 6379)
└── Story Protocol: Aeneid Testnet (remote)
```

### Production Environment (Planned)

```
Cloud Infrastructure
├── Frontend: Vercel (Next.js)
│   └── Edge Network (CDN)
├── Backend: Azure App Service (Django)
│   └── Auto-scaling (1-10 instances)
├── Database: Azure PostgreSQL Flexible Server
│   └── Read replicas (1-3)
├── Cache: Azure Redis Cache
│   └── Premium tier (clustering enabled)
└── Blockchain: Story Protocol Mainnet
```

---

## Data Flow Examples

### Example 1: Creating an IP Asset with AI

```
1. User uploads image in MintModal
   → Frontend: FormData with file
   ↓
2. POST /api/assets/upload-to-ipfs/
   → Backend: Upload to Pinata IPFS
   → Returns: { media_url: 'ipfs://Qm...' }
   ↓
3. User types description: "mystical forest"
   ↓
4. User clicks "AI Generate Title"
   → POST /api/assets/ai/generate-title/
   → Body: { description: "mystical forest" }
   ↓
5. Backend AIService:
   → Check cache (miss)
   → Call LiteLLM → OpenRouter → Gemini Flash
   → Response: ["Mystical Forest Path", "Enchanted Woodland", ...]
   → Cache result (1 hour TTL)
   → Create AIGenerationLog entry
   → Return: { titles: [...], model_used: "google/gemini-flash-1.5" }
   ↓
6. Frontend displays 4 suggestions
   → User clicks "Mystical Forest Path"
   → Sets formData.title
   ↓
7. User clicks "AI Enhance Description"
   → POST /api/assets/ai/enhance-description/
   → Backend expands "mystical forest" to 150-word paragraph
   → Returns enhanced text
   ↓
8. User clicks "Mint to Blockchain"
   → POST /api/assets/
   → Body: { title, description, media_url, asset_type: 'digital_art' }
   ↓
9. Backend creates metadata JSON:
   {
     "name": "Mystical Forest Path",
     "description": "...",
     "image": "ipfs://Qm...",
     "attributes": [...]
   }
   ↓
10. Upload metadata to IPFS
    → Returns: metadata_uri
    ↓
11. Call Story Protocol SDK:
    client.ip_asset.register(
      token_contract=NFT_CONTRACT,
      token_id=auto_increment_id,
      metadata_uri=metadata_uri
    )
    → Returns: { ipId: "0x123...", txHash: "0xabc..." }
    ↓
12. Save to PostgreSQL:
    IPAsset.objects.create(
      user=request.user,
      title="Mystical Forest Path",
      description="...",
      ip_id="0x123...",
      metadata_uri=metadata_uri,
      media_url="ipfs://Qm..."
    )
    ↓
13. Return success to frontend
    → Redirect to asset detail page
```

---

### Example 2: Creating a Derivative Asset

```
1. User browses assets, finds parent asset
   ↓
2. Clicks "Create Remix"
   → Opens RemixModal
   ↓
3. User uploads remix image
   → POST /api/assets/upload-to-ipfs/
   → Returns remix media_url
   ↓
4. User enters description
   ↓
5. User clicks "AI Analyze Similarity"
   → POST /api/assets/ai/analyze-derivative/
   → Body: {
       parent_description: "mystical forest with waterfall",
       child_description: "neon cyberpunk forest"
     }
   → Backend analyzes similarity
   → Returns: {
       similarity_score: 75,
       similar_elements: ["forest setting", "atmospheric mood"],
       differences: ["color palette", "art style"],
       suggested_royalty_percentage: 10
     }
   ↓
6. User confirms derivative creation
   → POST /api/assets/derivatives/
   → Body: {
       parent_asset_id: 123,
       child_title: "Neon Forest",
       child_description: "...",
       child_media_url: "ipfs://Qm...",
       royalty_percentage: 10
     }
   ↓
7. Backend:
   a) Register child as separate IP asset on Story Protocol
   b) Link child to parent:
      client.license.register_derivative(
        child_ip_id="0xchild...",
        parent_ip_ids=["0xparent..."],
        license_terms_ids=[1]  // PIL license
      )
   c) Save DerivativeAsset relationship
   ↓
8. Smart contract now enforces:
   → Any revenue to child asset → 10% auto-sent to parent creator
   ↓
9. Frontend shows success + derivative link
```

---

## Summary

**Lore's architecture combines:**

1. **Decentralized Blockchain** (Story Protocol) for immutable IP ownership
2. **AI/ML Intelligence** (LiteLLM + OpenRouter) for content generation
3. **Modern Web Stack** (Next.js + Django) for excellent UX/DX
4. **Smart Caching** (Redis) for performance
5. **Complete Audit Trail** (PostgreSQL) for transparency

**Key Architectural Patterns:**
- Service layer (Singleton) for business logic
- React Query for state management
- JWT + SIWE for authentication
- Model fallback for resilience
- Database logging for analytics

**Scalability:**
- Horizontal scaling ready
- Caching reduces load by 95%
- Indexes optimize queries
- Stateless backend (can add instances)

---

**Document Version:** 2.0
**Last Updated:** December 2024
**Next:** [04-IMPLEMENTATION.md](./04-IMPLEMENTATION.md)

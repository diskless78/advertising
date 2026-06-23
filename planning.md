# 📢 AI-Powered Advertising Content Publishing System
## Strategic Planning & Development Organization

> **Scope of this document:** Technology choices, AI model strategy, Vietnamese language handling,
> and code architecture patterns. No implementation code — planning and organization only.

---

## 1. System Overview

This system automates the full lifecycle of advertising content:

```
Employee composes draft
        ↓
AI/LLM Engine refines content (text + image + video)
        ↓
Admin reviews and approves
        ↓
Channel Engine publishes to TikTok / Facebook / YouTube / Website
        ↓
Engagement Collector reads likes, views, shares, subscriptions
        ↓
Analytics Engine analyses performance + suggests improvements
        ↓
Dashboard shows results to employee and business owner
```

All content is composed and published **in Vietnamese**.

---

## 2. Language & Technology Choices

### 2.1 Frontend — Employee Content Composer & Admin Dashboard

**Recommended: React + TypeScript (with Next.js framework)**

| Option | Pros | Cons |
|--------|------|------|
| **React + Next.js** ✅ | Industry standard; huge ecosystem; great for rich editors; excellent Vietnamese text handling (Unicode); SSR for SEO on website | Requires JavaScript/TypeScript knowledge |
| Vue.js | Easier learning curve; also excellent | Smaller ecosystem than React |
| Angular | Very structured; good for large teams | More complex; heavier setup |
| Plain HTML + JS | Simple to start | Hard to scale; no component reuse |

**Why React + Next.js is the right choice here:**
- The article composer needs a **rich text editor** (like TipTap or Quill) — these are best supported in React
- The admin review screen and dashboard need **real-time updates** — React handles this naturally
- Vietnamese Unicode is fully supported without any extra work
- Next.js gives you both the **admin web app** and the **public website** in one codebase if needed

> **Key UI libraries to use:**
> - **TipTap** — Rich text editor for composing Vietnamese articles (supports all Unicode, images inline)
> - **React Query** — For fetching and caching data from the backend
> - **Recharts or Chart.js** — For analytics dashboard charts
> - **Shadcn/ui or Ant Design** — UI component library for admin panels

---

### 2.2 Backend — API, Business Logic & Publishing Engine

**Recommended: Python (FastAPI)**

| Option | Pros | Cons |
|--------|------|------|
| **Python + FastAPI** ✅ | Best AI/ML ecosystem; async; fast to develop; all social SDKs available | Slightly slower raw speed than Go/C# (not a concern at this scale) |
| Node.js (Express) | Same language as frontend | Weak AI/ML ecosystem; LLM integration is harder |
| C# (.NET) | Strong typing; good for enterprise teams | Poor AI/ML library support; heavier setup |
| Go | Very fast | Immature AI ecosystem; steep learning curve |

**Why Python:**
- Every major AI/LLM library (LangChain, OpenAI, Google Gemini, HuggingFace) is **Python-first**
- Facebook Graph API, TikTok API, YouTube API all have official **Python SDKs**
- Your team can write both the AI logic and the API in the same language
- FastAPI is modern, fast, and generates automatic API documentation

---

### 2.3 Data Layer — Storage & Processing

**Recommended combination:**

| Purpose | Technology | Reason |
|---------|-----------|--------|
| **Primary database** | PostgreSQL | Reliable relational DB; excellent Vietnamese text (UTF-8); supports JSON columns for flexible data |
| **Cache & job queue** | Redis | Fast in-memory store; used for Celery task queuing and session caching |
| **File storage** (images, videos) | AWS S3 or Cloudflare R2 | Store large media files separately from the DB; accessible by all services |
| **Search** (optional, later phase) | Elasticsearch or PostgreSQL Full-Text | Search through articles in Vietnamese |
| **Analytics time-series** | PostgreSQL (with TimescaleDB extension) | Store engagement metrics over time (views, likes per hour) |

> **Vietnamese text in PostgreSQL:** Use `text` columns with `UTF-8` encoding and `vi_VN.utf8` locale or `ICU` collation. PostgreSQL handles Vietnamese characters natively with no special setup.

---

### 2.4 Background Task Processing

**Recommended: Celery + Redis**

Publishing to social channels, collecting metrics, and running analytics are **long-running background jobs** — they must not block the web API. Celery with Redis as the broker handles this cleanly:

- When admin approves → FastAPI sends a **Celery task** → Celery worker publishes in the background
- Engagement collection runs on a **scheduled Celery beat** every 30 minutes
- Analytics re-computation runs **nightly via Celery beat**

---

## 3. AI Model Strategy — Train or Use Existing?

### 3.1 Should you train your own model?

**Answer: No — do not train your own model. Use existing pre-trained models via API.**

| Approach | Cost | Time | Quality | Recommendation |
|----------|------|------|---------|----------------|
| Train from scratch | Extremely high (millions USD, months) | Months-years | Only if you have massive proprietary data | ❌ Not suitable |
| Fine-tune an existing model | High (thousands USD); requires labeled data | Weeks | Good for very specific domain | ⚠️ Only if needed later |
| **Use pre-trained model via API** ✅ | Pay-per-use (cents per article) | Zero setup time | Excellent for Vietnamese content | ✅ Recommended |

**Conclusion:** Use a pre-trained LLM via API. You pay only for what you use, get state-of-the-art quality, and can start immediately. Fine-tuning can be considered later (Phase 2 or 3) only if you find the default models don't match your brand voice well.

---

### 3.2 Which AI Models to Use (Vietnamese Support)

#### For Text Generation (Article Composition & Suggestions)

| Model | Provider | Vietnamese Quality | Cost | Notes |
|-------|----------|--------------------|------|-------|
| **GPT-4o** ✅ | OpenAI | ⭐⭐⭐⭐⭐ Excellent | ~$0.005/1K tokens | Best overall Vietnamese; recommended primary |
| **Gemini 1.5 Pro** ✅ | Google | ⭐⭐⭐⭐⭐ Excellent | ~$0.0035/1K tokens | Cheaper; great for long articles; good fallback |
| **Claude 3.5 Sonnet** | Anthropic | ⭐⭐⭐⭐ Very good | ~$0.003/1K tokens | Strong at following editorial tone/style |
| Llama 3.1 (self-hosted) | Meta (open source) | ⭐⭐⭐ Good | Free (server cost) | If you need data privacy; requires GPU server |
| **PhoGPT / VinaLLaMA** | Vietnamese open source | ⭐⭐⭐⭐ Native Vietnamese | Free (self-hosted) | Trained specifically on Vietnamese text; good for localization |

> **Recommendation:** Use **GPT-4o as the primary model** for content generation and AI suggestions.
> Use **Gemini 1.5 Pro as a secondary/fallback** (cheaper for bulk operations).
> Consider **PhoGPT or VinaLLaMA** if you want a fully Vietnamese-native model hosted on your own server (better data privacy, lower long-term cost).

#### For Image Generation (Advertising Visuals)

| Model | Provider | Notes |
|-------|----------|----|
| **DALL·E 3** ✅ | OpenAI | Best prompt-following; integrates with GPT-4o |
| **Stable Diffusion XL** | Stability AI / self-hosted | Free if self-hosted; more control |
| **Midjourney** | Midjourney | Best artistic quality; API access is limited |
| **Gemini Imagen** | Google | Good quality; native Google integration |

> **Recommendation:** Start with **DALL·E 3** (simplest integration since you're already using OpenAI). Move to self-hosted Stable Diffusion later if cost becomes a concern.

#### For Video Processing (Auto-generate video content)

| Tool | Purpose |
|------|---------|
| **FFmpeg** | Video trimming, format conversion, thumbnail extraction — free, open source |
| **MoviePy** (Python) | Programmatic video editing, text overlays — free |
| **Runway ML** (API) | AI video generation from text — paid, advanced |
| **HeyGen** (API) | AI avatar video generation — great for Vietnamese spokesperson videos |

> **Recommendation for now:** Use FFmpeg + MoviePy for basic video processing. Runway ML or HeyGen can be added in a later phase for advanced AI video generation.

---

### 3.3 Vietnamese Language Considerations

Since **all content is in Vietnamese**, here is what to be careful about:

| Concern | Solution |
|---------|----------|
| **Tonal diacritics** (ắ, ổ, ề, etc.) | Use UTF-8 encoding everywhere — all recommended databases and frameworks fully support this |
| **LLM quality in Vietnamese** | GPT-4o and Gemini 1.5 Pro both perform excellently in Vietnamese without any special setup |
| **Social media platform encoding** | All platforms (Facebook, TikTok, YouTube) support Vietnamese Unicode natively |
| **SEO for Vietnamese articles on website** | Use proper `lang="vi"` HTML attribute; Vietnamese keywords in meta descriptions |
| **AI prompt language** | Write your LLM system prompts in Vietnamese for best results; instruct the model to respond in Vietnamese |
| **Sorting & search** | PostgreSQL with `vi_VN` locale correctly sorts Vietnamese alphabet order |

---

## 4. Code Architecture Pattern

### 4.1 Recommended Pattern: Clean Architecture (Layered)

For a system of this size and complexity, **Clean Architecture** (also called Layered Architecture or Hexagonal Architecture) is the best fit.

```
┌─────────────────────────────────────────────────────────┐
│                    Presentation Layer                    │
│          (React frontend, FastAPI REST endpoints)        │
├─────────────────────────────────────────────────────────┤
│                    Application Layer                     │
│        (Use Cases / Services — business workflows)       │
├─────────────────────────────────────────────────────────┤
│                     Domain Layer                         │
│     (Core business rules, entities, value objects)       │
├─────────────────────────────────────────────────────────┤
│                  Infrastructure Layer                    │
│  (Database, Social APIs, LLM APIs, File Storage, Queue)  │
└─────────────────────────────────────────────────────────┘
```

**What this means in practice:**
- The **Domain Layer** holds your core business concepts: `Article`, `Draft`, `PublishResult`, `EngagementMetric`, `Suggestion` — with no dependency on FastAPI, databases, or any external service
- The **Application Layer** holds the workflows: `ComposeArticleUseCase`, `ApproveAndPublishUseCase`, `CollectEngagementUseCase`, `GenerateSuggestionsUseCase`
- The **Infrastructure Layer** holds the "adapters" that talk to the outside world: `OpenAIAdapter`, `FacebookPublisher`, `PostgreSQLRepository`, `S3FileStorage`
- The **Presentation Layer** holds the FastAPI route handlers and the React components

**Key benefit:** You can swap out any infrastructure (e.g., replace OpenAI with Gemini, replace PostgreSQL with MongoDB) without changing your business logic.

---

### 4.2 Repository Pattern (for Data Access)

Never let your business logic talk directly to the database. Define **Repository interfaces** in the domain layer, and implement them in the infrastructure layer.

```
Domain defines:        ArticleRepository (interface)
Infrastructure has:    PostgreSQLArticleRepository (concrete implementation)
```

This makes testing easy (you can use a fake in-memory repository in tests) and keeps your logic clean.

---

### 4.3 Event-Driven Pattern (for Background Jobs)

When the admin clicks "Approve", the system should:
1. Update the draft status in the database
2. **Emit an event** (e.g., `ArticleApproved`)
3. Celery workers **listen for that event** and start publishing to each channel independently

This means if TikTok fails but Facebook succeeds, the system handles each channel separately — no all-or-nothing publishing.

---

### 4.4 Adapter Pattern (for Social Channel Publishers)

Each social channel (Facebook, TikTok, YouTube, Website) is a different **Adapter** implementing the same `ChannelPublisher` interface:

```
ChannelPublisher (interface)
  ├── FacebookAdapter
  ├── TikTokAdapter
  ├── YouTubeAdapter
  └── WebsiteAdapter
```

Adding a new channel (e.g., Instagram) in the future requires only adding a new adapter — no changes to the core publishing workflow.

---

### 4.5 Strategy Pattern (for AI Model Switching)

The LLM model used for content generation should be switchable without code changes:

```
ContentGeneratorStrategy (interface)
  ├── GPT4oStrategy
  ├── GeminiStrategy
  └── PhoGPTStrategy
```

This allows you to:
- Switch models based on cost or availability
- A/B test which model produces better Vietnamese content
- Fall back to a secondary model if the primary API is down

---

## 5. Project Folder Structure (High Level)

```
advertising-system/
│
├── frontend/                        ← React + Next.js (TypeScript)
│   ├── app/                         ← Next.js pages (App Router)
│   │   ├── compose/                 ← Employee article composer
│   │   ├── admin/                   ← Admin review & approval
│   │   └── dashboard/               ← Analytics & performance dashboard
│   ├── components/                  ← Reusable UI components
│   └── lib/                         ← API client, utilities
│
├── backend/                         ← Python + FastAPI
│   ├── domain/                      ← Core entities & interfaces (NO external deps)
│   │   ├── entities/                ← Article, Draft, User, Metric, Suggestion
│   │   ├── repositories/            ← Abstract repository interfaces
│   │   └── services/                ← Domain service interfaces
│   │
│   ├── application/                 ← Use cases / business workflows
│   │   ├── compose_article/         ← Workflow: Employee submits → AI generates
│   │   ├── review_article/          ← Workflow: Admin approves or rejects
│   │   ├── publish_article/         ← Workflow: Post to all channels
│   │   ├── collect_engagement/      ← Workflow: Pull metrics from APIs
│   │   └── analyse_performance/     ← Workflow: Analyse + generate suggestions
│   │
│   ├── infrastructure/              ← Concrete implementations
│   │   ├── db/                      ← PostgreSQL repositories (SQLAlchemy)
│   │   ├── ai/                      ← OpenAI, Gemini, PhoGPT adapters
│   │   ├── channels/                ← Facebook, TikTok, YouTube, Website adapters
│   │   ├── storage/                 ← S3 / Cloudflare R2 file storage
│   │   └── queue/                   ← Celery tasks & beat scheduler
│   │
│   └── presentation/                ← FastAPI routes & request/response schemas
│       ├── api/
│       │   ├── content_routes.py
│       │   ├── admin_routes.py
│       │   ├── publish_routes.py
│       │   └── analytics_routes.py
│       └── schemas/                 ← Pydantic request/response models
│
└── infrastructure/                  ← DevOps
    ├── docker-compose.yml           ← Local development setup
    ├── nginx/                       ← Reverse proxy config
    └── .env.example                 ← Environment variables template
```

---

## 6. Summary of All Technology Decisions

| Concern | Decision | Reasoning |
|---------|----------|-----------|
| **Frontend language** | TypeScript (React + Next.js) | Best rich-text editor ecosystem; Vietnamese Unicode works natively |
| **Backend language** | Python (FastAPI) | First-class AI/ML support; all social SDKs available; fast development |
| **Database** | PostgreSQL | Native UTF-8 Vietnamese support; reliable; flexible JSON columns |
| **Cache / Queue** | Redis + Celery | Async background jobs for publishing and metric collection |
| **File storage** | AWS S3 / Cloudflare R2 | Scalable storage for images and videos |
| **LLM for Vietnamese text** | GPT-4o (primary) + Gemini 1.5 Pro (secondary) | Both excellent at Vietnamese; pay-per-use; no training required |
| **Train your own model?** | ❌ No — use pre-trained APIs | Training is extremely expensive and unnecessary at this stage |
| **Image generation** | DALL·E 3 (via OpenAI API) | Simple integration; high quality |
| **Video processing** | FFmpeg + MoviePy | Free; handles all standard formats |
| **Architecture pattern** | Clean Architecture (Layered) | Clear separation of concerns; easy to test; swappable components |
| **Data access pattern** | Repository Pattern | Decouples business logic from DB implementation |
| **Publishing pattern** | Adapter Pattern | Each social channel is independent; easy to add new channels |
| **Model switching** | Strategy Pattern | Switch between GPT-4o / Gemini / PhoGPT without code changes |
| **Background jobs** | Event-Driven + Celery | Approval triggers independent publish jobs per channel |

---

## 7. Development Phases (Planning Only)

| Phase | Focus | Output |
|-------|-------|--------|
| **Phase 0** | Set up project structure, environment, DB schema, auth | Working skeleton with login |
| **Phase 1** | Content Composer UI + AI content generation (Module 1) | Employee can compose and get AI-refined Vietnamese article |
| **Phase 2** | Admin Review Module (Module 2) | Admin can view, edit, approve or reject drafts |
| **Phase 3** | Channel Publishing — Facebook & YouTube first (Module 3) | Approved articles auto-posted to Facebook and YouTube |
| **Phase 4** | TikTok & Website publishing (Module 3 continued) | All 4 channels live |
| **Phase 5** | Engagement Collection Engine (Module 4) | Likes, views, shares collected every 30 minutes |
| **Phase 6** | Analytics + AI Suggestions Engine (Module 5) | Performance data + Vietnamese improvement suggestions |
| **Phase 7** | Dashboard UI (Module 5 frontend) | Visual charts and suggestions shown to employees |
| **Phase 8** | Testing, hardening, deployment | Production-ready system |

---

## 8. Pre-Development Checklist

Before writing any code, these accounts and credentials must be in place:

- [ ] **OpenAI API key** — for GPT-4o content generation and DALL·E 3 images
- [ ] **Google Cloud project** — for Gemini API + YouTube Data API v3 + YouTube Analytics API
- [ ] **Facebook Developer App** — with `pages_manage_posts` and `pages_read_engagement` permissions
- [ ] **TikTok Developer App** — registered at developers.tiktok.com with Content Posting API access
- [ ] **AWS account or Cloudflare R2** — for image and video file storage
- [ ] **Domain + hosting** — for the company website CMS API endpoint
- [ ] **PostgreSQL server** — local for dev; managed (RDS or Supabase) for production

---

*Last updated: 2026-06-23*
*Document type: Strategic Planning — No implementation code*

# AI Operations Manager — Build Plan

**Project**: AI Assistant Web App for Local Service Businesses
**Framework**: Next.js 14 (App Router) + TypeScript
**Duration**: 3 months (~12 weeks)
**AI Layer**: OpenRouter (unified API for GPT-4o + Claude 3.5 Sonnet)
**Version**: 1.0 (MVP)

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Tech Stack](#tech-stack)
3. [Architecture Decisions](#architecture-decisions)
4. [Phase Roadmap](#phase-roadmap)
5. [Database Schema](#database-schema)
6. [Environment Variables](#environment-variables)
7. [API Costs](#api-costs)
8. [Open Questions Resolved](#open-questions-resolved)

---

## Project Overview

Build an **AI Operations Manager** web application for local service businesses (HVAC, plumbing, dental, etc.). The app acts as an "AI Chief of Staff" that monitors business inboxes, CRMs, and dashboards to surface urgent items and execute follow-ups autonomously.

### Core Value Proposition
- Scan email inboxes for urgent items
- Classify emails by category (sales, support, recruiting, marketing)
- Score urgency (1-100)
- Follow up with stale CRM leads
- Provide dashboards for sales, recruiting, support, marketing
- AI assistant chat with human-in-the-loop approval
- Automate repetitive tasks to free up 15-20 hours/week for busy founders

### Pricing Model
- **Starter**: $299–$499/mo (email triage + weekly stale-lead follow-ups + 1 dashboard)
- **Growth**: $799–$1,200/mo (multi-dashboard + basic "fix it" automations)
- **Done-For-You**: $2,000–$3,500/mo (full AI ops manager + weekly summary call)
- **Paid Pilot**: $500 for 30 days, then roll into monthly
- **Setup Fee**: $500–$1,500 for onboarding/integrations

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 14 (App Router) |
| Language | TypeScript (strict mode) |
| Styling | Tailwind CSS + shadcn/ui components |
| Database | PostgreSQL (Supabase/Neon) + Prisma ORM |
| Vector Search | pgvector extension |
| Auth | NextAuth.js (Auth.js) — Google OAuth |
| AI Gateway | OpenRouter (GPT-4o + Claude 3.5 Sonnet) |
| AI Streaming | Vercel AI SDK |
| Background Jobs | Inngest |
| Queue/Cache | Upstash Redis |
| Email Receiving | Gmail API |
| Email Sending | Resend |
| SMS Notifications | Twilio |
| CRM | HubSpot API |
| Deployment | Vercel |

---

## Architecture Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Vector DB | **pgvector** (Supabase/Neon) | Simplest setup, zero new infrastructure, sufficient for MVP scale (<10K vectors), already on PostgreSQL. Migrate to dedicated vector DB at 50+ users. |
| Background Jobs | **Inngest** | Best developer experience for local dev, reliable cron-based email scanning every 15 min |
| Email Sending | **Resend** | Clean API, good deliverability, free tier (3,000 emails/mo) |
| Notifications | **SMS + email/app push** | Both channels for maximum reach when urgent alerts fire |
| Design | **Light + dark mode with toggle** | Caters to all user preferences, professional feel |
| AI Provider | **OpenRouter** | Unified API gateway for GPT-4o (reasoning/chat) + Claude 3.5 Sonnet (classification). Single API key, easy model swapping. |
| CRM | **HubSpot only** | Most common for local businesses, extensive API, good documentation |
| Platform | **Web app first, PWA-capable** | Mobile-responsive from day one, installable as PWA. Native app only at 50+ customers. |

---

## Phase Roadmap

### Phase 1 — Foundation (Weeks 1–3)

| Week | Task | Details | Deliverable |
|------|------|---------|-------------|
| 1 | Project scaffold & tooling | `npx create-next-app@latest`, TypeScript strict mode, Tailwind + shadcn/ui, ESLint, Prettier, Husky + lint-staged | Working dev environment |
| 1 | Database & ORM | Prisma + PostgreSQL (Supabase/Neon), pgvector extension, initial schema (User, Email, Lead, Action, AuditLog) | Database connected, Prisma migrations working |
| 2 | Auth | NextAuth.js with Google OAuth — required for Gmail API access. Session management, role-based access (Owner/Team) | Working auth flow with Google sign-in |
| 2 | Environment config | `.env` structure, Zod validation for env vars, runtime config | Validated env configuration |
| 3 | OpenRouter integration | Single API layer via OpenRouter SDK. Model routing: GPT-4o for reasoning/chat, Claude 3.5 Sonnet for email classification. Fallback handling, rate limiting | OpenRouter API tested and working |
| 3 | Background job queue | Inngest for scheduled tasks (email scanning every 15 min). Upstash Redis for queue management | Background jobs running |

**Phase 1 Deliverable**: Working auth flow, database connected, OpenRouter API tested, background jobs running.

---

### Phase 2 — Core Engine: Email Scanning (Weeks 4–5)

| Week | Task | Details | Deliverable |
|------|------|---------|-------------|
| 4 | Gmail API integration | OAuth flow for Gmail (`gmail.readonly`, `gmail.send`, `gmail.modify`). Fetch email history via `historyId`. Store emails in PostgreSQL with pgvector embeddings for semantic search. Resend for email delivery | Gmail API connected, emails stored with embeddings |
| 4 | Email classification pipeline | Background job fetches new emails → Claude 3.5 classifies (Urgent/Normal/Spam/Newsletter) → Extracts entities (customer name, phone, intent) → Urgency score (1–100) → Stores with category tag | Automated email classification |
| 5 | Real-time dashboard updates | Supabase Realtime for live updates when urgent emails arrive. Toast notifications: "🚨 Urgent: Emergency repair request from [Customer]" | Live notifications working |
| 5 | Command Center (Home Dashboard) | "Urgency Inbox": AI-scored list of unread emails requiring action today. "Stale Leads" preview. "At-Risk Customers" preview. | Command Center with urgency inbox |

**Phase 2 Deliverable**: Live email scanning pipeline, classified emails in DB, Command Center dashboard.

---

### Phase 3 — Dashboard Modules (Weeks 6–8)

| Module | Week | Features |
|--------|------|----------|
| **Sales Dashboard** | 6 | Pipeline velocity metrics, AI-identified hot leads (intent keywords, pricing mentions), stale lead detection (7+ days no touch), one-click "Draft Reply" + "Schedule Follow-up". HubSpot sync (two-way: deals, contacts, last-contacted timestamps). |
| **Recruiting Dashboard** | 7 | Candidate email detection in inbox, stale candidate alerts (14+ days), auto-draft interview scheduling emails, open roles vs. pipeline volume tracking. |
| **Support/Operations** | 7 | Unresolved complaint detection via sentiment analysis, missed appointment detection in email threads, vendor invoice reminders. |
| **Marketing Dashboard** | 8 | Campaign response tracking, lead source attribution, social media engagement summary (if connected). |
| **Analytics Overview** | 8 | Weekly AI-generated summary: "You missed 3 high-value leads this week. 12 support emails went unanswered 24+ hours." Time saved metric: "AI handled 47 email classifications, saving you ~6 hours." |

**Phase 3 Deliverable**: All dashboard modules functional with real data from email/Gmail/HubSpot.

---

### Phase 4 — AI Assistant Interface (Weeks 9–10)

| Week | Task | Details | Deliverable |
|------|------|---------|-------------|
| 9 | Chat interface | Right-sidebar modal with streaming responses via Vercel AI SDK. Context-aware: "Why did you flag this email as urgent?" Based on email content with citations. | AI chat interface working |
| 9 | Approval Queue | All AI-drafted emails/actions go to an "Approve & Send" table. Approve/Reject buttons. Audit trail logged per action. | Approval workflow functional |
| 10 | Auto-Pilot mode | Opt-in per category. AI sends follow-ups to stale leads automatically if confidence >90%. Safety mechanisms: daily send cap, business hours restriction, human override always available. | Auto-pilot with safety guards |
| 10 | Learning loop | Thumbs up/down on AI suggestions. Feedback stored → retrain/fine-tune classification thresholds over time. | Feedback loop implemented |

**Phase 4 Deliverable**: Fully functional AI chat assistant, approval workflow, auto-pilot with safety guards.

---

### Phase 5 — Polish, Deploy, Iterate (Weeks 11–12)

| Week | Task | Details | Deliverable |
|------|------|---------|-------------|
| 11 | PWA setup | `manifest.json`, service worker, "Add to Home Screen" prompt. Mobile-responsive testing (all dashboards + chat). | Installable PWA |
| 11 | Settings & configuration | Auto-pilot toggles per category, notification preferences (email/SMS), connected accounts management (Gmail, HubSpot). | Settings page complete |
| 11 | Design polish | Light mode default with dark mode toggle. Professional dark mode optional, clean white/light gray default. Inter font, 14px base, dense but readable. | Light/dark mode toggle |
| 12 | Testing & hardening | Error handling, edge cases (empty inbox, API failures), rate limit handling. Load testing background jobs. SMS notification testing. | Tested and hardened |
| 12 | Deployment | Vercel for hosting, Supabase/Neon for DB, Upstash for Redis. Domain setup, SSL, CI/CD pipeline. | Live production app |
| 12 | Beta launch | Invite 5–10 local business owners. Paid pilot ($500 for 30 days). Collect feedback → iterate. | Beta customers onboarded |

---

## Database Schema

```prisma
model User {
  id           String   @id @default(cuid())
  email        String   @unique
  businessName String
  googleTokens String?  // Encrypted refresh token
  hubspotToken String?  // Encrypted
  settings     Json     // Auto-pilot preferences, notification prefs
  emails       Email[]
  leads        Lead[]
  actions      Action[]
  auditLogs    AuditLog[]
  createdAt    DateTime @default(now())
}

model Email {
  id            String   @id @default(cuid())
  userId        String
  gmailId       String   @unique
  from          String
  to            String?
  subject       String
  body          String   @db.Text
  urgencyScore  Int      // 1–100
  category      String   // sales, support, recruiting, marketing, spam, newsletter
  intent        String?  // quote_request, complaint, billing, interview, etc.
  aiSummary     String?
  status        String   @default("pending") // pending, approved, sent, archived
  createdAt     DateTime @default(now())
  user          User     @relation(fields: [userId], references: [id])
}

model Lead {
  id            String   @id @default(cuid())
  userId        String
  email         String
  name          String
  source        String   // gmail, hubspot
  hubspotId     String?
  lastContact   DateTime
  dealValue     Float?
  status        String   @default("stale") // hot, warm, stale, converted
  followUpCount Int      @default(0)
  category      String?  // sales, recruiting
  user          User     @relation(fields: [userId], references: [id])
}

model Action {
  id             String   @id @default(cuid())
  userId         String
  type           String   // send_email, create_task, update_crm
  payload        Json     // Draft email content or task details
  aiConfidence   Float
  status         String   @default("pending_approval") // pending_approval, executed, rejected
  executedAt     DateTime?
  category       String?
  user           User     @relation(fields: [userId], references: [id])
}

model AuditLog {
  id        String   @id @default(cuid())
  userId    String
  action    String   // What happened
  details   Json
  ipAddress String?
  createdAt DateTime @default(now())
  user      User     @relation(fields: [userId], references: [id])
}
```

---

## Environment Variables

```env
DATABASE_URL=
NEXTAUTH_SECRET=
NEXTAUTH_URL=
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
HUBSPOT_ACCESS_TOKEN=
OPENROUTER_API_KEY=
UPSTASH_REDIS_URL=
UPSTASH_REDIS_TOKEN=
RESEND_API_KEY=
TWILIO_ACCOUNT_SID=
TWILIO_AUTH_TOKEN=
TWILIO_PHONE_NUMBER=
```

---

## API Costs

### Estimated Monthly Costs (MVP, ~5-10 beta users)

| Service | Free Tier | Estimated Cost at MVP Scale |
|---------|-----------|----------------------------|
| OpenRouter (GPT-4o + Claude 3.5) | No free tier | ~$15–$50/mo |
| Gmail API | 10K requests/day free | $0 |
| HubSpot API | Free CRM tier | $0 |
| PostgreSQL (Supabase/Neon) | 500MB, 50K rows free | $0 |
| Upstash Redis | 5K commands/day free | $0 |
| Vercel Hosting | 100GB bandwidth free | $0 |
| Resend | 3,000 emails/mo free | $0–$5/mo |
| Twilio SMS | Pay per use (~$0.01/SMS) | Minimal |

**Realistic MVP total: ~$15–$55/month**

### Cost Mitigation
1. **OpenRouter caching** — cache classification results for emails already scanned
2. **Token budgeting** — set max tokens per conversation cap
3. **Batch processing** — classify emails in batches
4. **Use Claude 3.5 Sonnet** for classification (cheaper than GPT-4o), GPT-4o only for chat

---

## UI/UX Specifications

| Aspect | Specification |
|--------|---------------|
| Layout | Collapsible left sidebar (module nav), main content area, right-side AI Assistant panel (collapsible) |
| Color Scheme | Light mode default, dark mode optional with toggle |
| Typography | Inter font, 14px base, dense but readable |
| Key Components | UrgencyBadge (Red/Orange/Green), LeadCard (days since contact, deal value, "Draft Follow-up" button), ApprovalQueueTable (pending AI actions with Approve/Reject), AIChatPanel (streaming with context citations) |
| PWA | Installable as PWA — icon, splash screen, push notifications without App Store |
| Responsive | Fully mobile-responsive — founders on job sites need 5-second access |

---

## Build Order Summary

1. **Foundation**: Auth → Database → Gmail Integration → Basic Dashboard
2. **AI Features**: Email scanning pipeline → Classification → Dashboard modules
3. **AI Assistant**: Chat interface → Approval queue → Auto-pilot
4. **Polish**: PWA → Design → Testing → Deploy → Beta launch

---

## MVP Deliverables Checklist

- [ ] Working auth flow with Google sign-in
- [ ] Email scanning pipeline (Gmail → Classification → Dashboard)
- [ ] Command Center dashboard with urgency inbox
- [ ] AI Assistant chat with approval queue
- [ ] Sales Dashboard with stale lead detection
- [ ] Recruiting Dashboard with candidate alerts
- [ ] Support/Operations Dashboard with complaint detection
- [ ] Analytics Overview with weekly AI summary
- [ ] Auto-Pilot mode with safety guards
- [ ] PWA setup (installable, mobile-responsive)
- [ ] Light/dark mode toggle
- [ ] SMS + email/app push notifications
- [ ] Settings page with auto-pilot toggles
- [ ] Deployment to Vercel with custom domain
- [ ] Paid beta launch with 5–10 local business owners

---

## Notes

- Use mock data for CRM if no HubSpot API key available during development
- Build foundation first: Auth → Database → Gmail Integration → Basic Dashboard. Then layer AI features.
- If using Cursor or Claude Code, scaffold the project structure first
- Phase 1 should produce a working prototype before adding AI features
- Safety first: Auto-pilot is opt-in only with strict caps and human override always available

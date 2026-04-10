# AddRan Advisor Roadmap

## UI Enhancements

### ✅ Completed
- [x] Avatar icons (CSS-based assistant & user avatars)
- [x] Animated typing dots (bouncing dots instead of "Thinking...")
- [x] Suggested prompts (clickable chips for common questions)
- [x] Send button icon (paper plane instead of "Send" text)
- [x] Gradient header (polished purple gradient)
- [x] Chat bubble styling (white assistant, gradient user)
- [x] Bold text and bullet list support
- [x] Smart scroll (keeps question visible when response arrives)
- [x] **TCU/AddRan logo** - SVG wordmark replacing emoji header
- [x] **Dark mode toggle** - CSS custom properties with theme persistence
- [x] **Feedback buttons** - Thumbs up/down on responses, logged to Firestore
- [x] **Fix horizontal drift on mobile** - Overflow containment, word-break, iOS zoom prevention
- [x] **Mobile polish** - Tighter spacing, larger touch targets, responsive font sizes
- [x] **Timestamp badges** - Hover-reveal time indicators on messages
- [x] **Header redesign** - Left-aligned app-bar layout (logo + actions top, title below)
- [x] **Expandable program cards** - Interactive cards when listing majors (backend detection + client rendering)
- [x] **Conversation export** - Save chat as PDF (jsPDF, header/footer, program cards, disclaimer)
- [x] **Markdown heading support** - `###` headings rendered as bold text in chat bubbles
- [x] **Smarter program card detection** - Single-word names (English, History) require nearby keyword context to avoid false positives

### 💡 Future Ideas
- [ ] **Advisor photos** - When suggesting "talk to an advisor"
- [ ] **Voice input** - Microphone button for accessibility
- [ ] **Progress indicator** - Show when fetching program data
- [ ] **Conversation history** - Let returning students pick up where they left off

---

## Program Data

### ✅ Completed
- [x] 60 program detail JSON files covering all 37 AddRan programs
  - 23 original BA/BS programs extracted from Excel spreadsheets
  - 4 language BAs (Chinese, French, German, Italian) under Modern Language Studies
  - 2 additional programs (International Relations Minor, Latin American Studies BA)
  - 20 departmental minors (Anthropology, Chinese, CRES, Creative Writing, Criminology, Economics, English, French, Geography, German, History, International Economics, Italian, Philosophy, Political Science, Religion, Sociology, Spanish, W&GS, Writing & Rhetoric)
  - 8 interdisciplinary minors (African American & Africana Studies, Asian Studies, British Colonial & Post-Colonial Studies, Classical Studies, Human-Animal Relationships, Latinx Studies, Middle East Studies, Urban Studies)
  - 2 military commissions (Aerospace Studies/AFROTC, Military Science/Army ROTC)
  - 1 Writing & Rhetoric BA from Excel extraction
- [x] Extraction pipeline (`scripts/extract_programs.py`) with verification
- [x] Program details wired into chatbot system prompt (~4,800 tokens)
- [x] DCDA dedicated data file with full course listings
- [x] Core Curriculum data (Fall 2025+) with rules and requirements
- [x] English Department hand-curated Spring 2026 highlights
- [x] Support resources CSV (career services, tutoring, etc.)
- [x] W&GS and CRES files updated with June 2026 merger note (factual only)
- [x] AI disclaimer in UI and system prompt (accuracy warning + data freshness)
- [x] Liberal arts value research knowledge base (`la-value-research.json`) — 30 sources across 5 categories (economic ROI, career outcomes, employer demand, job satisfaction, AI era), wired into system prompt with cite-don't-paraphrase instructions

### 🔜 Next Up
- [ ] **Periodic re-extraction** - Update JSONs when Excel files change
- [ ] **Data accuracy audit** - Cross-reference JSON files against source Excel spreadsheets in `other_programs/`

---

## Article Collection & Curation Pipeline

### ✅ Completed
- [x] Firestore `articles` collection schema with status (`approved`/`pending`/`rejected`) and tags
- [x] Admin dashboard at `/admin.html` — add/edit articles, URL auto-fetch, tag checkboxes
- [x] RSS feed monitoring — Fast Company, HBR, NYT (with AI-related keywords)
- [x] OpenAlex academic search — 4 query categories including AI + liberal arts
- [x] Scheduled RSS checking via Cloud Function
- [x] AI-assisted curation — Claude auto-generates summaries and suggests tags on ingestion
- [x] Article dedup by URL (rejected articles preserved as dedup records, no delete)
- [x] **Relevance scoring** - Claude rates article fit (1-10) during ingestion, color-coded badge in admin, sort by relevance
- [x] **Conversation analytics tab** - Admin dashboard shows anonymous conversation logs, stats (total/today/week), topic keyword detection (14 categories), program name detection (46 programs), side-by-side bar charts
- [x] **Unified conversations + feedback tab** - Merged separate Feedback and Conversations tabs into one; feedback matched to conversations client-side by question text; colored left borders (green=positive, red=negative, purple=none); expandable response accordion; filter dropdown (all/has-feedback/positive/negative); 5 stat boxes (total/today/week/thumbs up/thumbs down)
- [x] **CSV export** - Export all conversations with feedback ratings as CSV for text analysis

### 🔜 Next Up
- [ ] **Duplicate detection** - Flag articles with similar titles/content beyond URL match
- [ ] **Freshness tracking** - Surface articles older than 1 year for review

### 💡 Future Ideas
- [ ] **Weekly digest** - Email summary of new pending articles
- [ ] **Auto-approval threshold** - High-confidence articles go live without manual review
- [ ] **Feedback loop** - Demote articles that get negative student reactions

---

## Wizard Manifest Integration

Sandra consumes department-wizard manifests via a registry + live-fetch + cache chain so that wizard content changes reach the chatbot without code deploys.

### ✅ Completed

- [x] **Manifest loader** — live fetch with TTL + stale-while-revalidate, Firestore backup cache, static-file fallback (`functions/manifest-loader.js`)
- [x] **Manifest-to-context converter** — transforms wizard manifests into markdown sections for Claude's system prompt (`functions/manifest-to-context.js`)
- [x] **Wizard registry** — maps departments to live manifest URLs and fallback program IDs (`functions/wizard-registry.json`)
- [x] **Schema validation** — AJV-based validation on every fetch; rejects manifests that don't match `functions/schemas/manifest.schema.json`
- [x] **DCDA full-visibility fix (2026-04-10)** — converter now surfaces the full `courseCatalog` with descriptions, no longer truncates category course lists at 6 items, and handles `highlightedCourses` as both the legacy single-term object and the current array-of-term-objects shape (schema `oneOf`). POSC 31453 and every course past category index 6 is now reachable.
- [x] **Prompt-side precision rules (2026-04-10)** — added `COUNTING COURSES` rule (count unique codes, not occurrences) and replaced the obsolete "data may be outdated" instruction with language acknowledging the live manifest refresh.
- [x] **DCDA Firestore source of truth (2026-04-10)** — the DCDA wizard's generator now reads offerings directly from Firestore (`dcda_config/offerings_*`) via ADC, emitting every upcoming term instead of just one. Eliminates the silent drift where Admin-UI edits never reached Sandra until someone manually exported JSON files.

### 🔜 Next Up — English wizard (Engelina)

- [ ] **Add `courseCatalog` emitter** to `english-advising-wizard/scripts/generate-manifest.js`. Sandra's converter already knows how to surface the catalog when present — the only work is on the wizard side.
- [ ] **Triage English offerings files** — `src/data/` has 5 files (`offerings-fa25.json`, `offerings-fa26.json`, `offerings-fa26-2026-03-09.json` timestamped dupe, `offerings-sp26.json`, `offerings-wi26.json`). Identify what's authoritative and retire the rest.
- [ ] **Determine English's Firestore usage** — `firebase-admin` is not currently a dependency, suggesting static-JSON-only. Verify by grepping `src/` before deciding whether English also needs a Firestore migration.
- [ ] **Deploy and verify** — rebuild English wizard, confirm manifest has `courseCatalog > 0`, ask Engelina a course-level question, tail Sandra logs for a fresh `manifest_fetch_success` against English.

### 💡 Future Ideas

- [ ] **Cloud Function auto-regenerate** — Firestore trigger on `dcda_config/*` writes could regenerate and publish the manifest automatically, eliminating the `npm run build && firebase deploy` step from the feedback loop. Considered complex enough to scope as its own phase.
- [ ] **Retire static `data/offerings-*.json` files** in dcda-advising-wizard once the Firestore-only path has proven stable in production.

---

## Security / Infrastructure

### ✅ Completed
- [x] **Firebase Auth for admin** - Google sign-in with email whitelist; server-side ID token verification on all 7 admin endpoints
- [x] **Rate limiting on chat API** - Per-IP fixed-window rate limiting (30/hr chat, 60/hr feedback) via Firestore counters; 429 with friendly UI message
- [x] **Custom domain** - `sandra.digitcu.org` via Cloudflare DNS → Firebase Hosting

### 🔜 Next Up

### 💡 Future Ideas
- [ ] **SSRF protection on fetchUrlMetadata** - Validate URLs against allowlist / block private IP ranges
- [ ] **Server-side conversation history** - Store history by session instead of trusting client-submitted data

---

## Article Sources

### Active
- Fast Company (RSS)
- Harvard Business Review (RSS)
- New York Times Education (RSS)
- OpenAlex (API)

### Potential
- WEF: `https://www.weforum.org/feed/`
- Chronicle of Higher Education: `https://www.chronicle.com/feed`
- Inside Higher Ed
- NACE (National Association of Colleges and Employers)
- MIT Technology Review

---

## Technical Notes

### Article Schema
```javascript
{
  title: "Article Title",
  source: "Publication Name",
  url: "https://...",
  date: Timestamp,
  summary: "2-3 sentence summary for context",
  tags: ["family-talking-points", "employer-data"],
  status: "approved" | "pending" | "rejected",
  relevanceScore: 1-10,        // AI-assigned fit rating
  relevanceReason: "string",   // One-sentence explanation
  ai_curated: boolean,
  created_at: Timestamp,
  reviewed_at: Timestamp
}
```

### Program Data Schema
```javascript
{
  name: "Anthropology",
  abbreviation: "ANTH",
  degree: "BA",
  totalHours: 30,
  url: "https://www.tcu.edu/academics/programs/anthropology.php",
  descriptions: ["paragraph 1", "paragraph 2"],
  requirements: {
    requiredCourses: { hours: 9, courses: ["ANTH 20613 - Intro...", ...] },
    electiveCourses: { hours: 21, description: "..." }
  },
  careerOptions: ["Non-profit Manager", ...],
  contacts: [{ role: "Department Chair", name: "...", email: "...", phone: "..." }],
  internship: { description: "..." }
}
```

### Active RSS Endpoints
- Fast Company: `https://www.fastcompany.com/section/work-life/rss`
- Harvard Business Review: `https://feeds.hbr.org/harvardbusiness`
- New York Times (Education): `https://rss.nytimes.com/services/xml/rss/nyt/Education.xml`

### Infrastructure
- Firebase Hosting + Cloud Functions (Node.js 22, firebase-functions v7)
- Firestore for articles, feedback, conversations
- Claude Sonnet via Anthropic API (system prompt + program data + research KB + articles)
- Static knowledge: `la-value-research.json` (30 institutional sources), 60 program JSONs, Core Curriculum, DCDA, support resources

# Simple Showcase: Deployment Plan

## Executive Summary

Deploy an Eiffel-powered website to Oracle Cloud Free Tier that showcases the entire simple_* ecosystem. The site will demonstrate that a complete web application—server, database, HTML rendering, API—can be built end-to-end in Eiffel.

**Goal**: A live, public website running on simple_web, backed by simple_sql, rendered with simple_htmx + simple_alpine, proving AI-assisted Eiffel development is production-ready.

---

## Build Order (Revised)

| Phase | Task | Duration |
|-------|------|----------|
| 1 | **Build simple_alpine** | 10-12 hours |
| 2 | **Build simple_showcase** (local dev) | 8-10 hours |
| 3 | **Oracle + Cloudflare setup** | 2 hours |
| 4 | **Deploy** | 1 hour |

**Rationale**: Build and test locally first, deploy once when ready.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         INTERNET                                │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    CLOUDFLARE (Free Tier)                       │
│  • SSL termination (HTTPS)                                      │
│  • DDoS protection                                              │
│  • DNS management                                               │
│  • Static asset caching                                         │
└─────────────────────────────────────────────────────────────────┘
                              │ HTTP (port 80)
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│              ORACLE CLOUD FREE TIER (Windows VM)                │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                  showcase_server.exe                       │  │
│  │  • simple_web HTTP server                                  │  │
│  │  • simple_htmx HTML rendering                              │  │
│  │  • simple_sql SQLite backend                               │  │
│  │  • simple_json config parsing                              │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                   │
│                              ▼                                   │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    showcase.db (SQLite)                    │  │
│  │  • Analytics (page views, interactions)                    │  │
│  │  • Contact form submissions                                │  │
│  │  • Admin sessions                                          │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Infrastructure Setup

### Phase 1: Oracle Cloud Account & VM (30-45 minutes)

#### Step 1: Create Oracle Cloud Account
1. Go to https://cloud.oracle.com
2. Click "Start for Free"
3. Enter email, country, name
4. **Credit card required** (verification only, not charged for free tier)
5. Choose home region (closest to you for RDP latency)
6. Wait for account provisioning (~5 minutes)

#### Step 2: Create Windows VM
1. Navigate to: **Compute → Instances → Create Instance**
2. Configure:
   - **Name**: `showcase-server`
   - **Image**: Change to **Windows Server 2022 Standard**
   - **Shape**: VM.Standard.E2.1.Micro (free tier - 1GB RAM, 1/8 OCPU)
   - **Networking**: Create new VCN or use default, assign public IP
   - **Boot volume**: 50GB (default)
3. Click **Create**
4. Wait for provisioning (~5-10 minutes)
5. **Save the initial Windows password** displayed (you'll need it for RDP)

#### Step 3: Configure Security List (Firewall)
1. Go to: **Networking → Virtual Cloud Networks → [Your VCN] → Security Lists**
2. Add **Ingress Rules**:
   ```
   Source CIDR     Protocol    Dest Port    Description
   ─────────────────────────────────────────────────────
   0.0.0.0/0       TCP         80           HTTP
   0.0.0.0/0       TCP         443          HTTPS (optional)
   0.0.0.0/0       TCP         3389         RDP (already exists)
   ```
3. Windows Firewall (inside VM) will also need port 80 opened

#### Step 4: Connect via RDP
1. Get public IP from instance details
2. Open Remote Desktop Connection (mstsc.exe)
3. Connect to: `<public-ip>`
4. Username: `opc`
5. Password: (the one from Step 2)
6. Change password on first login

#### Step 5: Configure Windows Firewall
Inside the VM via RDP:
```powershell
# Allow HTTP traffic
New-NetFirewallRule -DisplayName "HTTP In" -Direction Inbound -Protocol TCP -LocalPort 80 -Action Allow
```

---

### Phase 2: Domain & Cloudflare Setup (20-30 minutes)

#### Step 1: Purchase Domain
**Recommended registrars** (cheap, no upselling):
- **Porkbun**: ~$9/year for .com
- **Namecheap**: ~$10/year for .com
- **Cloudflare Registrar**: At-cost pricing (~$9/year)

**Suggested domain names**:
- `eiffel-showcase.com`
- `simple-eiffel.dev`
- `eiffel-web.com`
- Or use your existing domain with a subdomain: `showcase.yourdomain.com`

#### Step 2: Create Cloudflare Account
1. Go to https://cloudflare.com
2. Sign up (free)
3. Click **Add a Site**
4. Enter your domain name
5. Select **Free Plan**

#### Step 3: Configure DNS in Cloudflare
1. Cloudflare scans existing DNS records
2. **Add/Edit A Record**:
   ```
   Type    Name    Content              Proxy Status
   ────────────────────────────────────────────────────
   A       @       <Oracle VM IP>       Proxied (orange cloud)
   A       www     <Oracle VM IP>       Proxied (orange cloud)
   ```
3. Continue to get nameservers

#### Step 4: Update Nameservers at Registrar
1. Cloudflare gives you 2 nameservers like:
   ```
   elsa.ns.cloudflare.com
   todd.ns.cloudflare.com
   ```
2. Go to your domain registrar's DNS settings
3. Replace existing nameservers with Cloudflare's
4. Save changes
5. Wait for propagation (can take 5 minutes to 24 hours, usually ~30 mins)

#### Step 5: Configure Cloudflare SSL Settings
1. In Cloudflare dashboard: **SSL/TLS → Overview**
2. Set encryption mode to: **Flexible**
   - This means: User→Cloudflare is HTTPS, Cloudflare→Your Server is HTTP
   - Your server only needs to serve HTTP on port 80
3. Enable **Always Use HTTPS** in SSL/TLS → Edge Certificates

#### Step 6: Verify Setup
```bash
# Check DNS propagation
nslookup yourdomain.com

# Should return Cloudflare IPs (not your server IP directly)
# That means Cloudflare is proxying correctly
```

---

## Application Development

### What You Have (Ready to Use)

| Library | Purpose | Location |
|---------|---------|----------|
| simple_web | HTTP server, routing, request/response | D:\prod\simple_web |
| simple_htmx | Fluent HTML/HTMX builder | D:\prod\simple_htmx |
| simple_sql | SQLite database | D:\prod\simple_sql |
| simple_json | JSON config parsing | D:\prod\simple_json |
| simple_gui_designer | Demo application | D:\prod\simple_gui_designer |

### What to Build: `simple_showcase`

#### Project Structure

```
D:\prod\simple_showcase\
├── simple_showcase.ecf           # Eiffel config
├── config.json                   # Runtime config
├── showcase.db                   # SQLite database (created on first run)
├── src/
│   ├── app/
│   │   └── showcase_app.e                    # Entry point
│   ├── server/
│   │   └── showcase_server.e                 # Main server, inherits handlers
│   ├── handlers/
│   │   ├── showcase_shared_state.e           # Base with DB, config access
│   │   ├── showcase_public_handlers.e        # Public pages
│   │   ├── showcase_admin_handlers.e         # Admin dashboard
│   │   ├── showcase_api_handlers.e           # Demo APIs
│   │   └── showcase_contact_handlers.e       # Contact form + email
│   ├── middleware/
│   │   └── showcase_analytics_middleware.e   # Log all requests
│   ├── auth/
│   │   └── showcase_auth.e                   # Password verify, sessions
│   ├── email/
│   │   └── showcase_email.e                  # SMTP via cURL
│   └── db/
│       └── showcase_db_setup.e               # Schema creation
├── static/
│   ├── css/
│   │   └── style.css
│   ├── js/
│   │   └── app.js
│   └── images/
│       └── eiffel-logo.png
└── testing/
    └── test_showcase.e
```

#### Routes

| Method | Route | Handler | Auth | Purpose |
|--------|-------|---------|------|---------|
| GET | `/` | public | No | Landing page - hero, value prop |
| GET | `/projects` | public | No | List all simple_* projects |
| GET | `/project/{name}` | public | No | Individual project details |
| GET | `/productivity` | public | No | AI productivity stats |
| GET | `/demo/gui-designer` | public | No | Embedded GUI designer |
| GET | `/demo/api` | public | No | Interactive API playground |
| GET | `/about` | public | No | The story, methodology |
| GET | `/contact` | public | No | Contact form |
| POST | `/contact` | contact | No | Submit form, send email |
| GET | `/admin` | admin | No | Login page |
| POST | `/admin/login` | admin | No | Authenticate |
| GET | `/admin/dashboard` | admin | Yes | Analytics overview |
| GET | `/admin/analytics` | admin | Yes | Detailed page views |
| GET | `/admin/contacts` | admin | Yes | Contact submissions |
| POST | `/admin/logout` | admin | Yes | End session |
| GET | `/static/*` | static | No | CSS, JS, images |

#### Database Schema

```sql
-- Analytics: Track every page view
CREATE TABLE IF NOT EXISTS analytics (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    path TEXT NOT NULL,
    method TEXT NOT NULL,
    ip_address TEXT,
    user_agent TEXT,
    referrer TEXT,
    response_code INTEGER,
    response_time_ms INTEGER,
    created_at TEXT DEFAULT (datetime('now'))
);

CREATE INDEX idx_analytics_path ON analytics(path);
CREATE INDEX idx_analytics_created ON analytics(created_at);

-- Contact form submissions
CREATE TABLE IF NOT EXISTS contacts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    email TEXT NOT NULL,
    message TEXT NOT NULL,
    ip_address TEXT,
    read_at TEXT,
    created_at TEXT DEFAULT (datetime('now'))
);

-- Admin sessions
CREATE TABLE IF NOT EXISTS sessions (
    token TEXT PRIMARY KEY,
    created_at TEXT DEFAULT (datetime('now')),
    expires_at TEXT NOT NULL
);

-- Clean up expired sessions periodically
CREATE INDEX idx_sessions_expires ON sessions(expires_at);
```

#### Config File (config.json)

```json
{
    "server": {
        "port": 80,
        "host": "0.0.0.0"
    },
    "database": {
        "path": "showcase.db"
    },
    "admin": {
        "password_hash": "<bcrypt or sha256 hash>",
        "session_duration_hours": 24
    },
    "email": {
        "smtp_host": "smtp.gmail.com",
        "smtp_port": 587,
        "smtp_user": "your-email@gmail.com",
        "smtp_password": "<app-password>",
        "from_address": "your-email@gmail.com",
        "to_address": "your-email@gmail.com"
    },
    "site": {
        "title": "Eiffel Showcase",
        "description": "End-to-end Eiffel web development"
    }
}
```

#### Email Setup (Gmail App Password)

1. Go to Google Account → Security
2. Enable 2-Factor Authentication (required)
3. Go to App Passwords
4. Generate password for "Mail" on "Windows Computer"
5. Use that 16-character password in config.json

---

## Implementation Order

### Day 1 Morning: Infrastructure (~2 hours)

| # | Task | Time |
|---|------|------|
| 1 | Create Oracle Cloud account | 15 min |
| 2 | Create Windows VM | 15 min |
| 3 | Configure security list (port 80) | 10 min |
| 4 | RDP in, configure Windows firewall | 10 min |
| 5 | Purchase domain | 10 min |
| 6 | Set up Cloudflare account | 10 min |
| 7 | Configure DNS, nameservers | 15 min |
| 8 | Verify HTTPS working (test page) | 15 min |

### Day 1 Afternoon: Application (~6-8 hours)

| # | Task | Est. Hours |
|---|------|------------|
| 1 | Scaffold simple_showcase project, ECF | 0.5 |
| 2 | Basic server with static file serving | 1 |
| 3 | Database setup, schema creation | 0.5 |
| 4 | Analytics middleware | 1 |
| 5 | Public pages (home, projects, about) | 1.5 |
| 6 | Admin auth (password, sessions) | 1 |
| 7 | Admin dashboard with analytics | 1 |
| 8 | Contact form + email sending | 1 |
| 9 | GUI designer demo integration | 0.5 |
| **Total** | | **8 hours** |

### Day 1 Evening: Deployment (~1 hour)

| # | Task | Time |
|---|------|------|
| 1 | Compile finalized exe | 15 min |
| 2 | Copy files to Oracle VM | 10 min |
| 3 | Create config.json with prod values | 10 min |
| 4 | Run server, test all routes | 15 min |
| 5 | Set up as Windows Service (optional) | 10 min |

---

## Page Content Plan

### Landing Page (/)

```
┌─────────────────────────────────────────────────────────────────┐
│                        EIFFEL SHOWCASE                          │
│                                                                 │
│     End-to-End Web Development in Eiffel                        │
│                                                                 │
│     This entire website—server, database, HTML rendering—       │
│     is built with Eiffel using the simple_* library ecosystem   │
│                                                                 │
│     [View Projects]  [See Productivity Stats]  [Try Demo]       │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│  THE STACK                                                      │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌───────────┐  │
│  │ simple_web  │ │ simple_htmx │ │ simple_sql  │ │simple_json│  │
│  │   Server    │ │    HTML     │ │   SQLite    │ │  Config   │  │
│  └─────────────┘ └─────────────┘ └─────────────┘ └───────────┘  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│  BY THE NUMBERS                                                 │
│                                                                 │
│     60,000+        750+           10            ~85             │
│     Lines          Tests        Projects       Hours            │
│                                                                 │
│     45-75x productivity multiplier vs traditional development   │
└─────────────────────────────────────────────────────────────────┘
```

### Projects Page (/projects)

List each simple_* library with:
- Name and description
- Line count, test count
- Key features
- GitHub link
- "This site uses this library for: X"

### Productivity Page (/productivity)

Visualize the AI productivity data:
- Timeline of development
- Lines per day velocity chart
- Cost savings calculation
- Human-AI collaboration model

### Demo Page (/demo/gui-designer)

Embed the GUI designer or link to interactive demo

---

## Running as Windows Service (Optional)

To keep the server running after logout:

```powershell
# Using NSSM (Non-Sucking Service Manager)
# Download from nssm.cc

nssm install ShowcaseServer "D:\showcase\showcase_server.exe"
nssm set ShowcaseServer AppDirectory "D:\showcase"
nssm set ShowcaseServer DisplayName "Eiffel Showcase Server"
nssm set ShowcaseServer Start SERVICE_AUTO_START
nssm start ShowcaseServer
```

Or simpler: Use Task Scheduler to run on startup.

---

## Checklist Summary

### Before Starting
- [ ] Credit card ready for Oracle (not charged)
- [ ] Domain name chosen
- [ ] Gmail app password created

### Infrastructure
- [ ] Oracle Cloud account created
- [ ] Windows VM running
- [ ] Port 80 open (Oracle + Windows firewall)
- [ ] Can RDP into VM
- [ ] Domain purchased
- [ ] Cloudflare account created
- [ ] DNS configured, proxied through Cloudflare
- [ ] HTTPS working (test with placeholder page)

### Application
- [ ] simple_showcase project scaffolded
- [ ] Database schema created
- [ ] Analytics middleware logging requests
- [ ] Public pages rendering
- [ ] Admin authentication working
- [ ] Admin dashboard showing analytics
- [ ] Contact form sending email
- [ ] GUI designer demo embedded

### Deployment
- [ ] Finalized exe compiled
- [ ] Files copied to VM
- [ ] Config.json with production values
- [ ] Server running on port 80
- [ ] All routes working via HTTPS
- [ ] Server set to auto-start

---

## Cost Summary

| Item | Cost |
|------|------|
| Oracle Cloud VM | $0 (free forever) |
| Cloudflare | $0 (free tier) |
| Domain | ~$10/year |
| **Total** | **~$10/year** |

---

## Success Criteria

The showcase is complete when:

1. **Public can visit** `https://yourdomain.com` and see the landing page
2. **Projects page** displays all simple_* libraries with stats
3. **Productivity page** shows AI development metrics
4. **Demo works** - GUI designer or API playground functional
5. **Contact form** sends email to you
6. **Admin dashboard** shows page view analytics
7. **Server stable** - runs without crashes, auto-restarts

---

## Tomorrow Morning: Quick Start

1. **First 30 min**: Oracle account + VM creation (wait for provisioning)
2. **While waiting**: Buy domain, create Cloudflare account
3. **Next 30 min**: Configure firewalls, DNS, verify HTTPS
4. **Then**: Start coding simple_showcase

**You'll have live infrastructure within the first hour, then it's all Eiffel coding.**

---

**Document Created**: December 3, 2025
**Target Completion**: December 4, 2025
**Estimated Total Time**: 10-12 hours

# Server-Side Tracking Migration (sGTM) — End-to-End Implementation Report

This report provides a comprehensive walkthrough of the server-side tracking migration I conducted for siete.ch.
The project was completed over two weeks of self-directed implementation, during which I designed, deployed, and debugged the full tracking architecture.
Throughout the process, I relied on external expert resources such as the implementation tutorials from **analyticsmania.com**, the **Team Simmer Technical Marketer** curriculum, and official **Google Help Center** documentation.
These sources played an essential role in validating architectural decisions, understanding server-side behavior and deepening my overall technical marketing skillset.

Cross-disciplinary across:
- client-side tracking (GTM Web)
- server-side tagging (sGTM)
- Cloudflare Workers
- Google Cloud Run infrastructure
- DNS and reverse proxy routing
- analytics debugging (GA4 DebugView, Network logs)
- WordPress tracking conflicts and plugin injections

---

# 1. Overview

| Layer | Component | Role |
|-------|-----------|------|
| Client | **GTM Web Container (GTM-XXXXXXX)** | Generates events in browser |
| Edge | **Cloudflare Worker** | Routes requests from ss.example.com → Google Cloud Run |
| Server | **sGTM Container (GTM-YYYYYYY)** | Processes events and forwards to GA4 |
| Analytics | **GA4 Property (G-XXXXXXXXXX)** | Receives server-side processed hits |
| Infrastructure | **Google Cloud Run** | Hosts the sGTM container |

---

# 2. Architecture

                 ┌─────────────────────────────┐
                 │      User Browser            │
                 └──────────────┬───────────────┘
                                │
                                ▼
                 ┌─────────────────────────────┐
                 │  GTM Web Container           │
                 │  (Client-Side)               │
                 └──────────────┬───────────────┘
                                │ 1st Party Request
                                ▼
                 ┌─────────────────────────────┐
                 │ ss.example.com               │
                 │  (CNAME → Cloudflare)        │
                 └──────────────┬───────────────┘
                                │ Reverse Proxy
                                ▼
                 ┌─────────────────────────────┐
                 │ Cloudflare Worker            │
                 │ Rewrites host → Google Run   │
                 └──────────────┬───────────────┘
                                │
                                ▼
                 ┌─────────────────────────────┐
                 │ Google Cloud Run             │
                 │ (sGTM Server Container)      │
                 └──────────────┬───────────────┘
                                │
                                ▼
                 ┌─────────────────────────────┐
                 │ Google Analytics 4           │
                 └─────────────────────────────┘


---

# 3. Screenshots

Cloudflare Worker - Performance 

![Cloudflare Screenshot](docs/cloudflare_performance.png)

GA4 - Traffic Snapshot

![GA4 Screenshot](docs/ga4_traffic_overview.png)

---

# 4. Tools, Platforms & Components Used

Tracking & Tag Management
- GTM Web Container (anonymized: GTM-XXXXXXX)
- GTM Server Container (anonymized: GTM-YYYYYYY)
- GA4 Property (anonymized: G-XXXXXXXXXX)
- WordPress
- WooCommerce
- WPCode Lite (for injecting gtag.js and GTM snippets)

Cloud Infrastructure
- Cloudflare Workers
- Cloudflare DNS
- Cloudflare Proxy Layer
- Google Cloud Run (container hosting)
- Google Cloud Console (API activation, billing administration)

Debugging & Diagnostics
- GA4 DebugView
- Chrome Network inspector
- Cloudflare Worker Logs
- sGTM Preview Mode

---

# 5. Main Challenges (for me)

## Challenge 1 - Server Container initially unreachable (404 Not Found)

Cause: The original Google App Enginge endpoint was inactive because no billing account existed.

Fix: 
- Created a new sGTM container in Google Cloud Run
- Configured active billing
- Updated Cloudflare Worker to new upstream host
- Re-published the container

## Challenge 2 - Browser continued sending hits directly to Google (u24a)

Cause: WordPress plugins injected their own tracking scripts: GTM4WP, Google for WooCommerce, GA for WooCommerce

Fix: 
- Removed all tracking plugins
- Replaced with minimal injection using WPCode Lite
- Defined server_container_url in the Web GTM tag: server_container_url = "https://ss.example.com"
- Cleared WordPress cache + Cloudflare Cache + Browser Cache

## Challenge 3 - ERR_EMPTY_RESPONSE when loading GTM container script

Cause: The Cloudflare proxy forwarded `/gtm.js` to the server container, but the server container was not configured to serve Web GTM code.

Fix: 
- Added a “GTM Web Container Client” inside the server GTM container
- Authorized it to serve the Web container code
- Re-published the server container

## Challenge 4 - Only `page_view` appeared in GA DebugView

Cause: Only the GA4 Base Tag was firing server-side. No generic event tag existed

Fix:
Created a generic GA4 event tag in sGTM:
- reads `{{Event_Name}}`
- forwards all custom events go GA4
- excludes `page_view` (already handled by base tag)





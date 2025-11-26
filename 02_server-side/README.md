# 02_server-side

Docu about deploy a serve-side tracking over cloudflare (for free) following soon.

This module documents a server side tracking architecture based on a Tag Manager Server Container.  
It is designed to compare signal quality, resilience, and control versus the client side baseline in 01.

Server side tracking introduces an HTTP relay layer between the browser and analytics or ad platforms.

---

## Architecture Overview
Client events are generated in the browser, forwarded to a server side endpoint, then redistributed to destinations.

Flow:
Browser  
Tag Manager Web Container
Server side Tag Manager Container  
Destinations (GA4, Meta CAPI, TikTok Events API, Google Ads, GMP)

Key property:
The browser does not send measurement hits directly to third party endpoints.  
Instead, a first party server endpoint receives and forwards them.

---

## Components

| Component | Example | Role |
|----------|---------|------|
| Web Tag Manager | GTM Web Container | Generates events in the browser |
| Server Container | GTM Server Container | Receives first party requests and forwards to destinations |
| Hosting for Server Container | Cloud Run or App Engine | Runs the server container |
| First party Subdomain | ss.example.com | First party collection endpoint |
| Analytics Destination | GA4 | Receives server forwarded hits |
| Advertising Destinations | Meta CAPI, TikTok Events API | Receives server forwarded conversions |

---

## First Party Endpoint Setup
The server container is exposed via a first party subdomain.

Example:
ss.example.com → Server Container URL

DNS mapping depends on your registrar and hosting provider, but the logic is always:
Subdomain CNAME or A record points to the server container endpoint.

It enables measurement of:
Event delivery under browser restrictions  
Adblock resistance  
Cookie lifetime extension  
Cross platform matching improvement  
Central governance of event logic


---
title: "Cloudflare BGP Route Leak Incident"
category: "Network"
date_of_incident: 2026-01-22
system_affected: "Global Anycast Network / BGP Routing"
failure_type: "Route Leak"
severity: "High"
---

### 1. System Architecture and Context

* Global Anycast network architecture utilizing Border Gateway Protocol (BGP) to advertise IP prefixes from multiple data centers simultaneously.
* Utilization of RPKI (Resource Public Key Infrastructure) for Route Origin Authorization (ROA) to validate prefix ownership.
* Peering infrastructure consisting of direct Private Network Interconnects (PNIs), Internet Exchange Points (IXPs), and Tier 1 Transit providers.
* Automated route monitoring systems designed to detect prefix hijacking and path anomalies.

### 2. What went wrong (Trigger)

* A Tier 1 transit provider incorrectly accepted and re-propagated a large set of Cloudflare IP prefixes from a third-party Autonomous System (AS).
* The third-party AS had misconfigured its BGP export filters during a router migration, leaking internal routes to the public internet.
* The leak included "more specific" prefixes (/24 instead of the usual /20 or /16), which, according to BGP path selection algorithms, are prioritized over shorter prefix advertisements regardless of path length.
* Invalid BGP advertisements bypassed certain filter layers because the transit provider's RPKI validation failed to drop "Invalid" states on that specific peering session.

### 3. Cascading Effects and Impact

* Global internet traffic intended for Cloudflare's edge was diverted to the misconfigured AS, which lacked the capacity to handle the volume.
* Inbound traffic met saturated links at the leaking AS, resulting in 95% packet loss for affected IP ranges.
* User-facing impact manifested as "Connection Timed Out" (Error 522) and "Origin Unreachable" errors across multiple geographic regions.
* The "blast radius" included several major Cloudflare IP ranges, impacting approximately 15% of total global traffic for a duration of 45 minutes.
* BGP convergence oscillations (flapping) occurred as the global routing table attempted to stabilize, leading to intermittent connectivity even after the initial leak began to be filtered.

### 4. Resolution and Prevention

* Network engineers identified the leaking AS and contacted the Transit provider's NOC to manually clear the BGP session.
* Cloudflare utilized "no-export" communities and prefix withdrawal on the impacted peering links to force traffic onto alternative, stable paths.
* Implemented stricter BGP Max-Prefix limits on all peering sessions to automatically shutdown sessions that advertise an unexpected number of routes.
* Expanded the "Path Validation" logic within the automated network management system to trigger immediate, automated alerts for "more specific" prefix advertisements observed via global BGP collectors.
* Advocated for and assisted the transit partner in implementing strict RPKI "Reject Invalid" policies across their entire edge to ensure unauthorized advertisements are dropped by default.

---
title: "Cloudflare July 2, 2019 Global Outage"
category: "Cloud Infrastructure"
date_of_incident: 2019-07-02
system_affected: "WAF (Web Application Firewall) / NGINX"
failure_type: "Configuration Drift / CPU Exhaustion"
severity: "Critical"
---

### 1. System Architecture and Context

* Global edge network utilizing NGINX worker processes to handle and route HTTP/HTTPS traffic.
* Web Application Firewall (WAF) integrated as a Lua module within the NGINX request pipeline.
* Centralized rule management system using "Quicksilver" for rapid, global distribution of security configurations.
* Edge nodes distributed across 193 cities globally, all receiving synchronous configuration updates.

### 2. What went wrong (Trigger)

* A software engineer deployed a new WAF managed rule designed to mitigate Cross-Site Scripting (XSS) attacks.
* The rule contained a highly unoptimized regular expression: `.*(?:.*=.*)`.
* The inclusion of nested, non-capturing groups with greedy quantifiers caused the NGINX regex engine (PCRE) to enter catastrophic backtracking when evaluating specific HTTP request bodies.
* The CPU demand for evaluating the regex against request strings grew exponentially, pinning NGINX worker processes at 100% utilization.

### 3. Cascading Effects and Impact

* Immediate CPU exhaustion occurred across all NGINX worker processes on a global scale as the rule was synchronized via Quicksilver.
* NGINX workers became unable to process incoming traffic or heartbeat signals, leading to 502 Bad Gateway errors for nearly all proxied traffic.
* Local health checks failed, but because the issue was universal across all PoPs (Points of Presence), traffic could not be rerouted to healthy nodes.
* Global traffic dropped by 82% during the peak of the incident.
* Internal administrative tools and dashboards became inaccessible or unresponsive due to the shared infrastructure load.

### 4. Resolution and Prevention

* Engineers executed a global "kill switch" to disable the WAF engine entirely, restoring CPU headroom and allowing NGINX to resume request processing.
* Implemented mandatory staged rollouts for all WAF rule changes, moving from global instant deployment to a canary-based model (P90, then global).
* Introduced automated regex complexity analysis to reject patterns prone to catastrophic backtracking during the CI/CD phase.
* Deployed a WAF execution timeout mechanism to terminate long-running regex evaluations without crashing the worker process.
* Began the transition of the WAF engine from NGINX/Lua to a specialized Rust-based implementation (Firecell) to provide better resource isolation.

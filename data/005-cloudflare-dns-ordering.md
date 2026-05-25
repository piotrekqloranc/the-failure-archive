---
title: "Cloudflare DNS CNAME/A Record Ordering Non-Compliance"
category: "Network"
date_of_incident: 2021-03-31
system_affected: "Authoritative DNS Pipeline"
failure_type: "Configuration Drift / Protocol Non-Compliance"
severity: "Medium"
---

### 1. System Architecture and Context

* Cloudflare authoritative DNS servers utilize "CNAME Flattening" to provide IP addresses (A/AAAA records) at the zone apex.
* The DNS response generation logic constructs Answer sections containing both the CNAME and the target address records.
* Stub resolvers and recursive DNS clients rely on the order of records within the DNS Answer section to traverse alias chains.
* DNS protocol implementations typically follow RFC 1034/1035 conventions for record sequencing.

### 2. What went wrong (Trigger)

* A change was deployed to the DNS response logic to optimize packet size and performance by reordering or omitting specific CNAME metadata.
* The update resulted in DNS responses where the A/AAAA record preceded the CNAME record in the Answer section, or the CNAME was omitted entirely while returning the A record.
* The trigger was the strict adherence to a literal interpretation of DNS specifications that did not account for legacy/stub resolver behaviors that expect a sequential chain.

### 3. Cascading Effects and Impact

* Legacy DNS resolver libraries (notably older getaddrinfo implementations in specific Unix-like systems) failed to process the out-of-order records.
* Impacted clients reported "Host Not Found" or "NXDOMAIN" errors despite the presence of valid A/AAAA records in the DNS response.
* Certain applications using non-standard DNS parsing logic experienced intermittent connection failures or timeout errors.
* The failure was localized to specific client environments but appeared globally across the Cloudflare edge where the new logic was active.

### 4. Resolution and Prevention

* Network engineers rolled back the DNS response logic to restore the conventional "CNAME followed by A/AAAA" record ordering.
* Implemented a "standard-first" approach for DNS record serialization to ensure maximum compatibility with legacy stub resolvers.
* Expanded the test suite for the DNS authoritative engine to include a wider range of legacy client OS environments and resolver libraries.
* Formalized internal documentation regarding "Record Sequencing" to prevent future optimizations from violating implicit industry standards that are not strictly enforced by RFCs but are required for interoperability.

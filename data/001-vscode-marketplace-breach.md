---
title: "Git Token and Credentials Harvesting via Malicious VS Code Extensions"
category: "Cybersecurity"
date_of_incident: 2024
system_affected: "Developer Workstations & VS Code Marketplace"
failure_type: "Supply Chain Attack / Credential Harvesting"
severity: "High"
---

### 1. System Architecture and Context

* Local integrated development environments (IDEs) like VS Code function as fully trusted environments on developer workstations.
* Developers cache highly sensitive authentication tokens, such as GitHub Personal Access Tokens (PATs) and Copilot states, locally on disk (e.g., `~/.config/github-copilot/hosts.json`).
* Third-party utility modules published on public marketplaces are routinely executed without dedicated sandboxing or permission constraints.
* By design, executing plugins inherit full workspace permissions, retaining raw read access to the developer's filesystem and external network ports.

### 2. What went wrong (Trigger)

* Threat actors uploaded malicious, typo-squatting variants of popular utility extensions onto the official VS Code Marketplace, mimicking standard formatting or linting helpers.
* Upon installation, the extension activation lifecycle triggered the automated execution of an obfuscated background NodeJS script.
* The script routines bypassed default malware detection controls because the parent process (VS Code) is historically whitelisted on developer machines.

### 3. Cascading Effects and Impact

* Background routines successfully parsed and located cached session keys, environment variables, and tokens inside local configuration files.
* Harvested tokens and credentials were exfiltrated outbound via HTTPS requests directly to the attackers' command-and-control servers.
* Attackers utilized the stolen tokens to execute bulk API requests, leading to the unauthorized cloning and theft of over 3,800 private source code repositories.
* Stolen repositories were subsequently scanned for hardcoded production passwords and API keys, creating a risk of secondary enterprise breaches.

### 4. Resolution and Prevention

* Enterprise administrators invoked global active token expiration rules, instantly revoking all leaked credentials and SSH associations.
* Implemented rigid workstation policies permitting only verified marketplace extension publishers and enabling strict workspace isolation.
* Enforced cryptographic commit validations requiring GPG key signatures to block unauthorized code pushes from compromised developer machines.
* Formalized secure coding guidelines requiring extension developers to use local OS keychain handlers (like `SecretsState` or `keytar`) instead of storing plaintext keys in filesystem configuration folders.

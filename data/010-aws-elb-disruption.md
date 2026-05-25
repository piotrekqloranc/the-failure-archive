---
title: "Amazon ELB Service Disruption (US-EAST-1)"
category: "Cloud Infrastructure"
date_of_incident: 2012-12-24
system_affected: "Elastic Load Balancing (ELB) Control Plane"
failure_type: "Human Error / Data Deletion"
severity: "High"
---

### 1. System Architecture and Context

* The Elastic Load Balancing (ELB) control plane manages the lifecycle, configuration, and health monitoring of all load balancer instances.
* Control plane operations rely on a primary metadata store to track the state and configuration of every ELB in a given region.
* The system is designed to automatically scale load balancer capacity in response to fluctuating traffic patterns.
* The service architecture is distributed across multiple Availability Zones in the US-EAST-1 region to facilitate fault tolerance.

### 2. What went wrong (Trigger)

* An authorized developer executed a maintenance tool intended to remove orphaned data from the ELB control plane's production database.
* The maintenance tool was configured with an incorrect parameter, leading it to identify and delete active production state records instead of only the intended orphaned records.
* This resulted in the unintentional deletion of critical configuration data for a significant portion of the load balancers in the US-EAST-1 region.

### 3. Cascading Effects and Impact

* The loss of state data caused the ELB control plane to lose visibility into existing load balancer instances, making them unmanageable.
* The system became unable to scale existing ELB capacity or perform health-check updates, resulting in traffic being routed to potentially unhealthy back-end instances.
* API calls for the creation of new load balancers or the modification of existing ones failed globally across the region.
* The disruption prevented customers from scaling their infrastructure to handle high holiday traffic volumes.
* Automated recovery mechanisms were hindered because the control plane lacked the necessary metadata to reconcile the current state with the desired configuration.

### 4. Resolution and Prevention

* Engineering teams performed a Point-in-Time Recovery (PITR) of the control plane database using automated backups to restore the deleted metadata.
* Implemented "Soft Delete" functionality for critical configuration tables to allow for immediate record restoration without requiring a full database recovery.
* Restricted the use of administrative maintenance tools to require a secondary peer review and formal approval process for any script performing bulk data modifications.
* Added automated safety guardrails to maintenance scripts to detect and block executions that exceed a predefined threshold of record changes.
* Enhanced the ELB control plane with additional internal consistency checks to alert on large-scale state discrepancies.

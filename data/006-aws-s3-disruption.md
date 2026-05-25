---
title: "Amazon S3 US-EAST-1 Service Disruption"
category: "Cloud Infrastructure"
date_of_incident: 2017-02-28
system_affected: "Amazon S3 (Simple Storage Service)"
failure_type: "Human Error / Cascading Failure"
severity: "Critical"
---

### 1. System Architecture and Context

* Amazon S3 in the US-EAST-1 region utilizes a distributed architecture partitioned into sub-systems, including an Index Sub-system and a Placement Sub-system.
* The Index Sub-system manages metadata and mapping information required for all GET, LIST, PUT, and DELETE requests.
* The Placement Sub-system manages the allocation of new storage and relies on the Index Sub-system for operation.
* Multiple AWS services, including Elastic Block Store (EBS), Lambda, and the Service Health Dashboard (SHD), maintain hard dependencies on S3 for storage, snapshotting, and resource loading.

### 2. What went wrong (Trigger)

* During a routine troubleshooting of the S3 billing system, an authorized S3 team member executed a command intended to remove a small number of servers for a billing sub-system.
* The command was entered with an incorrect parameter, resulting in the unintended removal of a larger set of servers.
* The decommissioned servers supported two critical infrastructure components: the Index Sub-system and the Placement Sub-system.

### 3. Cascading Effects and Impact

* The sudden loss of capacity forced a full restart of the Index Sub-system, during which S3 was unable to service any requests in the US-EAST-1 region.
* EBS volumes failed to initialize or create snapshots as they could not access S3-backed data.
* AWS Lambda and EC2 instances experienced startup failures due to an inability to pull code packages or Amazon Machine Images (AMIs) from S3.
* The Service Health Dashboard (SHD) failed to update its status because the console relied on S3 for its underlying data store.
* The recovery period was prolonged because the Index Sub-system had not undergone a full cold-start in several years, leading to a long metadata validation cycle before it could resume request processing.

### 4. Resolution and Prevention

* Engineers manually re-provisioned the removed server capacity and monitored the sequential recovery of the Index and Placement sub-systems until full availability was restored.
* The internal tool used for decommissioning servers was modified to include safety guardrails that prevent the removal of capacity below a minimum functional threshold.
* The Index Sub-system was re-architected into smaller, independent partitions (cells) to reduce the blast radius and decrease the time required for a full system restart.
* AWS implemented cross-region redundancy for the Service Health Dashboard to ensure status visibility even during major regional storage outages.

---
title: "Amazon EC2 and EBS Service Disruption (US-EAST-1)"
category: "Cloud Infrastructure"
date_of_incident: 2011-04-21
system_affected: "Amazon EBS (Elastic Block Store)"
failure_type: "Cascading Failure"
severity: "Critical"
---

### 1. System Architecture and Context

* EBS architecture utilizes a primary-secondary mirroring model for data durability within a single Availability Zone (AZ).
* A specialized control plane manages the mapping of volumes to physical storage nodes and coordinates replication.
* The infrastructure relies on a redundant high-speed network to handle both standard I/O and "re-mirroring" traffic.
* EBS serves as the underlying storage for Amazon RDS and Amazon EC2 instances.

### 2. What went wrong (Trigger)

* During a routine network capacity expansion in one AZ, a configuration error occurred during a traffic shift.
* Traffic was inadvertently diverted from a high-capacity primary network link to a low-capacity redundant backup link.
* The backup link immediately became saturated, causing significant packet loss and preventing EBS nodes from communicating with their mirrored counterparts.

### 3. Cascading Effects and Impact

* Loss of connectivity triggered a "re-mirroring storm" as thousands of EBS nodes simultaneously attempted to find new available storage to recreate lost mirrors.
* The volume of re-mirroring requests overwhelmed the EBS control plane's capacity to process metadata and coordinate storage allocation.
* A race condition occurred where nodes continuously searched for capacity that was unavailable due to the control plane bottleneck, leading to a "stuck" state for volumes.
* Dependency failures cascaded to Amazon RDS and EC2, as instances became unresponsive while waiting for I/O operations to complete on impacted EBS volumes.
* The Service Health Dashboard (SHD) experienced delays in reporting as the underlying systems used for status updates were also affected by the EBS failure.

### 4. Resolution and Prevention

* Engineering teams manually disabled the automated re-mirroring logic to reduce the load on the control plane.
* Additional capacity was injected into the EBS control plane to clear the backlog of pending requests and restore volume mapping stability.
* Implemented "back-off" algorithms and throttles for re-mirroring processes to prevent a single network event from triggering a global storm.
* Enhanced network configuration procedures to include automated validation of link capacity before shifting production traffic.
* Increased the isolation of control plane resources between Availability Zones to ensure that a localized failure cannot impact regional service stability.

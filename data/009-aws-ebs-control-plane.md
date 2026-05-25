---
title: "Amazon EC2 and EBS Service Disruption (US-EAST-1)"
category: "Cloud Infrastructure"
date_of_incident: 2012-10-22
system_affected: "Amazon EBS (Elastic Block Store) Control Plane"
failure_type: "Cascading Failure / Resource Exhaustion"
severity: "High"
---

### 1. System Architecture and Context

* The EBS service in the US-EAST-1 region is divided into multiple independent Availability Zones (AZs) to provide fault isolation.
* The EBS control plane manages volume lifecycle events, including creation, attachment, and state transitions.
* EBS nodes within a data center maintain persistent connections to the control plane for metadata synchronization and state reporting.
* Dependent services, such as Amazon RDS and Amazon EC2, rely on the EBS control plane to provision and mount block storage.

### 2. What went wrong (Trigger)

* A localized power degradation occurred in a single data center within the US-EAST-1 region, causing a subset of EBS servers to lose power.
* Upon restoration of power, a large volume of EBS nodes attempted to reconnect to the EBS control plane simultaneously.
* A latent software bug in the control plane's connection handling logic caused it to consume excessive CPU cycles when processing a high volume of simultaneous reconnections.
* The resulting "thundering herd" effect led to CPU exhaustion on the control plane servers, rendering them unable to process legitimate volume state requests.

### 3. Cascading Effects and Impact

* The EBS control plane in the affected AZ became unresponsive, preventing any new volume attachments, detachments, or creations.
* Existing EBS volumes that were in the process of recovering from the power event became stuck in an "impaired" or "attaching" state because the control plane could not complete the necessary handshake.
* The failure cascaded to the Amazon RDS service, as it was unable to perform automated failovers or scale storage due to the EBS control plane bottleneck.
* A small percentage of EBS volumes in the affected AZ experienced prolonged unavailability as the control plane struggled to clear the backlog of recovery operations.
* The blast radius was largely contained to a single AZ, but the high volume of API retries from customers impacted the responsiveness of regional API endpoints.

### 4. Resolution and Prevention

* Engineering teams manually throttled the reconnection rate of EBS nodes to reduce the instantaneous load on the control plane.
* Additional compute capacity was provisioned for the EBS control plane to facilitate the processing of the recovery backlog.
* The software bug causing inefficient CPU utilization during reconnection was identified and patched across all regions.
* Implemented more aggressive exponential back-off and jitter in the EBS node reconnection logic to prevent synchronized thundering herd events.
* Enhanced the monitoring and alerting systems to detect abnormal CPU utilization spikes in the control plane specifically during AZ-level recovery scenarios.

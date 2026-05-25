---
title: "Amazon DynamoDB Service Disruption (US-EAST-1)"
category: "Database"
date_of_incident: 2015-09-20
system_affected: "Amazon DynamoDB"
failure_type: "Resource Exhaustion / Cascading Failure"
severity: "Critical"
---

### 1. System Architecture and Context

* Amazon DynamoDB utilizes a distributed storage architecture where data is distributed across thousands of storage nodes.
* A centralized internal metadata service tracks the location of data partitions and maps them to specific storage nodes.
* Front-end request routers query this metadata service to direct incoming API calls (e.g., GetItem, PutItem) to the correct storage node.
* The metadata service relies on an underlying storage sub-system that uses a consensus-based membership protocol to manage its own cluster state.

### 2. What went wrong (Trigger)

* A significant increase in the number of membership changes (nodes joining and leaving the fleet) within the metadata storage cluster caused the internal membership state data to grow unexpectedly large.
* The metadata service was required to process and replicate this expanded state information across all nodes in its cluster.
* The size of the state data exceeded the efficient processing threshold of the metadata service, leading to exponential increases in CPU utilization and latency for metadata lookups.

### 3. Cascading Effects and Impact

* Metadata lookup requests from front-end routers began to time out, causing the routers to return errors to users and dependent services.
* Increased latency triggered a "retry storm" from client SDKs and internal services, further saturating the already compromised metadata service.
* High-level AWS services with hard dependencies on DynamoDB, including AWS Lambda, Amazon CloudWatch, and Amazon EC2 Auto Scaling, experienced secondary failures or severe performance degradation.
* The incident resulted in a prolonged period of unavailability for DynamoDB tables in the US-EAST-1 region, impacting both the data plane (CRUD operations) and control plane (table management).

### 4. Resolution and Prevention

* Engineers manually increased the provisioned capacity of the metadata service and adjusted internal timeout settings to allow the system to process the backlog of state updates.
* The frequency of membership changes allowed within the metadata cluster was strictly limited to prevent future state-size bloat.
* The metadata service was re-architected into smaller, independent cells (cellular architecture) to ensure that a failure in one segment of the metadata service cannot impact the entire regional fleet.
* Introduced enhanced monitoring and alerting specifically for the size and propagation latency of internal metadata state.
* Refined the back-off and throttling algorithms in front-end routers to protect the metadata service from being overwhelmed during recovery phases.

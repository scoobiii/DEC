# Google Cloud Service Health
29 Jul 2022 14:00 PDT

Incident Report
---------------

**Summary:**

On Tuesday, 19 July 2022 at 06:33 US/Pacific, a simultaneous failure of multiple, redundant cooling systems in one of the data centers that hosts the zone europe-west2-a impacted multiple Google Cloud services. This resulted in some customers experiencing service unavailability for impacted products.

To our customers whose businesses were impacted during this outage, we sincerely apologize. This is not the level of quality and reliability we strive to offer you, and we are taking immediate steps (detailed in the **Remediation & Prevention** section below) to improve the region's resilience.

_**Regional Impact**_

A number of regional Google Cloud services experienced impact during this incident, despite regional services being designed to survive the failure of a single zone. Upon investigation we have found two key contributing factors which led to these regional impacts:

*   At the start of the incident, we inadvertently modified traffic routing for internal services to avoid all three zones in the europe-west2 region, rather than just the impacted europe-west2-a zone. We corrected this on 19 July 2022 at 12:35 US/Pacific.
    
*   Our regional storage services, including GCS and BigQuery, replicate customer data across multiple zones. Due to the regional traffic routing change, they were unable to access any replica for a number of storage objects. This prevented customers from reading these objects until the traffic routing was corrected, at which point access was immediately restored.
    

_**Cooling System Impact Duration**_

Google engineers powered down the data center that hosted a portion of the impacted zone europe-west2-a on Tuesday, 19 July 2022 at 10:05 US/Pacific while the cooling system was repaired. The cooling system was repaired at 14:13 US/Pacific. The total duration of the cooling system failure impact was 4 hours, 8 minutes.

_**Cloud Service Restoration Duration**_

Google engineers began service restoration once the cooling system was repaired on Tuesday, 19 July 2022 at 14:13 US/Pacific. Cloud services were restored to operation by Wednesday, 20 July 2022 at 04:28 US/Pacific.

The duration of impact on Cloud services spans the window from 10:05 US/Pacific on Tuesday, 19 July 2022 (when a portion of europe-west2-a was powered down), to 04:28 US/Pacific on Wednesday, 20 July 2022, when Cloud services were restored; a total of 18 hours, 23 minutes.

_**Long Tail Duration**_

After the initial restoration of service to the zone, a small number of Google Compute Engine instances required additional work by our engineers to restore them to normal operations. This manifested as unavailable instances in GCE, and unavailable instances in products and services that rely on Google Compute Engine, such as Cloud SQL. This was fully mitigated on Wednesday, 20 July 2022 at 21:20 US/Pacific and the incident was closed with all services restored.

For the instances in this long tail, impact duration spans the window from the initial power down (at 10:05 US/Pacific on Tuesday, 19 July 2022) to the eventual full mitigation (at 21:20 US/Pacific on Wednesday, 20 July 2022); a total of 35 hours, 15 minutes.

Google is conducting a detailed analysis of the systems and processes involved in both the cooling failure and the service recovery, with specific followup AIs identified below.

**Root Cause:**

One of the data centers that hosts zone europe-west2-a could not maintain a safe operating temperature due to a simultaneous failure of multiple, redundant cooling systems combined with the extraordinarily high outside temperatures. We powered down this part of the zone to prevent an even longer outage or damage to machines. This caused a partial failure of capacity in that zone, leading to instance terminations, service degradation, and networking issues for a subset of customers.

**Remediation and Prevention:**

Google engineers were alerted to an issue affecting two cooling systems in one of the data centers that hosts europe-west2-a on Tuesday, 19 July 2022 at 06:33 US/Pacific and began an investigation. Engineers were engaged at 07:02 and began assessing viable mitigations. At 10:05, our engineers decided to power down servers in the impacted data center within europe-west2-a to prevent an even longer outage and further impact to infrastructure in the zone.

The cooling system was repaired at 14:13, and we restored our services by Wednesday, 20 July 2022, at 04:28 US/Pacific. A small subset of customers experienced residual effects which were fully mitigated by 21:20.

Google is committed to preventing a future recurrence and improving recovery times by taking the following actions:

*   A small number of services experienced problems in zonal failover automation. We will repair and carefully re-test our failover automation to ensure stronger resilience in our failover protocols during large scale events such as this one.
*   We will investigate and develop more advanced methods to progressively decrease the thermal load within a single data center space, reducing the probability that a full shutdown is required.
*   Our initial recovery of impacted services once cooling was restored was 14 hours, 15 minutes. Additionally, the recovery of the long tail of impacted Google Compute Engine instances and related services was an additional 16 hours, 52 minutes. We are examining our procedures, tooling, and automated recovery systems for gaps to substantially improve our recovery times in the future.
*   Google engineers are actively conducting a detailed analysis of the cooling system failure that triggered this incident.
*   Google engineers will be conducting an audit of cooling system equipment and standards across the data centers that house Google Cloud globally.

**Detailed Description of Impact:**

On Tuesday, 19 July 2022 from 06:33 to Wednesday, 20 July 2022 21:20 US/Pacific, some customers may have experienced high latency or errors in multiple Google Cloud services in the impacted location as detailed below:

**Infrastructure Services**

*   **Google Compute Engine:**
    
    As data center temperatures started to increase, on Tuesday, 19 July 2022 at 08:06 US/Pacific Compute Engine terminated 42% of the Preemptible VMs (PVMs) across the europe-west2 region to reduce thermal load in zone europe-west2-a and ensure space for zonal failover activities in zones europe-west2-b and europe-west2-c.
    
    When we proceeded to power down the impacted data center to mitigate the cooling overload on Tuesday, 19 July 2022 at 10:07 US/Pacific, Compute Engine terminated all VMs in the impacted data center, representing approximately 35% of the VMs in the europe-west2-a zone. We were able to re-enable PVM launches in the running europe-west2-b and europe-west2-c zones at 13:50 US/Pacific.
    
    Power was restored on 19 July 2022 at 14:13 US/Pacific, at which point we began the recovery process for Compute Engine. To ensure a safe restoration, the team carefully sequenced the startup of the services powering Compute Engine. This process completed and the majority of VMs came back online starting at 20:18 US/Pacific. We enabled PVMs once the majority of the other VMs were running at 21:27 US/Pacific.The total impact duration was 12 hours, 12 minutes.
    
    A small number of VMs (approximately 0.6% of the europe-west2-a zone) encountered conflicts in our control plane state, requiring a manual reconciliation process. We completed this reconciliation for the “long tail” of all VMs on Thursday, 21 July 2022 at 02:32 US/Pacific. A small number of control plane requests to delete missing VMs during the incident required manual resolution, which was completed at 15:50 US/Pacific.
    
    _**Impact Mitigation time:**_ Thursday, 21 July 2022 20:18 US/Pacific
    
*   **Persistent Disk (PD):**
    
    Approximately 38% of Persistent Disk volumes in zone europe-west2-a were unavailable from Tuesday, 19 July 2022 09:13 US/Pacific. Affected customers would observe unresponsive disks or I/O errors. In most cases, the GCE instances using these volumes were terminated shortly afterward, but about 1% of the unavailable Persistent Disk volumes in zone europe-west2-a were attached to instances that remained online throughout the incident. Approximately 96% of Persistent Disk volumes recovered automatically by 20:30 US/Pacific, but the remainder required additional work to recover, which completed on Wednesday, 20 July 2022 03:10 US/Pacific.
    
    Additionally, about 11% of Regional Persistent Disk volumes in the europe-west2 region experienced high disk latency at the beginning of the incident. Most of the Regional Persistent Disk volumes successfully detected the fault in zone europe-west2-a and switched to unreplicated mode, but about 8% of Regional Persistent Disk volumes in the europe-west2 region were unable to correctly detect the fault until the Persistent Disk team forced them into unreplicated mode, which completed around 11:50 US/Pacific. About 17% of Regional Persistent Disk customers in the europe-west2 region experienced errors performing control plane operations, including creating, deleting, attaching, detaching, and snapshotting Regional Persistent Disk volumes from 19 July 2022 07:18 US/Pacific to 13:40 US/Pacific as an unintended side effect of mitigation efforts.
    
    As a result of mitigation efforts, 38% of customers were unable to create new Persistent Disk volumes in zone europe-west2-a from Tuesday, 19 July 2022 11:16 US/Pacific to 20 July 2022 02:56 US/Pacific. Additionally, 48% of customers were unable to create new Persistent Disk volumes in zones europe-west2-b and europe-west2-c from Tuesday, 19 July 2022 10:23 US/Pacific to 11:36 US/Pacific.
    
    The Persistent Disk snapshot service was unavailable for 38% of customers in zone europe-west2-a from Tuesday, 19 July 2022 from 08:11 US/Pacific to 21:29 US/Pacific. During this time, affected customers could not create new Persistent Disk snapshots from disks located in zone europe-west2-a nor restore snapshots and disk images to disks in zone europe-west2-a. The total impact duration was 12 hours, 16 minutes.
    
    _**Impact Mitigation time:**_ Wednesday, 20 July 2022 03:10 US/Pacific
    
*   **Google Cloud Storage:**
    
    On Tuesday, 19 July 2022 between 07:18 and 08:54 US/Pacific, ReadObject availability for buckets located in europe-west2 dropped to approximately 86%, and the entire region's availability to 96%. This impacted 24% of customer projects within this region. Customers would have received HTTP 500s when reading previously written data. All other operations, as well as read for new ingress, or data that was outside of the affected zones were not impacted. The total impact duration was 96 minutes.
    
    Google Cloud Storage (GCS) stores replicas in at least 2 independent colossus clusters. As a regional (not zonal) product, GCS leverages placement algorithms to ensure locations, network, and data center diversity when selecting colossus clusters \[1\]. There was a legacy placement algorithm isolated to just the europe-west2 region allowing for three colossus clusters in the region to be impacted by the same underlying issue relating to a single data center. In a small number of cases GCS had replicated regional data residing in two offline clusters, and subsequently some customers were unable to access and read some of their existing regional scoped data when the regional clusters were taken offline. Writes and other operations were not impacted.
    
    \[1\] - [https://cloud.google.com/blog/products/storage-data-transfer/a-peek-behind-colossus-googles-file-system](https://cloud.google.com/blog/products/storage-data-transfer/a-peek-behind-colossus-googles-file-system)
    
    _**Impact Mitigation time:**_ Tuesday, 19 July 2022 08:54 US/Pacific.
    

**Other Services**

*   **API Gateway:**
    
    ~47% of projects with traffic experienced an elevated number of spikes in 5xx responses in europe-west2 on Tuesday, 19 July 2022 from 07:20 to 14:00 US/Pacific. Affected customers observed 5xx responses from their API Gateways. The total impact duration was 6 hours, 40 minutes.
    
    _**Impact Mitigation time:**_ Tuesday, 19 July 2022 14:00 US/Pacific.
    
*   **Cloud Bigtable:**
    
    ~70% of unreplicated Bigtable instances in europe-west2-a experienced 100% data plane unavailability from Tuesday, 19 July 2022 07:05 to Wednesday, 20 July 2022 02:20 US/Pacific. We observed failed control plane operations for 11% of instances that contained a replica in the europe-west2-a zone. Customers with replicated bigtables in europe-west2-a/b/c using Multi-Cluster routing may have seen increased latencies due to traffic failing over to other regions. The total impact duration was 19 hours, 15 minutes.
    
    _**Impact Mitigation time:**_ Wednesday, 20 July 2022 02:20 US/Pacific
    
*   **Cloud Composer:**
    
    Cloud Composer environments (data-plane side) and operations (control-plane side) experienced degraded performance in europe-west2 on Tuesday, 19 July 2022 from 09:00 to 22:30 US/Pacific. Control plane operations experienced high failure rates (failure rate reached 100% at the peak). The number of Composer environments reporting a normal, operational state dropped by ~37%. Total capacity of all environments in the region (measured by the total number of tasks being executed in the whole region) was significantly reduced and the drop reached ~50% at the peak. The total impact duration was 13 hours, 30 minutes.
    
    _**Impact Mitigation time:**_ Tuesday, 19 July 2022 22:30 US/Pacific
    
*   **Cloud Data Fusion:**
    
    ~20% of Data Fusion Instances in europe-west2 experienced service degradation, ranging from logs and metrics not being updated to complete loss of availability of their instance on Tuesday, 19 July 2022 10:15 to 21:30 US/Pacific. ~5% of instances were usable during the impact duration. The total impact duration was 11 hours, 15 minutes.
    
    _**Impact Mitigation time:**_ Tuesday, 19 July 2022 21:30 US/Pacific
    
*   **Cloud Datastore:**
    
    Customers may have experienced timeouts and degraded service on approximately 1% of writes in the europe-west2 Datastore instance. The total impact duration was 9 hours, 25 minutes, on Tuesday, 19 July 2022 from approximately 12:30 to 19:30 US/Pacific.
    
    Approximately 40% of customers experienced significant latency increase on query operations from Tuesday, 19 July 2022 09:45 to 12:45 US/Pacific.
    
    _**Impact Mitigation time:**_ Tuesday, 19 July 2022 19:30 US/Pacific
    
*   **Cloud Dataproc:**
    
    On Tuesday, 19 July 2022, from 08:00 US/Pacific to 22:00 US/Pacific, customers experienced increased error rates in creating and scaling up clusters in europe-west2. About 24% of CREATE operations and 13% of UPDATE operations were affected. Due to a backlog of queued requests requiring manual attention, error rates for old CREATE/DELETE requests remained elevated until Thursday, 21 July 2022 16:45 US/Pacific (this did not impact availability of already-running clusters). Some Dataproc clusters that were allocated in the impacted zone were unavailable during power down. Most of these came up by 19 July 2022 22:00 US/Pacific. By this time, CREATE operations in all zones began working. A few existing clusters were impacted by the long tail recovery in Google Compute Engine and were restored when that effort completed on Thursday, 21 July 2022 02:32 US/Pacific.
    
    Primary impact duration was approximately 14 hours.
    
    _**Impact Mitigation time:**_ Tuesday, 19 July 2022 22:00 US/Pacific
    
*   **Cloud Firestore:**
    
    Firestore streaming (listen, write) requests via Webchannel were 100% unavailable in the europe-west2 instance on Tuesday, 19 July 2022 from 07:15 to 09:55 US/Pacific. The total impact duration was 2 hours, 40 minutes.
    
    _**Impact Mitigation time:**_ Tuesday, 19 July 2022 09:55 US/Pacific
    
*   **Cloud Secret Manager**
    
    Cloud Secret Manager experienced an outage for secrets that were exclusively stored in europe-west2 from Tuesday, 19 July 2022 07:29 to 08:46 US/Pacific.
    
    Secret Manager is a global service that lets users define in which regions to store a given secret. The regional service instances for europe-west2 were deployed in the impacted data center. The total impact duration was 1 hour, 17 minutes.
    
    _**Impact Mitigation time:**_ Tuesday, 19 July 2022 08:46 US/Pacific
    
*   **Cloud Spanner:**
    
    A more comprehensive investigation into our Cloud Spanner logs indicated no evidence of any customer impact. If you were impacted, please contact Google Cloud Support using [https://cloud.google.com/support](https://cloud.google.com/support), and we will review your logs.
    
*   **Cloud SQL:**
    
    Customers experienced downtime on Tuesday, 19 July 2022 starting at 09:25 US/Pacific. 36% of zonal (non-HA) instances in europe-west2-a were affected. Additionally, 31% of regional (HA) instances whose primaries were located in europe-west2-a experienced extended downtime because they were unable to successfully fail over to another zone. Finally, customers experienced some failures during the incident for backup, instance creation, update, delete, restart, export, and Database Migration Service operations. The total impact duration was 17 hours, 30 minutes.
    
    _**Impact Mitigation time:**_ Tuesday, 19 July 2022 21:00 US/Pacific
    
*   **Dataflow:**
    
    Approximately 8% of Dataflow streaming jobs running in europe-west2 were stuck from Tuesday, 19 July 2022 07:11 to 12:38 US/Pacific. There was limited impact to batch Dataflow jobs. Some new Dataflow jobs could not be initiated. The total impact duration was 5 hours, 27 minutes.
    
    _**Impact Mitigation time:**_ Tuesday, 19 July 2022 12:38 US/Pacific
    
*   **Datastream:**
    
    Datastream streams experienced errors and processing lag in europe-west2-a as a result of impact on the data-plane infrastructure and services from Tuesday, 19 July 2002 09:05 to Wednesday, 20 July 2022 03:00 US/Pacific. Customers were advised to run their streams in another region. The total impact duration was 17 hours, 55 minutes.
    
    _**Impact Mitigation time:**_ Wednesday, 20 July 2022 03:00 US/Pacific
    
*   **Google App Engine, Cloud Functions, and Cloud Run:**
    
    Customers may have experienced high error rates for Google App Engine, Cloud Functions and Cloud Run in europe-west2. Customers with a multi-region architecture could failover to another region.
    
    Traffic for ~72% of projects from Cloud Tasks, Cloud Scheduler, Eventarc, Cloud Pubsub to App Engine and Cloud Functions experienced up to 18% dropped requests/events on Tuesday, 19 July 2022 07:18 to 10:05 US/Pacific.
    
    Traffic from other sources (including end-users) to ~35% of App Engine, Cloud Functions, and Cloud Run projects experienced elevated latency in the same time period. Furthermore, ~5% of Cloud Run projects experienced elevated error rates (reaching peaks of 90% unavailability) caused by new instances failing to be created (some of which were due to a dependency on Google Cloud Secret Manager). The total impact duration was:
    
    *   App Engine Flexible: 13 hours, 1 minute
    *   App Engine Standard: 3 hours, 36 minutes
    *   Cloud Functions: 3 hours, 36 minutes
    *   Cloud Run: 3 hours, 36 minutes
    
    _**Impact Mitigation time:**_ Tuesday, 19 July 2022 10:05 US/Pacific (App Engine Standard, Cloud Functions, Cloud Run) and Tuesday, 19 July 2022 20:30 US/Pacific (App Engine Flexible)
    
*   **Google BigQuery:**
    
    On Tuesday, 19 July 2022 between 04:40 US/Pacific and 13:43 US/Pacific, 12% of the projects in europe-west2 experienced errors and dataset unavailability. This was caused by mitigations and the power down of one of the data centers serving the europe-west2-a zone, which reduced the storage and compute capacity available to BigQuery. As a regional service, BigQuery was able to mitigate some of this capacity loss by shifting load to other data centers. However, BigQuery datasets which happened to be hosted solely in that data center were unavailable during the power down.
    
    _**Impact Mitigation time:**_ Tuesday, 19 July 2022 13:43 US/Pacific
    
*   **Google Cloud Tasks:**
    
    Cloud Tasks automatically distributes projects across cloud zones within a region. 6% of projects in europe-west2 were loaded in the data center that was powered down on Tuesday, 19 July 2022 10:05 US/Pacific, and stopped delivering tasks until 13:24, after which delivery was resumed. No tasks were executed in any newly created queues in the region and in any existing queues in the impacted zone. However, as a regional service, Cloud Task was eventually able to mitigate all of the capacity loss by shifting load to other zones. The total impact duration was 3 hours, 19 minutes.
    
    _**Impact Mitigation time:**_ Tuesday, 19 July 2022 13:24 US/Pacific
    
*   **Google Cloud Scheduler:**
    
    Cloud Scheduler automatically distributes projects across Cloud zones within a region. 6% of projects in europe-west2 were loaded in the datacenter that was powered down on Tuesday, 19 July 2022 10:05 US/Pacific and stopped executing jobs until 13:24, after which execution was resumed. As a regional service, Cloud Scheduler was eventually able to mitigate all of the capacity loss by shifting load to other zones. The total impact duration was 3 hours, 19 minutes.
    
    _**Impact Mitigation time:**_ Tuesday, 19 July 2022 13:24 US/Pacific
    
*   **Cloud Filestore:**
    
    45% of instances in europe-west2-a experienced service unavailability from Tuesday, 19 July 2022 10:05 US/Pacific to Wednesday, 20 July 2022 01:10 US/Pacific. This impact lasted until the mitigation time. At this point customers with working instances experienced no additional issues. 6.4% of impacted instances did not recover automatically and had to be recovered manually, which completed on Wednesday, 20 July 18:32 US/Pacific. The symptoms of these varied from unavailability to degraded availability. Customers were advised to fail over to another region, if possible. The total impact duration was 10 hours, 30 minutes.
    
    _**Impact Mitigation time:**_ Wednesday, 20 July 2022 01:10 US/Pacific
    
*   **Google Kubernetes Engine:**
    
    15% of zonal clusters in europe-west2-a and 57% of regional & zonal cluster nodes in europe-west2-a were fully unavailable from Tuesday, 19 July 2022 from 09:30 to 21:30 US/Pacific. Regional cluster control planes remained available, but overall cluster health may have been impacted by node unavailability. Customers were encouraged to move their workloads to other regions if possible. The approximate impact duration was 12 hours. However, a few clusters experienced lag through the next day due to underlying GCE impact.
    
    _**Impact Mitigation time:**_ Tuesday, 19 July 2022 21:30 US/Pacific
    
*   **Looker:**
    
    Looker instances in europe-west2 were unavailable or experienced degraded performance starting Tuesday, July 19, 2022 09:33 through Wednesday, July 20, 2022 at 05:57 US/Pacific. Once temperatures inside the data center returned to safe levels, the Looker team restored file systems for impacted customers, and all Looker instances were back online on Wednesday, July 20, 2022 at 05:57 US/Pacific. The total impact duration was 20 hours, 24 minutes.
    
    _**Impact Mitigation time:**_ Wednesday, 20 July 2022 05:57 US/Pacific
    
*   **Managed Service for Microsoft Active Directory:**
    
    28.57% of the Managed Active Directory domains configured in the europe-west2 region were impacted during the incident. Starting July 19, 2022 09:20:00 US/Pacific, the impacted customers had one less domain controller (DC) serving incoming traffic. However, the domains continued to operate with reduced redundancy. Furthermore, periodic backups (12h frequency) were not performed for the impacted domains during the incident window. Subsequent backups were successful. The total impact duration was 12 hours, 30 minutes.
    
    _**Impact Mitigation time:**_ Tuesday, 19 July 2022 21:50 US/Pacific
    
*   **Memorystore for Memcached:**
    
    20.8% of instances in europe-west2 experienced degraded performance and availability issues resulting in a total cache flush. Impact start was Tuesday 19 July at 09:35 US/Pacific.All instances with VMs in europe-west2-a were unavailable or performed with degraded performance for the entire duration of the event. Instances that were only located in europe-west2-a were unavailable. Instances with some VMs in europe-west2-a lost their corresponding memcache availability (storage/access).All affected instances experienced a cache flush. DELETE, and RECREATE operations were unavailable for instances located in europe-west2-a. CREATE operations were not affected. UPDATE was not affected as there were no requests during that time but was theoretically unavailable. The total impact duration was 1 day, 5 hours, 27 minutes.
    
    _**Impact Mitigation time:**_ Wednesday, 20 July 2022 15:02 US/Pacific
    
*   **Memorystore for Redis:**
    
    ~10% of instances in europe-west2 experienced timeouts with the management API (create, update, delete, etc) from Tuesday, 19 July 2022 09:24 to Wednesday, 20 July 2022 21:19 US/Pacific. Customers whose instances are located in the affected data center in the europe-west2-a zone were unable to access their instances. Additionally, all affected instances experienced a cache flush. In total, ~39% of basic-tier instances in europe-west2-a were affected. The total impact duration was 11 hours, 11 minutes.
    
    _**Impact Mitigation time:**_ Wednesday, 20 July 2022 21:19 US/Pacific
    
*   **Vertex AI online prediction:**
    
    Vertex AI prediction showed an elevated error rate on Tuesday, 19 July 2022 from 10:00 to 15:11 US/Pacific for a total impact duration of 5 hours 11 minutes.
    
    _**Impact Mitigation time:**_ Tuesday, 19 July 2022 15:11 US/Pacific
    
*   **Virtual Private Cloud (VPC):**
    
    Approximately 35% of the VMs in the europe-west2-a zone were unreachable from Tuesday, 19 July 2022 10:06 US/ Pacific to 20:32 US/ Pacific. This includes all Cloud traffic into and out of 1275 / 3509 VMs in europe-west2-a . Both the control plane and the data plane were impacted. The total impact duration was 10 hours, 26 minutes.
    
    _**Impact Mitigation time:**_ Tuesday, 19 July 2022 20:32 US/Pacific
    

* * *

21 Jul 2022 12:12 PDT

This is a preliminary Incident Report (Mini-IR). A Full Incident Report with additional details is being prepared and will be posted at a later date.

We apologize for the inconvenience this service disruption/outage may have caused. We would like to provide some information about this incident below. Please note, this information is based on our best knowledge at the time of posting and is subject to change as our investigation continues. If you have experienced impact outside of what is listed below, please reach out to Google Cloud Support using [https://cloud.google.com/support](https://cloud.google.com/support) .

(All Times US/Pacific)

**Incident Start:** 2022-07-19 06:33

**Incident End:** 2022-07-20 21:20

**Duration:** 1 day, 14 hours, 47 minutes

**Regions/Zones:** europe-west2

**Description:**

On Tuesday, 19 July 2022, a partial cooling failure in one of the buildings that hosts the zone europe-west2-a impacted multiple Google Cloud services. This resulted in some customers experiencing service unavailability for impacted products. The cooling system impairment began at 04:30 PDT, and was fully restored at 15:28 PDT. Google engineers were engaged at 06:40 PDT after a rise in temperature was noted by automated monitoring systems, and took a number of steps to reduce thermal load in the datacenter before ultimately turning down services and powering off servers in the affected zone at 09:04 PDT due to temperature excursions. Engineering teams worked in shifts around the clock to restore services once temperatures inside the datacenter returned to acceptable levels, and inital restoration began at 14:13 PDT. Many services in the affected zone were available again by 20:24 PDT on 19 July 2022, and service restoration was complete on 20 July 2022 at 04:28 PDT. A small number of customers experienced residual effects which were fully mitigated by 21:20 PDT on 20 July 2022. Preliminary root cause has been identified as two separate chiller unit failures, coupled with high ambient weather conditions, at one of the buildings that hosts the europe-west2-a zone.

We sincerely apologize to our customers who were impacted by this service disruption. Google and its suppliers are conducting a detailed analysis of the cooling system failure which triggered this incident, and Google engineers will subsequently conduct an audit of cooling system equipment and standards across the data centers which house Google cloud zones, to ensure that the lessons learned from this incident are applied consistently at all locations.

**Customer Impact:**

*   **Google Cloud Storage:** Some customers with data replicated only in the impacted region experienced HTTP 500 errors. Customers with data replicated outside of the impacted region were not affected.
    
*   **Google BigQuery:** Some customers experienced dataset unavailability.
    
*   **Google App Engine and Cloud Functions:** Customers may have experienced high error rates for Google App Engine and Cloud Functions in europe-west2. Customers could failover to another region.
    
*   **Dataflow:** Some Dataflow streaming jobs running in the impacted area were stuck. Some new Dataflow jobs could not be initiated.
    
*   **Persistent Disk (PD):** Customers were unable to create PD devices in the impacted zone. Instances in the impacted zone were terminated, and therefore weren’t able to access their PD devices. A small number of Replicated PD volumes experienced delays in the failover to a healthy zone.
    
*   **API Gateway:** Customers saw an elevated number of clone exits and spikes in europe-west2.
    
*   **Cloud Spanner:** There was no impact for customers that were below our recommended CPU usage. Customers above the limit saw some latency impact. Some multi-region customers running in europe-west2 experienced increased latency.
    
*   **Google Cloud Tasks:** Customers experienced high latency for API requests. Some queues were not being loaded, impacting tasks. Customers may have also seen high delivery latency for their tasks, depending on which cell was serving their queues.
    
*   **Google Compute Engine:** Customers impacted by this issue would have experienced abnormal instance terminations for instances which were running in the impacted building. Affected customers who experienced an instance failure in europe-west2-a were advised to launch new instances in other zones of europe-west2.
    
*   **Vertex AI online prediction:** End-point Vertex AI prediction displayed time out errors for some customers.
    
*   **VPC (Traffic Virtnet):** VPCs in zone europe-west2-a were inaccessible. Customers also experienced 100% packet loss to zone europe-west2-a. Customers were unable to make any control plane changes in europe-west2-a.
    
*   **Cloud Firestore:** Firestore streaming (listen, write) requests via Webchannel were affected.
    
*   **Cloud Datastore:** Customers may have experienced timeouts and degraded service.
    
*   **Looker:** Customers hosted in europe-west2 experienced significantly increased failed requests.
    
*   **Cloud Composer:** Cloud Composer running environments and operations experienced degraded performance in europe-west2. Resource creation experienced high failure rates.
    
*   **Cloud Data Fusion:** Data Fusion Instances in europe-west2 were unavailable, and pipelines configured to run in europe-west2 were failing. Customers in europe-west2 were not able to create new pipelines.
    
*   **Managed Service for Microsoft Active Directory:** Customers were unable to perform any operations on Managed Active Directory domains which were single region (europe-west2). Customers also experienced a degraded experience if one domain controller remained unavailable due to zonal impact.
    
*   **Cloud Dataproc:** Customers were not able to create or scale up clusters in europe-west2-a. Some customers in europe-west2-c experienced elevated error rates during cluster creation and scale up. Customers were able to choose a different zone to create clusters.
    
*   **Cloud Bigtable:** Some customers in europe-west2 experienced service unavailability and elevated latency. Customer workloads using replicated databases with impacted replicas in europe-west2, could be moved to the region close to the other replicas to reduce latency.
    
*   **Datastream:** Datastream streams experienced errors and processing lag as a result of impact on the data-plane infrastructure and services. Customers were advised to run their streams in another region.
    
*   **Google Kubernetes Engine:** A proportion of clusters in europe-west2-a were fully unavailable, and nodepools for some regional clusters in europe-west2 were disrupted. Customers were encouraged to move their workloads to other regions if possible.
    
*   **Cloud Filestore:** A small number of customers in europe-west2-a may have experienced service unavailability. Customers with working instances experienced no issues. Customers were advised to fail over to another region, if possible.
    
*   **MemoryStore for Memcached:** Some existing instances in europe-west2-a were unavailable. Additionally, customers may have experienced degraded performance in europe-west2. Instance creation was not affected.
    
*   **MemoryStore for Redis:** Customers experienced timeouts with the management API (create, update, delete ,etc) and cache flushes.
    
*   **Cloud SQL:** Customers experienced operation failures for backups, creates, Database Migration Service operations, updates, deletes, restarts and exports. Cloud SQL experienced instance unavailability for several zonal instances in europe-west2. A number of HA instances also experienced failover issues in europe-west2.
    

20 Jul 2022 22:48 PDT

The issue with Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2 has been resolved for all affected users as of Wednesday, 2022-07-20 21:20 US/Pacific.

Services Mitigated: Google Cloud Storage, Google BigQuery, Google App Engine and Cloud Functions, Dataflow, Persistent Disk (PD), API Gateway, Cloud Spanner, Google Cloud Tasks, Google Compute Engine, Vertex AI online prediction, VPC (Traffic Virtnet), Cloud Firestore, Cloud Datastore, Looker, Cloud Composer, Cloud Data Fusion, Managed Service for Microsoft Active Directory, Cloud Dataproc, Bigtable, Datastream, GKE, Cloud Filestore, MemoryStore for Memcached, MemoryStore for Redis, Cloud SQL.

We thank you for your patience while we worked on resolving the issue.

20 Jul 2022 21:25 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is impacting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams have determined that further investigation is required to mitigate the issue. ETA to be determined.

We apologize to all who are affected by the disruption.

**Services Mitigated:** Google Cloud Storage, Google BigQuery, Google App Engine and Cloud Functions, Dataflow, Persistent Disk (PD), API Gateway, Cloud Spanner, Google Cloud Tasks, Google Compute Engine, Vertex AI online prediction, VPC (Traffic Virtnet), Cloud Filestore, Cloud Datastore, Looker, Cloud Composer, Cloud Data Fusion, Managed Service for Microsoft Active Directory, Cloud Dataproc, Bigtable, Datastream, GKE, Cloud Filestore, MemoryStore for Memcached

**Services Mitigation in progress:** MemoryStore for Redis, Cloud SQL.

**Product Impact:**

**Cloud Filestore:**

*   Impact/Diagnosis: A small number of customers in europe-west2-a may be experiencing service unavailability. No new issues are expected for customers with working instances.
*   Workaround: Customers are advised to fail over to another region, if possible.
*   Impact Mitigation: 07/20/22 09:18 US/Pacific

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management API (create, update, delete, etc.), and cache flushes.
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Impact Mitigation: 07/20/22 15:00 US/Pacific

**Cloud SQL:**

*   Impact/Diagnosis: 4 zonal instances are still impacted. Most non-HA instances backed by europe-west2-a are working again in europe-west2-a. And most HA instances in europe-west2 are operational again as of 19:00 US/Pacific.
*   Workaround: None at this time.

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a could be unavailable.
*   Workaround: Customers were encouraged to move their workloads to other regions if possible.
*   Impact Mitigation time: 07/20/22 07:57 US/Pacific

**Datastream:**

*   Impact/Diagnosis: Datastream Streams in that region might experience errors and lags which eventually could lead to lost position
*   Workaround: Customers were advised to run their Streams in another region.
*   Impact Mitigation time: 07/20/22 06:02 US/Pacific

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: Workloads using replicated databases with replicas in europe-west2, may be moved to the region close to the other replicas to reduce latency.
*   Impact Mitigation time: 07/20/22 04:51 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up could experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b, europe-west2-c or other regions.
*   Impact Mitigation time: 07/20/22 04:54 US/Pacific

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers were unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers could also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Impact Mitigation time: 07/19/22 22:15 US/Pacific

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 21:30 US/Pacific

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region could see elevated errors when attempting to read objects.
*   Impact Mitigation time: 07/19/22 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets were unavailable for the impacted locations.
*   Impact Mitigation time: 07/19/22 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers could experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Impact Mitigation time: 07/19/22 11:05 US/Pacific

**Dataflow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area might have been stuck. New Dataflow jobs could not start.
*   Impact Mitigation time: 07/19/22 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations experienced downgraded performance in europe-west2.
*   Impact Mitigation time: 07/19/22 22:30 US/Pacific

**Persistent Disk (PD):**

*   Impact/Diagnosis: Customers were unable to create any PD device in europe-west2. PD devices in europe-west2-a were unavailable
*   Workaround: Create PD devices in a different region/zone.
*   Impact Mitigation time: From 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c. From 07/20/22 04:51 US/Pacific all PDs should be healthy and back online.

**API Gateway:**

*   Impact/Diagnosis: Customers could experience elevated 5xx errors in europe-west2.
*   Impact Mitigation time: 07/19/22 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers could not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region or 45% for Multi Region to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max). Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can choose to route to europe-west1, no workarounds needed for europe-west2
*   Impact Mitigation time: 07/19/22 17:00 US/Pacific

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 experienced significantly increased failed requests.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/20/22 02:55 AM US/Pacific

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers experienced high latency for all API requests. Some queues were not being loaded and they have stopped executing tasks. Customers could also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Impact Mitigation time: 07/19/22 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH).
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Impact Mitigation time: 07/19/22 20:22 US/Pacific

**Vertex AI online prediction**

*   Impact/Diagnosis: End user could experience timeouts
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 15:11 US/Pacific

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a were inaccessible. Customers could also experience a 100% packet loss to europe-west2-a. Customers were unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 12:49 US/Pacific

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming experienced Listen and Write outage starting around 07:15 US/Pacific.
*   Impact Mitigation time: 07/19/22 09:55 US/Pacific

**Cloud Datastore:**

*   Impact/Diagnosis: Service was degraded. Customers may have experienced timeouts.
*   Impact Mitigation time: 07/19/22 19:30 US/Pacific

We will provide more information by Wednesday, 2022-07-20 23:30 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

20 Jul 2022 21:23 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is impacting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams have determined that further investigation is required to mitigate the issue. ETA to be determined.

We apologize to all who are affected by the disruption.

**Services Mitigated:** Google Cloud Storage, Google BigQuery, Google App Engine and Cloud Functions, Dataflow, Persistent Disk (PD), API Gateway, Cloud Spanner, Google Cloud Tasks, Google Compute Engine, Vertex AI online prediction, VPC (Traffic Virtnet), Cloud Filestore, Cloud Datastore, Looker, Cloud Composer, Cloud Data Fusion, Managed Service for Microsoft Active Directory, Cloud Dataproc, Bigtable, Datastream, GKE, Cloud Filestore, MemoryStore for Memcached

**Services Mitigation in progress:** MemoryStore for Redis, Cloud SQL.

**Product Impact:**

**Cloud Filestore:**

*   Impact/Diagnosis: A small number of customers in europe-west2-a may be experiencing service unavailability. No new issues are expected for customers with working instances.
*   Workaround: Customers are advised to fail over to another region, if possible.
*   Impact Mitigation: 07/20/22 09:18 US/Pacific

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management API (create, update, delete, etc.), and cache flushes.
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Impact Mitigation: 07/20/22 15:00 US/Pacific

**Cloud SQL:**

*   Impact/Diagnosis: 4 zonal instances are still impacted. Most non-HA instances backed by europe-west2-a are working again in europe-west2-a. And most HA instances in europe-west2 are operational again as of 19:00 US/Pacific.
*   Workaround: None at this time.

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a could be unavailable.
*   Workaround: Customers were encouraged to move their workloads to other regions if possible.
*   Impact Mitigation time: 07/20/22 07:57 US/Pacific

**Datastream:**

*   Impact/Diagnosis: Datastream Streams in that region might experience errors and lags which eventually could lead to lost position
*   Workaround: Customers were advised to run their Streams in another region.
*   Impact Mitigation time: 07/20/22 06:02 US/Pacific

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: Workloads using replicated databases with replicas in europe-west2, may be moved to the region close to the other replicas to reduce latency.
*   Impact Mitigation time: 07/20/22 04:51 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up could experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b, europe-west2-c or other regions.
*   Impact Mitigation time: 07/20/22 04:54 US/Pacific

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers were unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers could also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Impact Mitigation time: 07/19/22 22:15 US/Pacific

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 21:30 US/Pacific

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region could see elevated errors when attempting to read objects.
*   Impact Mitigation time: 07/19/22 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets were unavailable for the impacted locations.
*   Impact Mitigation time: 07/19/22 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers could experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Impact Mitigation time: 07/19/22 11:05 US/Pacific

**Dataflow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area might have been stuck. New Dataflow jobs could not start.
*   Impact Mitigation time: 07/19/22 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations experienced downgraded performance in europe-west2.
*   Impact Mitigation time: 07/19/22 22:30 US/Pacific

**Persistent Disk (PD):**

*   Impact/Diagnosis: Customers were unable to create any PD device in europe-west2. PD devices in europe-west2-a were unavailable
*   Workaround: Create PD devices in a different region/zone.
*   Impact Mitigation time: From 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c. From 07/20/22 04:51 US/Pacific all PDs should be healthy and back online.

**API Gateway:**

*   Impact/Diagnosis: Customers could experience elevated 5xx errors in europe-west2.
*   Impact Mitigation time: 07/19/22 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers could not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region or 45% for Multi Region to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max). Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can choose to route to europe-west1, no workarounds needed for europe-west2
*   Impact Mitigation time: 07/19/22 17:00 US/Pacific

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 experienced significantly increased failed requests.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/20/22 02:55 AM US/Pacific

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers experienced high latency for all API requests. Some queues were not being loaded and they have stopped executing tasks. Customers could also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Impact Mitigation time: 07/19/22 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH).
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Impact Mitigation time: 07/19/22 20:22 US/Pacific

**Vertex AI online prediction**

*   Impact/Diagnosis: End user could experience timeouts
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 15:11 US/Pacific

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a were inaccessible. Customers could also experience a 100% packet loss to europe-west2-a. Customers were unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 12:49 US/Pacific

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming experienced Listen and Write outage starting around 07:15 US/Pacific.
*   Impact Mitigation time: 07/19/22 09:55 US/Pacific

**Cloud Datastore:**

*   Impact/Diagnosis: Service was degraded. Customers may have experienced timeouts.
*   Impact Mitigation time: 07/19/22 19:30 US/Pacific

We will provide more information by Wednesday, 2022-07-20 21:30 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

20 Jul 2022 19:26 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is impacting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams have determined that further investigation is required to mitigate the issue. ETA to be determined.

We apologize to all who are affected by the disruption.

**Services Mitigated:** Google Cloud Storage, Google BigQuery, Google App Engine and Cloud Functions, Dataflow, Persistent Disk (PD), API Gateway, Cloud Spanner, Google Cloud Tasks, Google Compute Engine, Vertex AI online prediction, VPC (Traffic Virtnet), Cloud Filestore, Cloud Datastore, Looker, Cloud Composer, Cloud Data Fusion, Managed Service for Microsoft Active Directory, Cloud Dataproc, Bigtable, Datastream, GKE, Cloud Filestore, MemoryStore for Memcached

**Services Mitigation in progress:** MemoryStore for Redis, Cloud SQL.

**Product Impact:**

**Cloud Filestore:**

*   Impact/Diagnosis: A small number of customers in europe-west2-a may be experiencing service unavailability. No new issues are expected for customers with working instances.
*   Workaround: Customers are advised to fail over to another region, if possible.
*   Impact Mitigation: 07/20/22 09:18 US/Pacific

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management API (create, update, delete, etc.), and cache flushes.
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.
*   Impact Mitigation: 07/20/22 15:00 US/Pacific

**Cloud SQL:**

*   Impact/Diagnosis: 4 zonal instances are still impacted. Most non-HA instances backed by europe-west2-a are working again in europe-west2-a. And most HA instances in europe-west2 are operational again as of 19:00 US/Pacific.
*   Workaround: None at this time.

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a could be unavailable.
*   Workaround: Customers were encouraged to move their workloads to other regions if possible.
*   Impact Mitigation time: 07/20/22 07:57 US/Pacific

**Datastream:**

*   Impact/Diagnosis: Datastream Streams in that region might experience errors and lags which eventually could lead to lost position
*   Workaround: Customers were advised to run their Streams in another region.
*   Impact Mitigation time: 07/20/22 06:02 US/Pacific

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: Workloads using replicated databases with replicas in europe-west2, may be moved to the region close to the other replicas to reduce latency.
*   Impact Mitigation time: 07/20/22 04:51 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up could experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b, europe-west2-c or other regions.
*   Impact Mitigation time: 07/20/22 04:54 US/Pacific

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers were unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers could also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 22:15 US/Pacific

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 21:30 US/Pacific

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region could see elevated errors when attempting to read objects.
*   Impact Mitigation time: 07/19/22 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets were unavailable for the impacted locations.
*   Impact Mitigation time: 07/19/22 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers could experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Impact Mitigation time: 07/19/22 11:05 US/Pacific

**Dataflow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area might have been stuck. New Dataflow jobs could not start.
*   Impact Mitigation time: 07/19/22 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations experienced downgraded performance in europe-west2.
*   Impact Mitigation time: 07/19/22 22:30 US/Pacific

**Persistent Disk (PD):**

*   Impact/Diagnosis: Customers were unable to create any PD device in europe-west2. PD devices in europe-west2-a were unavailable
*   Workaround: Create PD devices in a different region/zone.
*   Impact Mitigation time: From 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c. From 07/20/22 04:51 US/Pacific all PDs should be healthy and back online.

**API Gateway:**

*   Impact/Diagnosis: Customers could experience elevated 5xx errors in europe-west2.
*   Impact Mitigation time: 07/19/22 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers could not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region or 45% for Multi Region to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max). Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can choose to route to europe-west1, no workarounds needed for europe-west2
*   Impact Mitigation time: 07/19/22 17:00 US/Pacific

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 experienced significantly increased failed requests.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/20/22 02:55 AM US/Pacific

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers experienced high latency for all API requests. Some queues were not being loaded and they have stopped executing tasks. Customers could also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Impact Mitigation time: 07/19/22 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH).
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Impact Mitigation time: 07/19/22 20:22 US/Pacific

**Vertex AI online prediction**

*   Impact/Diagnosis: End user could experience timeouts
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 15:11 US/Pacific

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a were inaccessible. Customers could also experience a 100% packet loss to europe-west2-a. Customers were unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 12:49 US/Pacific

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming experienced Listen and Write outage starting around 07:15 US/Pacific.
*   Impact Mitigation time: 07/19/22 09:55 US/Pacific

**Cloud Datastore:**

*   Impact/Diagnosis: Service was degraded. Customers may have experienced timeouts.
*   Impact Mitigation time: 07/19/22 19:30 US/Pacific

We will provide more information by Wednesday, 2022-07-20 21:30 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

20 Jul 2022 18:26 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is impacting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams have determined that further investigation is required to mitigate the issue. ETA to be determined.

We apologize to all who are affected by the disruption.

**Services Mitigated:** Google Cloud Storage, Google BigQuery, Google App Engine and Cloud Functions, Dataflow, Persistent Disk (PD), API Gateway, Cloud Spanner, Google Cloud Tasks, Google Compute Engine, Vertex AI online prediction, VPC (Traffic Virtnet), Cloud Filestore, Cloud Datastore, Looker, Cloud Composer, Cloud Data Fusion, Managed Service for Microsoft Active Directory, Cloud Dataproc, Bigtable, Datastream, GKE, Cloud Filestore, MemoryStore for Memcached

**Services Mitigation in progress:** MemoryStore for Redis, Cloud SQL.

**Product Impact:**

**Cloud Filestore:**

*   Impact/Diagnosis: A small number of customers in europe-west2-a may be experiencing service unavailability. No new issues are expected for customers with working instances.
*   Workaround: Customers are advised to fail over to another region, if possible.
*   Impact Mitigation: 07/20/22 09:18 US/Pacific

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management API (create, update, delete, etc.), and cache flushes.
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.
*   Impact Mitigation: 07/20/22 15:00 US/Pacific

**Cloud SQL:**

*   Impact/Diagnosis: 4 zonal instances are still impacted. Most non-HA instances backed by europe-west2-a are working again in europe-west2-a. And most HA instances in europe-west2 are operational again as of 19:00 US/Pacific.
*   Workaround: None at this time.

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a could be unavailable.
*   Workaround: Customers were encouraged to move their workloads to other regions if possible.
*   Impact Mitigation time: 07/20/22 07:57 US/Pacific

**Datastream:**

*   Impact/Diagnosis: Datastream Streams in that region might experience errors and lags which eventually could lead to lost position
*   Workaround: Customers were advised to run their Streams in another region.
*   Impact Mitigation time: 07/20/22 06:02 US/Pacific

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: Workloads using replicated databases with replicas in europe-west2, may be moved to the region close to the other replicas to reduce latency.
*   Impact Mitigation time: 07/20/22 04:51 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up could experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b, europe-west2-c or other regions.
*   Impact Mitigation time: 07/20/22 04:54 US/Pacific

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers were unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers could also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 22:15 US/Pacific

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 21:30 US/Pacific

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region could see elevated errors when attempting to read objects.
*   Impact Mitigation time: 07/19/22 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets were unavailable for the impacted locations.
*   Impact Mitigation time: 07/19/22 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers could experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Impact Mitigation time: 07/19/22 11:05 US/Pacific

**Dataflow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area might have been stuck. New Dataflow jobs could not start.
*   Impact Mitigation time: 07/19/22 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations experienced downgraded performance in europe-west2.
*   Impact Mitigation time: 07/19/22 22:30 US/Pacific

**Persistent Disk (PD):**

*   Impact/Diagnosis: Customers were unable to create any PD device in europe-west2. PD devices in europe-west2-a were unavailable
*   Workaround: Create PD devices in a different region/zone.
*   Impact Mitigation time: From 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c. From 07/20/22 04:51 US/Pacific all PDs should be healthy and back online.

**API Gateway:**

*   Impact/Diagnosis: Customers could experience elevated 5xx errors in europe-west2.
*   Impact Mitigation time: 07/19/22 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers could not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region or 45% for Multi Region to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max). Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can choose to route to europe-west1, no workarounds needed for europe-west2
*   Impact Mitigation time: 07/19/22 17:00 US/Pacific

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 experienced significantly increased failed requests.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/20/22 02:55 AM US/Pacific

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers experienced high latency for all API requests. Some queues were not being loaded and they have stopped executing tasks. Customers could also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Impact Mitigation time: 07/19/22 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH).
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Impact Mitigation time: 07/19/22 20:22 US/Pacific

**Vertex AI online prediction**

*   Impact/Diagnosis: End user could experience timeouts
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 15:11 US/Pacific

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a were inaccessible. Customers could also experience a 100% packet loss to europe-west2-a. Customers were unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 12:49 US/Pacific

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming experienced Listen and Write outage starting around 07:15 US/Pacific.
*   Impact Mitigation time: 07/19/22 09:55 US/Pacific

**Cloud Datastore:**

*   Impact/Diagnosis: Service was degraded. Customers may have experienced timeouts.
*   Impact Mitigation time: 07/19/22 19:30 US/Pacific

We will provide more information by Wednesday, 2022-07-20 19:30 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

20 Jul 2022 17:30 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is impacting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams have determined that further investigation is required to mitigate the issue. ETA to be determined.

We apologize to all who are affected by the disruption.

**Services Mitigated:** Google Cloud Storage, Google BigQuery, Google App Engine and Cloud Functions, Dataflow, Persistent Disk (PD), API Gateway, Cloud Spanner, Google Cloud Tasks, Google Compute Engine, Vertex AI online prediction, VPC (Traffic Virtnet), Cloud Filestore, Cloud Datastore, Looker, Cloud Composer, Cloud Data Fusion, Managed Service for Microsoft Active Directory, Cloud Dataproc, Bigtable, Datastream, GKE, Cloud Filestore, MemoryStore for Memcached

**Services Mitigation in progress:** MemoryStore for Redis, Cloud SQL.

**Product Impact:**

**Cloud Filestore:**

*   Impact/Diagnosis: A small number of customers in europe-west2-a may be experiencing service unavailability. No new issues are expected for customers with working instances.
*   Workaround: Customers are advised to fail over to another region, if possible.
*   Impact Mitigation: 07/20/22 09:18 US/Pacific

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management API (create, update, delete, etc.), and cache flushes.
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.
*   Impact Mitigation: 07/20/22 15:00 US/Pacific

**Cloud SQL:**

*   Impact/Diagnosis: 4 zonal instances are still impacted. Most non-HA instances backed by europe-west2-a are working again in europe-west2-a. And most HA instances in europe-west2 are operational again as of 19:00 US/Pacific.
*   Workaround: None at this time.

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a could be unavailable.
*   Workaround: Customers were encouraged to move their workloads to other regions if possible.
*   Impact Mitigation time: 07/20/22 07:57 US/Pacific

**Datastream:**

*   Impact/Diagnosis: Datastream Streams in that region might experience errors and lags which eventually could lead to lost position
*   Workaround: Customers were advised to run their Streams in another region.
*   Impact Mitigation time: 07/20/22 06:02 US/Pacific

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: Workloads using replicated databases with replicas in europe-west2, may be moved to the region close to the other replicas to reduce latency.
*   Impact Mitigation time: 07/20/22 04:51 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up could experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b, europe-west2-c or other regions.
*   Impact Mitigation time: 07/20/22 04:54 US/Pacific

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers were unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers could also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 22:15 US/Pacific

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 21:30 US/Pacific

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region could see elevated errors when attempting to read objects.
*   Impact Mitigation time: 07/19/22 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets were unavailable for the impacted locations.
*   Impact Mitigation time: 07/19/22 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers could experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Impact Mitigation time: 07/19/22 11:05 US/Pacific

**Dataflow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area might have been stuck. New Dataflow jobs could not start.
*   Impact Mitigation time: 07/19/22 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations experienced downgraded performance in europe-west2.
*   Impact Mitigation time: 07/19/22 22:30 US/Pacific

**Persistent Disk (PD):**

*   Impact/Diagnosis: Customers were unable to create any PD device in europe-west2. PD devices in europe-west2-a were unavailable
*   Workaround: Create PD devices in a different region/zone.
*   Impact Mitigation time: From 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c. From 07/20/22 04:51 US/Pacific all PDs should be healthy and back online.

**API Gateway:**

*   Impact/Diagnosis: Customers could experience elevated 5xx errors in europe-west2.
*   Impact Mitigation time: 07/19/22 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers could not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region or 45% for Multi Region to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max). Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can choose to route to europe-west1, no workarounds needed for europe-west2
*   Impact Mitigation time: 07/19/22 17:00 US/Pacific

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 experienced significantly increased failed requests.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/20/22 02:55 AM US/Pacific

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers experienced high latency for all API requests. Some queues were not being loaded and they have stopped executing tasks. Customers could also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Impact Mitigation time: 07/19/22 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH).
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Impact Mitigation time: 07/19/22 20:22 US/Pacific

**Vertex AI online prediction**

*   Impact/Diagnosis: End user could experience timeouts
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 15:11 US/Pacific

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a were inaccessible. Customers could also experience a 100% packet loss to europe-west2-a. Customers were unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 12:49 US/Pacific

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming experienced Listen and Write outage starting around 07:15 US/Pacific.
*   Impact Mitigation time: 07/19/22 09:55 US/Pacific

**Cloud Datastore:**

*   Impact/Diagnosis: Service was degraded. Customers may have experienced timeouts.
*   Impact Mitigation time: 07/19/22 19:30 US/Pacific

We will provide more information by Wednesday, 2022-07-20 18:30 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

20 Jul 2022 16:26 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is impacting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams continue to work on service restoration. ETA to be determined.

We apologize to all who are affected by the disruption.

**Services Mitigated:** Google Cloud Storage, Google BigQuery, Google App Engine and Cloud Functions, Dataflow, Persistent Disk (PD), API Gateway, Cloud Spanner, Google Cloud Tasks, Google Compute Engine, Vertex AI online prediction, VPC (Traffic Virtnet), Cloud Filestore, Cloud Datastore, Looker, Cloud Composer, Cloud Data Fusion, Managed Service for Microsoft Active Directory, Cloud Dataproc, Bigtable, Datastream, GKE, Cloud Filestore, MemoryStore for Memcached

**Services Mitigation in progress:** MemoryStore for Redis, Cloud SQL.

**Product Impact:**

**Cloud Filestore:**

*   Impact/Diagnosis: A small number of customers in europe-west2-a may be experiencing service unavailability. No new issues are expected for customers with working instances.
*   Workaround: Customers are advised to fail over to another region, if possible.
*   Impact Mitigation: 07/20/22 09:18 US/Pacific

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management API (create, update, delete, etc.), and cache flushes.
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.
*   Impact Mitigation: 07/20/22 15:00 US/Pacific

**Cloud SQL:**

*   Impact/Diagnosis: 4 zonal instances are still impacted. Most non-HA instances backed by europe-west2-a are working again in europe-west2-a. And most HA instances in europe-west2 are operational again as of 19:00 US/Pacific.
*   Workaround: None at this time.

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a could be unavailable.
*   Workaround: Customers were encouraged to move their workloads to other regions if possible.
*   Impact Mitigation time: 07/20/22 07:57 US/Pacific

**Datastream:**

*   Impact/Diagnosis: Datastream Streams in that region might experience errors and lags which eventually could lead to lost position
*   Workaround: Customers were advised to run their Streams in another region.
*   Impact Mitigation time: 07/20/22 06:02 US/Pacific

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: Workloads using replicated databases with replicas in europe-west2, may be moved to the region close to the other replicas to reduce latency.
*   Impact Mitigation time: 07/20/22 04:51 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up could experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b, europe-west2-c or other regions.
*   Impact Mitigation time: 07/20/22 04:54 US/Pacific

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers were unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers could also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 22:15 US/Pacific

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 21:30 US/Pacific

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region could see elevated errors when attempting to read objects.
*   Impact Mitigation time: 07/19/22 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets were unavailable for the impacted locations.
*   Impact Mitigation time: 07/19/22 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers could experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Impact Mitigation time: 07/19/22 11:05 US/Pacific

**Dataflow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area might have been stuck. New Dataflow jobs could not start.
*   Impact Mitigation time: 07/19/22 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations experienced downgraded performance in europe-west2.
*   Impact Mitigation time: 07/19/22 22:30 US/Pacific

**Persistent Disk (PD):**

*   Impact/Diagnosis: Customers were unable to create any PD device in europe-west2. PD devices in europe-west2-a were unavailable
*   Workaround: Create PD devices in a different region/zone.
*   Impact Mitigation time: From 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c. From 07/20/22 04:51 US/Pacific all PDs should be healthy and back online.

**API Gateway:**

*   Impact/Diagnosis: Customers could experience elevated 5xx errors in europe-west2.
*   Impact Mitigation time: 07/19/22 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers could not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region or 45% for Multi Region to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max). Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can choose to route to europe-west1, no workarounds needed for europe-west2
*   Impact Mitigation time: 07/19/22 17:00 US/Pacific

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 experienced significantly increased failed requests.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/20/22 02:55 AM US/Pacific

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers experienced high latency for all API requests. Some queues were not being loaded and they have stopped executing tasks. Customers could also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Impact Mitigation time: 07/19/22 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH).
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Impact Mitigation time: 07/19/22 20:22 US/Pacific

**Vertex AI online prediction**

*   Impact/Diagnosis: End user could experience timeouts
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 15:11 US/Pacific

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a were inaccessible. Customers could also experience a 100% packet loss to europe-west2-a. Customers were unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 12:49 US/Pacific

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming experienced Listen and Write outage starting around 07:15 US/Pacific.
*   Impact Mitigation time: 07/19/22 09:55 US/Pacific

**Cloud Datastore:**

*   Impact/Diagnosis: Service was degraded. Customers may have experienced timeouts.
*   Impact Mitigation time: 07/19/22 19:30 US/Pacific

We will provide more information by Wednesday, 2022-07-20 17:30 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

20 Jul 2022 15:24 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is impacting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams are working on restoring their services. ETA to be determined.

We apologize to all who are affected by the disruption.

**Services Mitigated:** Google Cloud Storage, Google BigQuery, Google App Engine and Cloud Functions, Dataflow, Persistent Disk (PD), API Gateway, Cloud Spanner, Google Cloud Tasks, Google Compute Engine, Vertex AI online prediction, VPC (Traffic Virtnet), Cloud Filestore, Cloud Datastore, Looker, Cloud Composer, Cloud Data Fusion, Managed Service for Microsoft Active Directory, Cloud Dataproc, Bigtable, Datastream, GKE, Cloud Filestore, MemoryStore for Memcached

**Services Mitigation in progress:** MemoryStore for Redis, Cloud SQL.

**Product Impact:**

**Cloud Filestore:**

*   Impact/Diagnosis: A small number of customers in europe-west2-a may be experiencing service unavailability. No new issues are expected for customers with working instances.
*   Workaround: Customers are advised to fail over to another region, if possible.
*   Impact Mitigation: 07/20/22 09:18 US/Pacific

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management API (create, update, delete, etc.), and cache flushes.
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.
*   Impact Mitigation: 07/20/22 15:00 US/Pacific

**Cloud SQL:**

*   Impact/Diagnosis: 4 zonal instances are still impacted. Most non-HA instances backed by europe-west2-a are working again in europe-west2-a. And most HA instances in europe-west2 are operational again as of 19:00 US/Pacific.
*   Workaround: None at this time.

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a could be unavailable.
*   Workaround: Customers were encouraged to move their workloads to other regions if possible.
*   Impact Mitigation time: 07/20/22 07:57 US/Pacific

**Datastream:**

*   Impact/Diagnosis: Datastream Streams in that region might experience errors and lags which eventually could lead to lost position
*   Workaround: Customers were advised to run their Streams in another region.
*   Impact Mitigation time: 07/20/22 06:02 US/Pacific

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: Workloads using replicated databases with replicas in europe-west2, may be moved to the region close to the other replicas to reduce latency.
*   Impact Mitigation time: 07/20/22 04:51 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up could experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b, europe-west2-c or other regions.
*   Impact Mitigation time: 07/20/22 04:54 US/Pacific

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers were unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers could also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 22:15 US/Pacific

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 21:30 US/Pacific

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region could see elevated errors when attempting to read objects.
*   Impact Mitigation time: 07/19/22 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets were unavailable for the impacted locations.
*   Impact Mitigation time: 07/19/22 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers could experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Impact Mitigation time: 07/19/22 11:05 US/Pacific

**Dataflow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area might have been stuck. New Dataflow jobs could not start.
*   Impact Mitigation time: 07/19/22 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations experienced downgraded performance in europe-west2.
*   Impact Mitigation time: 07/19/22 22:30 US/Pacific

**Persistent Disk (PD):**

*   Impact/Diagnosis: Customers were unable to create any PD device in europe-west2. PD devices in europe-west2-a were unavailable
*   Workaround: Create PD devices in a different region/zone.
*   Impact Mitigation time: From 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c. From 07/20/22 04:51 US/Pacific all PDs should be healthy and back online.

**API Gateway:**

*   Impact/Diagnosis: Customers could experience elevated 5xx errors in europe-west2.
*   Impact Mitigation time: 07/19/22 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers could not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region or 45% for Multi Region to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max). Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can choose to route to europe-west1, no workarounds needed for europe-west2
*   Impact Mitigation time: 07/19/22 17:00 US/Pacific

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 experienced significantly increased failed requests.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/20/22 02:55 AM US/Pacific

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers experienced high latency for all API requests. Some queues were not being loaded and they have stopped executing tasks. Customers could also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Impact Mitigation time: 07/19/22 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH).
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Impact Mitigation time: 07/19/22 20:22 US/Pacific

**Vertex AI online prediction**

*   Impact/Diagnosis: End user could experience timeouts
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 15:11 US/Pacific

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a were inaccessible. Customers could also experience a 100% packet loss to europe-west2-a. Customers were unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 12:49 US/Pacific

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming experienced Listen and Write outage starting around 07:15 US/Pacific.
*   Impact Mitigation time: 07/19/22 09:55 US/Pacific

**Cloud Datastore:**

*   Impact/Diagnosis: Service was degraded. Customers may have experienced timeouts.
*   Impact Mitigation time: 07/19/22 19:30 US/Pacific

We will provide more information by Wednesday, 2022-07-20 16:30 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

20 Jul 2022 13:30 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is impacting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams are working on restoring their services. ETA to be determined.

We apologize to all who are affected by the disruption.

**Services Mitigated:** Google Cloud Storage, Google BigQuery, Google App Engine and Cloud Functions, Dataflow, Persistent Disk (PD), API Gateway, Cloud Spanner, Google Cloud Tasks, Google Compute Engine, Vertex AI online prediction, VPC (Traffic Virtnet), Cloud Filestore, Cloud Datastore, Looker, Cloud Composer, Cloud Data Fusion, Managed Service for Microsoft Active Directory, Cloud Dataproc, Bigtable, Datastream, GKE, Cloud Filestore,

**Services Mitigation in progress:** MemoryStore for Memcached, MemoryStore for Redis, Cloud SQL.

**Product Impact:**

**Cloud Filestore:**

*   Impact/Diagnosis: A small number of customers in europe-west2-a may be experiencing service unavailability. No new issues are expected for customers with working instances.
*   Workaround: Customers are advised to fail over to another region, if possible.
*   Impact Mitigation: 07/20/22 09:18 US/Pacific

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management API (create, update, delete, etc.), and cache flushes.
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.

**Cloud SQL:**

*   Impact/Diagnosis: 4 zonal instances are still impacted. Most non-HA instances backed by europe-west2-a are working again in europe-west2-a. And most HA instances in europe-west2 are operational again as of 19:00 US/Pacific.
*   Workaround: None at this time.

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a could be unavailable.
*   Workaround: Customers were encouraged to move their workloads to other regions if possible.
*   Impact Mitigation time: 07/20/22 07:57 US/Pacific

**Datastream:**

*   Impact/Diagnosis: Datastream Streams in that region might experience errors and lags which eventually could lead to lost position
*   Workaround: Customers were advised to run their Streams in another region.
*   Impact Mitigation time: 07/20/22 06:02 US/Pacific

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: Workloads using replicated databases with replicas in europe-west2, may be moved to the region close to the other replicas to reduce latency.
*   Impact Mitigation time: 07/20/22 04:51 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up could experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b, europe-west2-c or other regions.
*   Impact Mitigation time: 07/20/22 04:54 US/Pacific

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers were unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers could also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 22:15 US/Pacific

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 21:30 US/Pacific

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region could see elevated errors when attempting to read objects.
*   Impact Mitigation time: 07/19/22 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets were unavailable for the impacted locations.
*   Impact Mitigation time: 07/19/22 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers could experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Impact Mitigation time: 07/19/22 11:05 US/Pacific

**Dataflow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area might have been stuck. New Dataflow jobs could not start.
*   Impact Mitigation time: 07/19/22 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations experienced downgraded performance in europe-west2.
*   Impact Mitigation time: 07/19/22 22:30 US/Pacific

**Persistent Disk (PD):**

*   Impact/Diagnosis: Customers were unable to create any PD device in europe-west2. PD devices in europe-west2-a were unavailable
*   Workaround: Create PD devices in a different region/zone.
*   Impact Mitigation time: From 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c. From 07/20/22 04:51 US/Pacific all PDs should be healthy and back online.

**API Gateway:**

*   Impact/Diagnosis: Customers could experience elevated 5xx errors in europe-west2.
*   Impact Mitigation time: 07/19/22 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers could not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region or 45% for Multi Region to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max). Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can choose to route to europe-west1, no workarounds needed for europe-west2
*   Impact Mitigation time: 07/19/22 17:00 US/Pacific

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 experienced significantly increased failed requests.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/20/22 02:55 AM US/Pacific

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers experienced high latency for all API requests. Some queues were not being loaded and they have stopped executing tasks. Customers could also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Impact Mitigation time: 07/19/22 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH).
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Impact Mitigation time: 07/19/22 20:22 US/Pacific

**Vertex AI online prediction**

*   Impact/Diagnosis: End user could experience timeouts
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 15:11 US/Pacific

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a were inaccessible. Customers could also experience a 100% packet loss to europe-west2-a. Customers were unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 12:49 US/Pacific

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming experienced Listen and Write outage starting around 07:15 US/Pacific.
*   Impact Mitigation time: 07/19/22 09:55 US/Pacific

**Cloud Datastore:**

*   Impact/Diagnosis: Service was degraded. Customers may have experienced timeouts.
*   Impact Mitigation time: 07/19/22 19:30 US/Pacific

We will provide more information by Wednesday, 2022-07-20 15:30 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

20 Jul 2022 12:17 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is impacting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams are working on restoring their services. ETA to be determined.

We apologize to all who are affected by the disruption.

**Services Mitigated:** Google Cloud Storage, Google BigQuery, Google App Engine and Cloud Functions, Dataflow, Persistent Disk (PD), API Gateway, Cloud Spanner, Google Cloud Tasks, Google Compute Engine, Vertex AI online prediction, VPC (Traffic Virtnet), Cloud Filestore, Cloud Datastore, Looker, Cloud Composer, Cloud Data Fusion, Managed Service for Microsoft Active Directory, Cloud Dataproc, Bigtable, Datastream, GKE, Cloud Filestore,

**Services Mitigation in progress:** MemoryStore for Memcached, MemoryStore for Redis, Cloud SQL.

**Product Impact:**

**Cloud Filestore:**

*   Impact/Diagnosis: A small number of customers in europe-west2-a may be experiencing service unavailability. No new issues are expected for customers with working instances.
*   Workaround: Customers are advised to fail over to another region, if possible.
*   Impact Mitigation: 07/20/22 09:18 US/Pacific

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management API (create, update, delete, etc.), and cache flushes.
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.

**Cloud SQL:**

*   Impact/Diagnosis: 4 zonal instances are still impacted. Most non-HA instances backed by europe-west2-a are working again in europe-west2-a. And most HA instances in europe-west2 are operational again as of 19:00 US/Pacific.
*   Workaround: None at this time.

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a could be unavailable.
*   Workaround: Customers were encouraged to move their workloads to other regions if possible.
*   Impact Mitigation time: 07/20/22 07:57 US/Pacific

**Datastream:**

*   Impact/Diagnosis: Datastream Streams in that region might experience errors and lags which eventually could lead to lost position
*   Workaround: Customers were advised to run their Streams in another region.
*   Impact Mitigation time: 07/20/22 06:02 US/Pacific

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: Workloads using replicated databases with replicas in europe-west2, may be moved to the region close to the other replicas to reduce latency.
*   Impact Mitigation time: 07/20/22 04:51 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up could experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b, europe-west2-c or other regions.
*   Impact Mitigation time: 07/20/22 04:54 US/Pacific

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers were unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers could also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 22:15 US/Pacific

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 21:30 US/Pacific

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region could see elevated errors when attempting to read objects.
*   Impact Mitigation time: 07/19/22 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets were unavailable for the impacted locations.
*   Impact Mitigation time: 07/19/22 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers could experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Impact Mitigation time: 07/19/22 11:05 US/Pacific

**Dataflow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area might have been stuck. New Dataflow jobs could not start.
*   Impact Mitigation time: 07/19/22 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations experienced downgraded performance in europe-west2.
*   Impact Mitigation time: 07/19/22 22:30 US/Pacific

**Persistent Disk (PD):**

*   Impact/Diagnosis: Customers were unable to create any PD device in europe-west2. PD devices in europe-west2-a were unavailable
*   Workaround: Create PD devices in a different region/zone.
*   Impact Mitigation time: From 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c. From 07/20/22 04:51 US/Pacific all PDs should be healthy and back online.

**API Gateway:**

*   Impact/Diagnosis: Customers could experience elevated 5xx errors in europe-west2.
*   Impact Mitigation time: 07/19/22 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers could not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region or 45% for Multi Region to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max). Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can choose to route to europe-west1, no workarounds needed for europe-west2
*   Impact Mitigation time: 07/19/22 17:00 US/Pacific

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 experienced significantly increased failed requests.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/20/22 02:55 AM US/Pacific

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers experienced high latency for all API requests. Some queues were not being loaded and they have stopped executing tasks. Customers could also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Impact Mitigation time: 07/19/22 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH).
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Impact Mitigation time: 07/19/22 20:22 US/Pacific

**Vertex AI online prediction**

*   Impact/Diagnosis: End user could experience timeouts
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 15:11 US/Pacific

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a were inaccessible. Customers could also experience a 100% packet loss to europe-west2-a. Customers were unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 12:49 US/Pacific

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming experienced Listen and Write outage starting around 07:15 US/Pacific.
*   Impact Mitigation time: 07/19/22 09:55 US/Pacific

**Cloud Datastore:**

*   Impact/Diagnosis: Service was degraded. Customers may have experienced timeouts.
*   Impact Mitigation time: 07/19/22 19:30 US/Pacific

We will provide more information by Wednesday, 2022-07-20 13:45 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

20 Jul 2022 11:20 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is impacting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams are working on restoring their services. ETA to be determined.

We apologize to all who are affected by the disruption.

**Services Mitigated:** Google Cloud Storage, Google BigQuery, Google App Engine and Cloud Functions, Dataflow, Persistent Disk (PD), API Gateway, Cloud Spanner, Google Cloud Tasks, Google Compute Engine, Vertex AI online prediction, VPC (Traffic Virtnet), Cloud Filestore, Cloud Datastore, Looker, Cloud Composer, Cloud Data Fusion, Managed Service for Microsoft Active Directory, Cloud Dataproc, Bigtable, Datastream, GKE, Cloud Filestore,

**Services Mitigation in progress:** MemoryStore for Memcached, MemoryStore for Redis, Cloud SQL.

**Product Impact:**

**Cloud Filestore:**

*   Impact/Diagnosis: A small number of customers in europe-west2-a may be experiencing service unavailability. No new issues are expected for customers with working instances.
*   Workaround: Customers are advised to fail over to another region, if possible.
*   Impact Mitigation: 07/20/22 09:18 US/Pacific

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management API (create, update, delete, etc.), and cache flushes.
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.

**Cloud SQL:**

*   Impact/Diagnosis: 4 zonal instances are still impacted. Most non-HA instances backed by europe-west2-a are working again in europe-west2-a. And most HA instances in europe-west2 are operational again as of 19:00 US/Pacific.
*   Workaround: None at this time.

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a could be unavailable.
*   Workaround: Customers were encouraged to move their workloads to other regions if possible.
*   Impact Mitigation time: 07/20/22 07:57 US/Pacific

**Datastream:**

*   Impact/Diagnosis: Datastream Streams in that region might experience errors and lags which eventually could lead to lost position
*   Workaround: Customers were advised to run their Streams in another region.
*   Impact Mitigation time: 07/20/22 06:02 US/Pacific

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: Workloads using replicated databases with replicas in europe-west2, may be moved to the region close to the other replicas to reduce latency.
*   Impact Mitigation time: 07/20/22 04:51 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up could experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b, europe-west2-c or other regions.
*   Impact Mitigation time: 07/20/22 04:54 US/Pacific

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers were unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers could also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 22:15 US/Pacific

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 21:30 US/Pacific

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region could see elevated errors when attempting to read objects.
*   Impact Mitigation time: 07/19/22 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets were unavailable for the impacted locations.
*   Impact Mitigation time: 07/19/22 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers could experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Impact Mitigation time: 07/19/22 11:05 US/Pacific

**Dataflow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area might have been stuck. New Dataflow jobs could not start.
*   Impact Mitigation time: 07/19/22 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations experienced downgraded performance in europe-west2.
*   Impact Mitigation time: 07/19/22 22:30 US/Pacific

**Persistent Disk (PD):**

*   Impact/Diagnosis: Customers were unable to create any PD device in europe-west2. PD devices in europe-west2-a were unavailable
*   Workaround: Create PD devices in a different region/zone.
*   Impact Mitigation time: From 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c. From 07/20/22 04:51 US/Pacific all PDs should be healthy and back online.

**API Gateway:**

*   Impact/Diagnosis: Customers could experience elevated 5xx errors in europe-west2.
*   Impact Mitigation time: 07/19/22 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers could not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region or 45% for Multi Region to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max). Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can choose to route to europe-west1, no workarounds needed for europe-west2
*   Impact Mitigation time: 07/19/22 17:00 US/Pacific

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 experienced significantly increased failed requests.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/20/22 02:55 AM US/Pacific

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers experienced high latency for all API requests. Some queues were not being loaded and they have stopped executing tasks. Customers could also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Impact Mitigation time: 07/19/22 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH).
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Impact Mitigation time: 07/19/22 20:22 US/Pacific

**Vertex AI online prediction**

*   Impact/Diagnosis: End user could experience timeouts
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 15:11 US/Pacific

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a were inaccessible. Customers could also experience a 100% packet loss to europe-west2-a. Customers were unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 12:49 US/Pacific

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming experienced Listen and Write outage starting around 07:15 US/Pacific.
*   Impact Mitigation time: 07/19/22 09:55 US/Pacific

**Cloud Datastore:**

*   Impact/Diagnosis: Service was degraded. Customers may have experienced timeouts.
*   Impact Mitigation time: 07/19/22 19:30 US/Pacific

We will provide more information by Wednesday, 2022-07-20 12:15 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

20 Jul 2022 10:24 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is impacting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams are working on restoring their services. ETA to be determined.

We apologize to all who are affected by the disruption.

**Services Mitigated:** Google Cloud Storage, Google BigQuery, Google App Engine and Cloud Functions, Dataflow, Persistent Disk (PD), API Gateway, Cloud Spanner, Google Cloud Tasks, Google Compute Engine, Vertex AI online prediction, VPC (Traffic Virtnet), Cloud Filestore, Cloud Datastore, Looker, Cloud Composer, Cloud Data Fusion, Managed Service for Microsoft Active Directory, Cloud Dataproc, Bigtable, Datastream, GKE, Cloud Filestore,

**Services Mitigation in progress:** MemoryStore for Memcached, MemoryStore for Redis, Cloud SQL.

**Product Impact:**

**Cloud Filestore:**

*   Impact/Diagnosis: A small number of customers in europe-west2-a may be experiencing service unavailability. No new issues are expected for customers with working instances.
*   Workaround: Customers are advised to fail over to another region, if possible.
*   Impact Mitigation: 07/20/22 09:18 US/Pacific

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management API (create, update, delete, etc.), and cache flushes.
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.

**Cloud SQL:**

*   Impact/Diagnosis: 4 zonal instances are still impacted. Most non-HA instances backed by europe-west2-a are working again in europe-west2-a. And most HA instances in europe-west2 are operational again as of 19:00 US/Pacific.
*   Workaround: None at this time.

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a could be unavailable.
*   Workaround: Customers were encouraged to move their workloads to other regions if possible.
*   Impact Mitigation time: 07/20/22 07:57 US/Pacific

**Datastream:**

*   Impact/Diagnosis: Datastream Streams in that region might experience errors and lags which eventually could lead to lost position
*   Workaround: Customers were advised to run their Streams in another region.
*   Impact Mitigation time: 07/20/22 06:02 US/Pacific

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: Workloads using replicated databases with replicas in europe-west2, may be moved to the region close to the other replicas to reduce latency.
*   Impact Mitigation time: 07/20/22 04:51 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up could experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b, europe-west2-c or other regions.
*   Impact Mitigation time: 07/20/22 04:54 US/Pacific

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers were unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers could also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 22:15 US/Pacific

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 21:30 US/Pacific

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region could see elevated errors when attempting to read objects.
*   Impact Mitigation time: 07/19/22 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets were unavailable for the impacted locations.
*   Impact Mitigation time: 07/19/22 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers could experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Impact Mitigation time: 07/19/22 11:05 US/Pacific

**Dataflow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area might have been stuck. New Dataflow jobs could not start.
*   Impact Mitigation time: 07/19/22 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations experienced downgraded performance in europe-west2.
*   Impact Mitigation time: 07/19/22 22:30 US/Pacific

**Persistent Disk (PD):**

*   Impact/Diagnosis: Customers were unable to create any PD device in europe-west2. PD devices in europe-west2-a were unavailable
*   Workaround: Create PD devices in a different region/zone.
*   Impact Mitigation time: From 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c. From 07/20/22 04:51 US/Pacific all PDs should be healthy and back online.

**API Gateway:**

*   Impact/Diagnosis: Customers could experience elevated 5xx errors in europe-west2.
*   Impact Mitigation time: 07/19/22 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers could not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region or 45% for Multi Region to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max). Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can choose to route to europe-west1, no workarounds needed for europe-west2
*   Impact Mitigation time: 07/19/22 17:00 US/Pacific

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 experienced significantly increased failed requests.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/20/22 02:55 AM US/Pacific

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers experienced high latency for all API requests. Some queues were not being loaded and they have stopped executing tasks. Customers could also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Impact Mitigation time: 07/19/22 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH).
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Impact Mitigation time: 07/19/22 20:22 US/Pacific

**Vertex AI online prediction**

*   Impact/Diagnosis: End user could experience timeouts
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 15:11 US/Pacific

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a were inaccessible. Customers could also experience a 100% packet loss to europe-west2-a. Customers were unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 12:49 US/Pacific

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming experienced Listen and Write outage starting around 07:15 US/Pacific.
*   Impact Mitigation time: 07/19/22 09:55 US/Pacific

**Cloud Datastore:**

*   Impact/Diagnosis: Service was degraded. Customers may have experienced timeouts.
*   Impact Mitigation time: 07/19/22 19:30 US/Pacific

We will provide more information by Wednesday, 2022-07-20 11:30 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

20 Jul 2022 09:31 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is impacting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams are working on restoring their services. ETA to be determined.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Cloud Filestore:**

*   Impact/Diagnosis: A small number of customers in europe-west2-a may be experiencing service unavailability. No new issues are expected for customers with working instances.
*   Workaround: Customers are advised to fail over to another region, if possible.
*   Impact Mitigation: 07/20/22 09:18 US/Pacific

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management API (create, update, delete, etc.), and cache flushes.
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.

**Cloud SQL:**

*   Impact/Diagnosis: 4 zonal instances are still impacted. Most non-HA instances backed by europe-west2-a are working again in europe-west2-a. And most HA instances in europe-west2 are operational again as of 19:00 US/Pacific.
*   Workaround: None at this time.

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a could be unavailable.
*   Workaround: Customers were encouraged to move their workloads to other regions if possible.
*   Impact Mitigation time: 07/20/22 07:57 US/Pacific

**Datastream:**

*   Impact/Diagnosis: Datastream Streams in that region might experience errors and lags which eventually could lead to lost position
*   Workaround: Customers were advised to run their Streams in another region.
*   Impact Mitigation time: 07/20/22 06:02 US/Pacific

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: Workloads using replicated databases with replicas in europe-west2, may be moved to the region close to the other replicas to reduce latency.
*   Impact Mitigation time: 07/20/22 04:51 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up could experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b, europe-west2-c or other regions.
*   Impact Mitigation time: 07/20/22 04:54 US/Pacific

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers were unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers could also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 22:15 US/Pacific

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 21:30 US/Pacific

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region could see elevated errors when attempting to read objects.
*   Impact Mitigation time: 07/19/22 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets were unavailable for the impacted locations.
*   Impact Mitigation time: 07/19/22 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers could experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Impact Mitigation time: 07/19/22 11:05 US/Pacific

**Dataflow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area might have been stuck. New Dataflow jobs could not start.
*   Impact Mitigation time: 07/19/22 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations experienced downgraded performance in europe-west2.
*   Impact Mitigation time: 07/19/22 22:30 US/Pacific

**Persistent Disk (PD):**

*   Impact/Diagnosis: Customers were unable to create any PD device in europe-west2. PD devices in europe-west2-a were unavailable
*   Workaround: Create PD devices in a different region/zone.
*   Impact Mitigation time: From 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c. From 07/20/22 04:51 US/Pacific all PDs should be healthy and back online.

**API Gateway:**

*   Impact/Diagnosis: Customers could experience elevated 5xx errors in europe-west2.
*   Impact Mitigation time: 07/19/22 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers could not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region or 45% for Multi Region to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max). Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can choose to route to europe-west1, no workarounds needed for europe-west2
*   Impact Mitigation time: 07/19/22 17:00 US/Pacific

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 experienced significantly increased failed requests.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/20/22 02:55 AM US/Pacific

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers experienced high latency for all API requests. Some queues were not being loaded and they have stopped executing tasks. Customers could also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Impact Mitigation time: 07/19/22 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH).
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Impact Mitigation time: 07/19/22 20:22 US/Pacific

**Vertex AI online prediction**

*   Impact/Diagnosis: End user could experience timeouts
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 15:11 US/Pacific

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a were inaccessible. Customers could also experience a 100% packet loss to europe-west2-a. Customers were unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 12:49 US/Pacific

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming experienced Listen and Write outage starting around 07:15 US/Pacific.
*   Impact Mitigation time: 07/19/22 09:55 US/Pacific

**Cloud Datastore:**

*   Impact/Diagnosis: Service was degraded. Customers may have experienced timeouts.
*   Impact Mitigation time: 07/19/22 19:30 US/Pacific

We will provide more information by Wednesday, 2022-07-20 10:30 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

20 Jul 2022 08:10 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is impacting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams are working on restoring their services. ETA to be determined.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Cloud Filestore:**

*   Impact/Diagnosis: A small number of customers in europe-west2-a may be experiencing service unavailability. No new issues are expected for customers with working instances.
*   Workaround: Customers are advised to fail over to another region, if possible.

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management API (create, update, delete, etc.), and cache flushes.
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.

**Cloud SQL:**

*   Impact/Diagnosis: 4 zonal instances are still impacted. Most non-HA instances backed by europe-west2-a are working again in europe-west2-a. And most HA instances in europe-west2 are operational again as of 19:00 US/Pacific.
*   Workaround: None at this time.

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a could be unavailable.
*   Workaround: Customers were encouraged to move their workloads to other regions if possible.
*   Impact Mitigation time: 07/20/22 07:57 US/Pacific

**Datastream:**

*   Impact/Diagnosis: Datastream Streams in that region might experience errors and lags which eventually could lead to lost position
*   Workaround: Customers were advised to run their Streams in another region.
*   Impact Mitigation time: 07/20/22 06:02 US/Pacific

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: Workloads using replicated databases with replicas in europe-west2, may be moved to the region close to the other replicas to reduce latency.
*   Impact Mitigation time: 07/20/22 04:51 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up could experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b, europe-west2-c or other regions.
*   Impact Mitigation time: 07/20/22 04:54 US/Pacific

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers were unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers could also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 22:15 US/Pacific

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 21:30 US/Pacific

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region could see elevated errors when attempting to read objects.
*   Impact Mitigation time: 07/19/22 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets were unavailable for the impacted locations.
*   Impact Mitigation time: 07/19/22 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers could experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Impact Mitigation time: 07/19/22 11:05 US/Pacific

**Dataflow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area might have been stuck. New Dataflow jobs could not start.
*   Impact Mitigation time: 07/19/22 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations experienced downgraded performance in europe-west2.
*   Impact Mitigation time: 07/19/22 22:30 US/Pacific

**Persistent Disk (PD):**

*   Impact/Diagnosis: Customers were unable to create any PD device in europe-west2. PD devices in europe-west2-a were unavailable
*   Workaround: Create PD devices in a different region/zone.
*   Impact Mitigation time: From 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c. From 07/20/22 04:51 US/Pacific all PDs should be healthy and back online.

**API Gateway:**

*   Impact/Diagnosis: Customers could experience elevated 5xx errors in europe-west2.
*   Impact Mitigation time: 07/19/22 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers could not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region or 45% for Multi Region to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max). Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can choose to route to europe-west1, no workarounds needed for europe-west2
*   Impact Mitigation time: 07/19/22 17:00 US/Pacific

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 experienced significantly increased failed requests.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/20/22 02:55 AM US/Pacific

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers experienced high latency for all API requests. Some queues were not being loaded and they have stopped executing tasks. Customers could also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Impact Mitigation time: 07/19/22 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH).
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Impact Mitigation time: 07/19/22 20:22 US/Pacific

**Vertex AI online prediction**

*   Impact/Diagnosis: End user could experience timeouts
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 15:11 US/Pacific

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a were inaccessible. Customers could also experience a 100% packet loss to europe-west2-a. Customers were unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 12:49 US/Pacific

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming experienced Listen and Write outage starting around 07:15 US/Pacific.
*   Impact Mitigation time: 07/19/22 09:55 US/Pacific

**Cloud Datastore:**

*   Impact/Diagnosis: Service was degraded. Customers may have experienced timeouts.
*   Impact Mitigation time: 07/19/22 19:30 US/Pacific

We will provide more information by Wednesday, 2022-07-20 09:30 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

20 Jul 2022 07:11 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is impacting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams are working on restoring their services. ETA to be determined.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Cloud Filestore:**

*   Impact/Diagnosis: A small number of customers in europe-west2-a may be experiencing service unavailability. No new issues are expected for customers with working instances.
*   Workaround: Customers are advised to fail over to another region, if possible.

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management API (create, update, delete, etc.), and cache flushes.
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable.
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Cloud SQL:**

*   Impact/Diagnosis: 4 zonal instances are still impacted. Most non-HA instances backed by europe-west2-a are working again in europe-west2-a. And most HA instances in europe-west2 are operational again as of 19:00 US/Pacific.
*   Workaround: None at this time.

**Datastream:**

*   Impact/Diagnosis: Datastream Streams in that region might experience errors and lags which eventually could lead to lost position
*   Workaround: Customers are advised to run their Streams in another region.
*   Impact Mitigation time: 07/20/22 06:02 US/Pacific

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: Workloads using replicated databases with replicas in europe-west2, may be moved to the region close to the other replicas to reduce latency.
*   Impact Mitigation time: 07/20/22 04:51 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up could experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b, europe-west2-c or other regions.
*   Impact Mitigation time: 07/20/22 04:54 US/Pacific

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers were unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers could also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 22:15 US/Pacific

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 21:30 US/Pacific

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region could see elevated errors when attempting to read objects.
*   Impact Mitigation time: 07/19/22 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets were unavailable for the impacted locations.
*   Impact Mitigation time: 07/19/22 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers could experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Impact Mitigation time: 07/19/22 11:05 US/Pacific

**Dataflow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area might have been stuck. New Dataflow jobs could not start.
*   Impact Mitigation time: 07/19/22 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations experienced downgraded performance in europe-west2.
*   Impact Mitigation time: 07/19/22 22:30 US/Pacific

**Persistent Disk (PD):**

*   Impact/Diagnosis: Customers were unable to create any PD device in europe-west2. PD devices in europe-west2-a were unavailable
*   Workaround: Create PD devices in a different region/zone.
*   Impact Mitigation time: From 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c. From 07/20/22 04:51 US/Pacific all PDs should be healthy and back online.

**API Gateway:**

*   Impact/Diagnosis: Customers could experience elevated 5xx errors in europe-west2.
*   Impact Mitigation time: 07/19/22 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers could not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region or 45% for Multi Region to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max). Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can choose to route to europe-west1, no workarounds needed for europe-west2
*   Impact Mitigation time: 07/19/22 17:00 US/Pacific

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 experienced significantly increased failed requests.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/20/22 02:55 AM US/Pacific

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers experienced high latency for all API requests. Some queues were not being loaded and they have stopped executing tasks. Customers could also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Impact Mitigation time: 07/19/22 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH).
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Impact Mitigation time: 07/19/22 20:22 US/Pacific

**Vertex AI online prediction**

*   Impact/Diagnosis: End user could experience timeouts
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 15:11 US/Pacific

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a were inaccessible. Customers could also experience a 100% packet loss to europe-west2-a. Customers were unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 12:49 US/Pacific

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming experienced Listen and Write outage starting around 07:15 US/Pacific.
*   Impact Mitigation time: 07/19/22 09:55 US/Pacific

**Cloud Datastore:**

*   Impact/Diagnosis: Service was degraded. Customers may have experienced timeouts.
*   Impact Mitigation time: 07/19/22 19:30 US/Pacific

We will provide more information by Wednesday, 2022-07-20 08:15 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

20 Jul 2022 06:12 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is impacting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams are working on restoring their services. ETA to be determined.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Cloud Filestore:**

*   Impact/Diagnosis: A small number of customers in europe-west2-a may be experiencing service unavailability. No new issues are expected for customers with working instances.
*   Workaround: Customers are advised to fail over to another region, if possible.

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management API (create, update, delete, etc.), and cache flushes.
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable.
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Cloud SQL:**

*   Impact/Diagnosis: 4 zonal instances are still impacted. Most non-HA instances backed by europe-west2-a are working again in europe-west2-a. And most HA instances in europe-west2 are operational again as of 19:00 US/Pacific.
*   Workaround: None at this time.

**Datastream:**

*   Impact/Diagnosis: Datastream Streams in that region might experience errors and lags which eventually could lead to lost position
*   Workaround: Customers are advised to run their Streams in another region.
*   Impact Mitigation time: 07/20/22 06:02 US/Pacific

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: Workloads using replicated databases with replicas in europe-west2, may be moved to the region close to the other replicas to reduce latency.
*   Impact Mitigation time: 07/20/22 04:51 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up could experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b, europe-west2-c or other regions.
*   Impact Mitigation time: 07/20/22 04:54 US/Pacific

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers were unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers could also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 22:15 US/Pacific

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 21:30 US/Pacific

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region could see elevated errors when attempting to read objects.
*   Impact Mitigation time: 07/19/22 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets were unavailable for the impacted locations.
*   Impact Mitigation time: 07/19/22 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers could experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Impact Mitigation time: 07/19/22 11:05 US/Pacific

**Dataflow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area might have been stuck. New Dataflow jobs could not start.
*   Impact Mitigation time: 07/19/22 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations experienced downgraded performance in europe-west2.
*   Impact Mitigation time: 07/19/22 22:30 US/Pacific

**Persistent Disk (PD):**

*   Impact/Diagnosis: Customers were unable to create any PD device in europe-west2. PD devices in europe-west2-a were unavailable
*   Workaround: Create PD devices in a different region/zone.
*   Impact Mitigation time: From 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c. From 07/20/22 04:51 US/Pacific all PDs should be healthy and back online.

**API Gateway:**

*   Impact/Diagnosis: Customers could experience elevated 5xx errors in europe-west2.
*   Impact Mitigation time: 07/19/22 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers could not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region or 45% for Multi Region to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max). Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can choose to route to europe-west1, no workarounds needed for europe-west2
*   Impact Mitigation time: 07/19/22 17:00 US/Pacific

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 experienced significantly increased failed requests.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/20/22 02:55 AM US/Pacific

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers experienced high latency for all API requests. Some queues were not being loaded and they have stopped executing tasks. Customers could also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Impact Mitigation time: 07/19/22 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH).
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Impact Mitigation time: 07/19/22 20:22 US/Pacific

**Vertex AI online prediction**

*   Impact/Diagnosis: End user could experience timeouts
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 15:11 US/Pacific

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a were inaccessible. Customers could also experience a 100% packet loss to europe-west2-a. Customers were unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 12:49 US/Pacific

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming experienced Listen and Write outage starting around 07:15 US/Pacific.
*   Impact Mitigation time: 07/19/22 09:55 US/Pacific

**Cloud Datastore:**

*   Impact/Diagnosis: Service was degraded. Customers may have experienced timeouts.
*   Impact Mitigation time: 07/19/22 19:30 US/Pacific

We will provide more information by Wednesday, 2022-07-20 07:15 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

20 Jul 2022 05:16 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is impacting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams are working on restoring their services. ETA to be determined.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Cloud Filestore:**

*   Impact/Diagnosis: A small number of customers in europe-west2-a may be experiencing service unavailability. No new issues are expected for customers with working instances.
*   Workaround: Customers are advised to fail over to another region, if possible.

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management API (create, update, delete, etc.), and cache flushes.
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable.
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: Workloads using replicated databases with replicas in europe-west2, may be moved to the region close to the other replicas to reduce latency.

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up may experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b, europe-west2-c or other regions.

**Datastream:**

*   Impact/Diagnosis: Datastream Streams in that region might experience errors and lags which eventually could lead to lost position
*   Workaround: Customers are advised to run their Streams in another region.

**Cloud SQL:**

*   Impact/Diagnosis: 4 zonal instances are still impacted. Most non-HA instances backed by europe-west2-a are working again in europe-west2-a. And most HA instances in europe-west2 are operational again as of 19:00 US/Pacific.
*   Workaround: None at this time.

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers will be unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers will also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 22:15 US/Pacific

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 21:30 US/Pacific

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region may have seen elevated errors when attempting to read objects.
*   Impact Mitigation time: 07/19/22 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets were unavailable for the impacted locations.
*   Impact Mitigation time: 07/19/22 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers may have experienced high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Impact Mitigation time: 07/19/22 11:05 US/Pacific

**Dataflow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may have been stuck. New Dataflow jobs may not start.
*   Impact Mitigation time: 07/19/22 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations experienced downgraded performance in europe-west2.
*   Impact Mitigation time: 07/19/22 22:30 US/Pacific

**Persistent Disk (PD):**

*   Impact/Diagnosis: All PDs should be healthy and back online
*   Workaround: Create PD devices in a different region/zone.
*   Impact Mitigation time: From 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c. From 07/20/22 04:18 US/Pacific all PDs should be healthy and back online.

**API Gateway:**

*   Impact/Diagnosis: Customers may have experienced elevated 5xx errors in europe-west2.
*   Impact Mitigation time: 07/19/22 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers will not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region or 45% for Multi Region to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max). Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can choose to route to europe-west1, no workarounds needed for europe-west2
*   Impact Mitigation time: 07/19/22 17:00 US/Pacific

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 will experience significantly increased failed requests.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/20/22 02:55 AM US/Pacific

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers will experience high latency for all API requests. Some queues are not being loaded and they have stopped executing tasks. Customers may also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Impact Mitigation time: 07/19/22 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH).
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Impact Mitigation time: 07/19/22 20:22 US/Pacific

**Vertex AI online prediction**

*   Impact/Diagnosis: End user will experience timeouts
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 15:11 US/Pacific

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a currently inaccessible. Customers will also experience 100% packet loss to europe-west2-a. Customers are unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 12:49 US/Pacific

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming experienced Listen and Write outage starting around 07:15 US/Pacific.
*   Impact Mitigation time: 07/19/22 09:55 US/Pacific

**Cloud Datastore:**

*   Impact/Diagnosis: Service was degraded. Customers may have experienced timeouts.
*   Impact Mitigation time: 07/19/22 19:30 US/Pacific

We will provide more information by Wednesday, 2022-07-20 06:15 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

20 Jul 2022 04:13 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is impacting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams are working on restoring their services. ETA to be determined.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Cloud Filestore:**

*   Impact/Diagnosis: A small number of customers in europe-west2-a may be experiencing service unavailability. No new issues are expected for customers with working instances.
*   Workaround: Customers are advised to fail over to another region, if possible.

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management API (create, update, delete, etc.), and cache flushes.
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable.
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: Workloads using replicated databases with replicas in europe-west2, may be moved to the region close to the other replicas to reduce latency.

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up may experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b, europe-west2-c or other regions.

**Datastream:**

*   Impact/Diagnosis: Datastream Streams in that region might experience errors and lags which eventually could lead to lost position
*   Workaround: Customers are advised to run their Streams in another region.

**Cloud SQL:**

*   Impact/Diagnosis: 4 zonal instances are still impacted. Most non-HA instances backed by europe-west2-a are working again in europe-west2-a. And most HA instances in europe-west2 are operational again as of 19:00 US/Pacific.
*   Workaround: None at this time.

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers will be unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers will also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 22:15 US/Pacific

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.
*   Impact Mitigation time: 07/19/22 21:30 US/Pacific

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region may have seen elevated errors when attempting to read objects.
*   Impact Mitigation time: 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets were unavailable for the impacted locations.
*   Impact Mitigation time: 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers may have experienced high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Impact Mitigation time: 11:05 US/Pacific

**Dataflow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may have been stuck. New Dataflow jobs may not start.
*   Impact Mitigation time: 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations experienced downgraded performance in europe-west2.
*   Impact Mitigation time: 22:30 US/Pacific

**Persistent Disk (PD):**

*   Impact/Diagnosis: Existing PD devices in europe-west2-a may be unavailable. Replicated PD devices in europe-west2 may see some replica unavailability. Most volumes in europe-west2-a are now available. There are a small number of HDD backed PD volumes that continue to experience impact that will result in IO errors.
*   Workaround: Create PD devices in a different region/zone.
*   Impact Mitigation time: From 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c.

**API Gateway:**

*   Impact/Diagnosis: Customers may have experienced elevated 5xx errors in europe-west2.
*   Impact Mitigation time: 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers will not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region or 45% for Multi Region to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max). Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can choose to route to europe-west1, no workarounds needed for europe-west2
*   Impact Mitigation time: 17:00 US/Pacific

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 will experience significantly increased failed requests.
*   Workaround: None at this time.
*   Impact Mitigation time: 02:55 AM US/Pacific

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers will experience high latency for all API requests. Some queues are not being loaded and they have stopped executing tasks. Customers may also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Impact Mitigation time: 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH).
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Impact Mitigation time: 20:22 US/Pacific

**Vertex AI online prediction**

*   Impact/Diagnosis: End user will experience timeouts
*   Workaround: None at this time.
*   Impact Mitigation time: 15:11 US/Pacific

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a currently inaccessible. Customers will also experience 100% packet loss to europe-west2-a. Customers are unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 12:49 US/Pacific

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming experienced Listen and Write outage starting around 07:15 US/Pacific.
*   Impact Mitigation time: 09:55 US/Pacific

**Cloud Datastore:**

*   Impact/Diagnosis: Service was degraded. Customers may have experienced timeouts.
*   Impact Mitigation time: 19:30 US/Pacific

We will provide more information by Wednesday, 2022-07-20 05:15 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

20 Jul 2022 03:00 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is impacting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams are working on restoring their services. ETA to be determined.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region may have seen elevated errors when attempting to read objects.
*   Impact Mitigation time: 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets were unavailable for the impacted locations.
*   Impact Mitigation time: 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers may have experienced high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Impact Mitigation time: 11:05 US/Pacific

**Dataflow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may have been stuck. New Dataflow jobs may not start.
*   Impact Mitigation time: 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations experienced downgraded performance in europe-west2.
*   Impact Mitigation time: 22:30 US/Pacific

**Cloud Filestore:**

*   Impact/Diagnosis: A small number of customers in europe-west2-a may be experiencing service unavailability. No new issues are expected for customers with working instances.
*   Workaround: Customers are advised to fail over to another region, if possible.

**Cloud SQL:**

*   Impact/Diagnosis: Most non-HA instances backed by europe-west2-a are working again in europe-west2-a. And most HA instances in europe-west2 are operational again as of 19:00 US/Pacific.
*   Impact Mitigation time: 22:30 US/Pacific

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management API (create, update, delete, etc.), and cache flushes.
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.

**Persistent Disk (PD):**

*   Impact/Diagnosis: Existing PD devices in europe-west2-a may be unavailable. Replicated PD devices in europe-west2 may see some replica unavailability. Most volumes in europe-west2-a are now available. There are a small number of HDD backed PD volumes that continue to experience impact that will result in IO errors.
*   Workaround: Create PD devices in a different region/zone.
*   Impact mitigation time: From 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c.

**API Gateway:**

*   Impact/Diagnosis: Customers may have experienced elevated 5xx errors in europe-west2.
*   Impact Mitigation time: 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers will not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region or 45% for Multi Region to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max). Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can choose to route to europe-west1, no workarounds needed for europe-west2
*   Impact Mitigation time: 17:00 US/Pacific

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable.
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 will experience significantly increased failed requests.
*   Workaround: None at this time.

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers will experience high latency for all API requests. Some queues are not being loaded and they have stopped executing tasks. Customers may also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Impact Mitigation time: 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH).
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Impact Mitigation time: 20:22 US/Pacific

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers will be unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers will also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.

**Vertex AI online prediction**

*   Impact/Diagnosis: End user will experience timeouts
*   Workaround: None at this time.
*   Impact Mitigation time: 15:11 US/Pacific

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a currently inaccessible. Customers will also experience 100% packet loss to europe-west2-a. Customers are unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 12:49 US/Pacific

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: Workloads using replicated databases with replicas in europe-west2, may be moved to the region close to the other replicas to reduce latency..

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming experienced Listen and Write outage starting around 07:15 US/Pacific.
*   Impact Mitigation time: 09:55 US/Pacific

**Cloud Datastore:**

*   Impact/Diagnosis: Service was degraded. Customers may have experienced timeouts.
*   Impact Mitigation time: 19:30 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up may experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b, europe-west2-c or other regions.

**Datastream:**

*   Impact/Diagnosis: Datastream Streams in that region might experience errors and lags which eventually could lead to lost position
*   Workaround: Customers are advised to run their Streams in another region.

We will provide more information by Wednesday, 2022-07-20 04:15 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

20 Jul 2022 01:24 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is impacting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams are working on restoring their services. ETA to be determined.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region may have seen elevated errors when attempting to read objects.
*   Impact Mitigation time: 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets were unavailable for the impacted locations.
*   Impact Mitigation time: 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers may have experienced high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Impact Mitigation time: 11:05 US/Pacific

**Dataflow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may have been stuck. New Dataflow jobs may not start.
*   Impact Mitigation time: 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations experienced downgraded performance in europe-west2.
*   Impact Mitigation time: 22:30 US/Pacific

**Cloud Filestore:**

*   Impact/Diagnosis: A small number of customers in europe-west2-a may be experiencing service unavailability. No new issues are expected for customers with working instances.
*   Workaround: Customers are advised to fail over to another region, if possible.

**Cloud SQL:**

*   Impact/Diagnosis: Most non-HA instances backed by europe-west2-a are working again in europe-west2-a. And most HA instances in europe-west2 are operational again as of 19:00 US/Pacific.
*   Impact Mitigation time: 22:30 US/Pacific

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management API (create, update, delete, etc.), and cache flushes.
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.

**Persistent Disk (PD):**

*   Impact/Diagnosis: Existing PD devices in europe-west2-a may be unavailable. Replicated PD devices in europe-west2 may see some replica unavailability. Most volumes in europe-west2-a are now available. There are a small number of HDD backed PD volumes that continue to experience impact that will result in IO errors.
*   Workaround: Create PD devices in a different region/zone.
*   Impact mitigation time: From 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c.

**API Gateway:**

*   Impact/Diagnosis: Customers may have experienced elevated 5xx errors in europe-west2.
*   Impact Mitigation time: 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers will not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region or 45% for Multi Region to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max). Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can choose to route to europe-west1, no workarounds needed for europe-west2
*   Impact Mitigation time: 17:00 US/Pacific

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable.
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 will experience significantly increased failed requests.
*   Workaround: None at this time.

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers will experience high latency for all API requests. Some queues are not being loaded and they have stopped executing tasks. Customers may also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Impact Mitigation time: 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH).
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Impact Mitigation time: 20:22 US/Pacific

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers will be unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers will also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.

**Vertex AI online prediction**

*   Impact/Diagnosis: End user will experience timeouts
*   Workaround: None at this time.
*   Impact Mitigation time: 15:11 US/Pacific

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a currently inaccessible. Customers will also experience 100% packet loss to europe-west2-a. Customers are unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 12:49 US/Pacific

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: Workloads using replicated databases with replicas in europe-west2, may be moved to the region close to the other replicas to reduce latency..

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming experienced Listen and Write outage starting around 07:15 US/Pacific.
*   Impact Mitigation time: 09:55 US/Pacific

**Cloud Datastore:**

*   Impact/Diagnosis: Service was degraded. Customers may have experienced timeouts.
*   Impact Mitigation time: 19:30 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up may experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b, europe-west2-c or other regions.

**Datastream:**

*   Impact/Diagnosis: Datastream Streams in that region might experience errors and lags which eventually could lead to lost position
*   Workaround: Customers are advised to run their Streams in another region.

We will provide more information by Wednesday, 2022-07-20 03:00 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

20 Jul 2022 00:55 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is impacting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams are working on restoring their services. ETA to be determined.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region may have seen elevated errors when attempting to read objects.
*   Impact Mitigation time: 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets were unavailable for the impacted locations.
*   Impact Mitigation time: 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers may have experienced high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Impact Mitigation time: 11:05 US/Pacific

**Dataflow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may have been stuck. New Dataflow jobs may not start.
*   Impact Mitigation time: 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations will experience downgraded performance in europe-west2.
*   Workaround: None at this time.

**Cloud Filestore:**

*   Impact/Diagnosis: europe-west2-a is unavailable for all customers.
*   Workaround: Customers are advised to fail over to another region, if possible.

**Cloud SQL:**

*   Impact/Diagnosis: Most non-HA instances backed by europe-west2-a are working again in europe-west2-a. And most HA instances in europe-west2 are operational again as of 19:00 US/Pacific.
*   Impact Mitigation time: 22:30 US/Pacific

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management API (create, update, delete, etc.), and cache flushes.
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.

**Persistent Disk (PD):**

*   Impact/Diagnosis: Existing PD devices in europe-west2-a may be unavailable. Replicated PD devices in europe-west2 may see some replica unavailability. Most volumes in europe-west2-a are now available. There are a small number of HDD backed PD volumes that continue to experience impact that will result in IO errors.
*   Workaround: Create PD devices in a different region/zone.
*   Impact mitigation time: From 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c.

**API Gateway:**

*   Impact/Diagnosis: Customers may have experienced elevated 5xx errors in europe-west2.
*   Impact Mitigation time: 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers will not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region or 45% for Multi Region to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max). Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can choose to route to europe-west1, no workarounds needed for europe-west2
*   Impact Mitigation time: 17:00 US/Pacific

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable.
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 will experience significantly increased failed requests.
*   Workaround: None at this time.

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers will experience high latency for all API requests. Some queues are not being loaded and they have stopped executing tasks. Customers may also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Impact Mitigation time: 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH).
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Impact Mitigation time: 20:22 US/Pacific

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers will be unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers will also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.

**Vertex AI online prediction**

*   Impact/Diagnosis: End user will experience timeouts
*   Workaround: None at this time.
*   Impact Mitigation time: 15:11 US/Pacific

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a currently inaccessible. Customers will also experience 100% packet loss to europe-west2-a. Customers are unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 12:49 US/Pacific

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: Workloads using replicated databases with replicas in europe-west2, may be moved to the region close to the other replicas to reduce latency..

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming experienced Listen and Write outage starting around 07:15 US/Pacific.
*   Impact Mitigation time: 09:55 US/Pacific

**Cloud Datastore:**

*   Impact/Diagnosis: Service was degraded. Customers may have experienced timeouts.
*   Impact Mitigation time: 19:30 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up may experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b, europe-west2-c or other regions.

**Datastream:**

*   Impact/Diagnosis: Datastream Streams in that region might experience errors and lags which eventually could lead to lost position
*   Workaround: Customers are advised to run their Streams in another region.

We will provide more information by Wednesday, 2022-07-20 01:00 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

19 Jul 2022 23:28 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is impacting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams are working on restoring their services. ETA to be determined.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region will see elevated errors when attempting to read objects.
*   Workaround: None at this time.
*   Impact Mitigation time: 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets will be unavailable for the impacted locations.
*   Workaround: None at this time.
*   Impact Mitigation time: 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers may experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Workaround: Customers are advised to fail over to another region, if possible.
*   Impact Mitigation time: 11:05 US/Pacific

**Dataflow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may be stuck. New Dataflow jobs may not start.
*   Workaround: Customers are advised to run their jobs in another region.
*   Impact Mitigation time: 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations will experience downgraded performance in europe-west2.
*   Workaround: None at this time.

**Cloud Filestore:**

*   Impact/Diagnosis: europe-west2-a is unavailable for all customers.
*   Workaround: Customers are advised to fail over to another region, if possible.

**Cloud SQL:**

*   Impact/Diagnosis: Most non-HA instances backed by europe-west2-a are working again in europe-west2-a. And most HA instances in europe-west2 are operational again as of 19:00 US/Pacific.
*   Workaround: None at this time.

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management API (create, update, delete, etc.), and cache flushes.
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.

**Persistent Disk (PD):**

*   Impact/Diagnosis: Existing PD devices in europe-west2-a may be unavailable. Replicated PD devices in europe-west2 may see some replica unavailability. Most volumes in europe-west2-a are now available. There are a small number of HDD backed PD volumes that continue to experience impact that will result in IO errors.
*   Workaround: Create PD devices in a different region/zone.
*   Impact mitigation time: From 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c.

**API Gateway:**

*   Impact/Diagnosis: Customers may see elevated 5xx errors in europe-west2.
*   Workaround: None at this time.
*   Impact Mitigation time: 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers will not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region or 45% for Multi Region to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max). Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can choose to route to europe-west1, no workarounds needed for europe-west2
*   Impact Mitigation time: 17:00 US/Pacific

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable.
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 will experience significantly increased failed requests.
*   Workaround: None at this time.

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers will experience high latency for all API requests. Some queues are not being loaded and they have stopped executing tasks. Customers may also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Workaround: None at this time.
*   Impact Mitigation time: 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH).
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Impact Mitigation time: 20:22 US/Pacific

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers will be unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers will also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.

**Vertex AI online prediction**

*   Impact/Diagnosis: End user will experience timeouts
*   Workaround: None at this time.
*   Impact Mitigation time: 15:11 US/Pacific

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a currently inaccessible. Customers will also experience 100% packet loss to europe-west2-a. Customers are unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 12:49 US/Pacific

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: None at this time.

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming Listen and Write outage starting around 07:15 US/Pacific.
*   Workaround: None at this time.
*   Impact Mitigation time: 09:55 US/Pacific

**Cloud Datastore:**

*   Impact/Diagnosis: Service is degraded. Customers may experience timeouts.
*   Workaround: None at this time.
*   Impact Mitigation time: 19:30 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up may experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b, europe-west2-c or other regions.

We will provide more information by Wednesday, 2022-07-20 01:00 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

19 Jul 2022 22:16 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is impacting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams are working on restoring their services. ETA to be determined.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region will see elevated errors when attempting to read objects.
*   Workaround: None at this time.
*   Impact Mitigation time: 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets will be unavailable for the impacted locations.
*   Workaround: None at this time.
*   Impact Mitigation time: 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers may experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Workaround: Customers are advised to fail over to another region, if possible.
*   Impact Mitigation time: 11:05 US/Pacific

**Dataflow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may be stuck. New Dataflow jobs may not start.
*   Workaround: Customers are advised to run their jobs in another region.
*   Impact Mitigation time: 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations will experience downgraded performance in europe-west2.
*   Workaround: None at this time.

**Cloud Filestore:**

*   Impact/Diagnosis: europe-west2-a is unavailable for all customers.
*   Workaround: Customers are advised to fail over to another region, if possible.

**Cloud SQL:**

*   Impact/Diagnosis: Most non-HA instances backed by europe-west2-a are working again in europe-west2-a. And most HA instances in europe-west2 are operational again as of 19:00 US/Pacific.
*   Workaround: None at this time.

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management API (create, update, delete, etc.), and cache flushes.
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.

**Persistent Disk (PD):**

*   Impact/Diagnosis: Existing PD devices in europe-west2-a may be unavailable. Replicated PD devices in europe-west2 may see some replica unavailability. Most volumes in europe-west2-a are now available. There are a small number of HDD backed PD volumes that continue to experience impact that will result in IO errors.
*   Workaround: Create PD devices in a different region/zone.
*   Impact mitigation time: From 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c.

**API Gateway:**

*   Impact/Diagnosis: Customers may see elevated 5xx errors in europe-west2.
*   Workaround: None at this time.
*   Impact Mitigation time: 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers will not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region or 45% for Multi Region to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max). Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can choose to route to europe-west1, no workarounds needed for europe-west2
*   Impact Mitigation time: 17:00 US/Pacific

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable.
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 will experience significantly increased failed requests.
*   Workaround: None at this time.

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers will experience high latency for all API requests. Some queues are not being loaded and they have stopped executing tasks. Customers may also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Workaround: None at this time.
*   Impact Mitigation time: 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH).
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Impact Mitigation time: 20:22 US/Pacific

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers will be unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers will also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.

**Vertex AI online prediction**

*   Impact/Diagnosis: End user will experience timeouts
*   Workaround: None at this time.
*   Impact Mitigation time: 15:11 US/Pacific

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a currently inaccessible. Customers will also experience 100% packet loss to europe-west2-a. Customers are unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 12:49 US/Pacific

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: None at this time.

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming Listen and Write outage starting around 07:15 US/Pacific.
*   Workaround: None at this time.
*   Impact Mitigation time: 09:55 US/Pacific

**Cloud Datastore:**

*   Impact/Diagnosis: Service is degraded. Customers may experience timeouts.
*   Workaround: None at this time.
*   Impact Mitigation time: 19:30 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up may experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b, europe-west2-c or other regions.

We will provide more information by Tuesday, 2022-07-19 23:30 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

19 Jul 2022 21:11 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is impacting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams are working on restoring their services. ETA to be determined.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region will see elevated errors when attempting to read objects.
*   Workaround: None at this time.
*   Impact Mitigation time: 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets will be unavailable for the impacted locations.
*   Workaround: None at this time.
*   Impact Mitigation time: 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers may experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Workaround: Customers are advised to fail over to another region, if possible.
*   Impact Mitigation time: 11:05 US/Pacific

**Dataflow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may be stuck. New Dataflow jobs may not start.
*   Workaround: Customers are advised to run their jobs in another region.
*   Impact Mitigation time: 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations will experience downgraded performance in europe-west2.
*   Workaround: None at this time.

**Cloud Filestore:**

*   Impact/Diagnosis: europe-west2-a is unavailable for all customers.
*   Workaround: Customers are advised to fail over to another region, if possible.

**Cloud SQL:**

*   Impact/Diagnosis: 50% of non-HA instances backed by europe-west2-a are hard-down in europe-west2-a. Most HA instances in europe-west2 are operational again as of 19:00 US/Pacific.
*   Workaround: None at this time.

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management API (create, update, delete, etc.), and cache flushes.
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.

**Persistent Disk (PD):**

*   Impact/Diagnosis: Existing PD devices in europe-west2-a may be unavailable. Replicated PD devices in europe-west2 may see some replica unavailability. Most volumes in europe-west2-a are now available. There are a small number of HDD backed PD volumes that continue to experience impact that will result in IO errors.
*   Workaround: Create PD devices in a different region/zone.
*   Impact mitigated: Starting at 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c.

**API Gateway:**

*   Impact/Diagnosis: Customers may see elevated 5xx errors in europe-west2.
*   Workaround: None at this time.
*   Impact Mitigation time: 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers will not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region or 45% for Multi Region to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max). Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can choose to route to europe-west1, no workarounds needed for europe-west2
*   Impact Mitigation time: 17:00 US/Pacific

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable.
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 will experience significantly increased failed requests.
*   Workaround: None at this time.

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers will experience high latency for all API requests. Some queues are not being loaded and they have stopped executing tasks. Customers may also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Workaround: None at this time.
*   Impact Mitigation time: 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH).
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Impact Mitigation time: 20:22 US/Pacific

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers will be unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers will also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.

**Vertex AI online prediction**

*   Impact/Diagnosis: End user will experience timeouts
*   Workaround: None at this time.
*   Impact Mitigation time: 15:11 US/Pacific

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a currently inaccessible. Customers will also experience 100% packet loss to europe-west2-a. Customers are unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 12:49 US/Pacific

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: None at this time.

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming Listen and Write outage starting around 07:15 US/Pacific.
*   Workaround: None at this time.
*   Impact Mitigation time: 09:55 US/Pacific

**Cloud Datastore:**

*   Impact/Diagnosis: Service is degraded. Customers may experience timeouts.
*   Workaround: None at this time.
*   Impact Mitigation time: 19:30 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up may experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b, europe-west2-c or other regions.

We will provide more information by Tuesday, 2022-07-19 22:30 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

19 Jul 2022 19:45 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is affecting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams are working on restoring their services. ETA to be determined.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region will see elevated errors when attempting to read objects
*   Workaround: None at this time.
*   Impact Mitigation time: 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets will be unavailable for the impacted locations.
*   Workaround: None at this time.
*   Impact Mitigation time: 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers may experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Workaround: Customers are advised to fail over to another region if possible.
*   Impact Mitigation time: 11:05 US/Pacific

**DataFlow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may be stuck. New Dataflow jobs may not start.
*   Workaround: Customers are advised to run their jobs in another region.
*   Impact Mitigation time: 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations will experience downgraded performance in europe-west2.
*   Workaround: None at this time.

**Cloud Filestore:**

*   Impact/Diagnosis: europe-west2-a is unavailable for all customers
*   Workaround: Customers are advised to fail over to another region if possible.

**Cloud SQL:**

*   Impact/Diagnosis: 50% of non-HA instances backed by europe-west2-a are hard-down in europe-west2-a. All but 13 HA instances in europe-west2 are operational again as of 19:00 PT.
*   Workaround: None at this time.

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management api (create,update,delete,etc), and cache flushes
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.

**Persistent Disk:**

*   Impact/Diagnosis: Existing PD devices in europe-west2-a may be unavailable. Replicated PD devices in europe-west2 may see some replica unavailability.
*   Workaround: Create PD devices in a different region/zone.
*   Impact mitigated: Starting at 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c.

**API Gateway:**

*   Impact/Diagnosis: Customers may see elevated 5xx errors in europe-west2
*   Workaround: None at this time.
*   Impact Mitigation time: 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers will not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region, 45% for Multi Region . to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max) . Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can chose to route to europe-west1, no workarounds needed for europe-west2
*   Impact Mitigation time: 17:00 US/Pacific

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 will experience significantly increased failed requests.
*   Workaround: None at this time.

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers will experience high latency for all API requests. Some queues are not being loaded and they have stopped executing tasks. Customers may also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Workaround: None at this time.
*   Impact Mitigation time: 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers will be unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers will also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.

**Vertex AI online prediction**

*   Impact/Diagnosis: End user will experience timeouts
*   Workaround: None at this time.

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a currently inaccessible. Customers will also experience 100% packet loss to europe-west2-a. Customers are unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 12:49 US/Pacific

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: None at this time.

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming Listen and Write outage starting around 07:15 US/Pacific
*   Workaround: none
*   Impact Mitigation time: 09:55 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up may experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b , europe-west2-c or other regions.

We will provide more information by Tuesday, 2022-07-19 21:30 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

19 Jul 2022 19:06 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is affecting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams are working on restoring their services. ETA to be determined.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region will see elevated errors when attempting to read objects
*   Workaround: None at this time.
*   Impact Mitigation time: 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets will be unavailable for the impacted locations.
*   Workaround: None at this time.
*   Impact Mitigation time: 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers may experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Workaround: Customers are advised to fail over to another region if possible.
*   Impact Mitigation time: 11:05 US/Pacific

**DataFlow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may be stuck. New Dataflow jobs may not start.
*   Workaround: Customers are advised to run their jobs in another region.
*   Impact Mitigation time: 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations will experience downgraded performance in europe-west2.
*   Workaround: None at this time.

**Cloud Filestore:**

*   Impact/Diagnosis: europe-west2-a is unavailable for all customers
*   Workaround: Customers are advised to fail over to another region if possible.

**Cloud SQL:**

*   Impact/Diagnosis: Non-HA instances backed by europe-west2-a are hard-down in europe-west2-a. HA instances that were in europe-west2-a when the incident started, are down with stuck failovers.
*   Workaround: None at this time.

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management api (create,update,delete,etc), and cache flushes
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.

**Persistent Disk:**

*   Impact/Diagnosis: Existing PD devices in europe-west2-a may be unavailable. Replicated PD devices in europe-west2 may see some replica unavailability.
*   Workaround: Create PD devices in a different region/zone.
*   Impact mitigated: Starting at 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c.

**API Gateway:**

*   Impact/Diagnosis: Customers may see elevated 5xx errors in europe-west2
*   Workaround: None at this time.
*   Impact Mitigation time: 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers will not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region, 45% for Multi Region . to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max) . Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can chose to route to europe-west1, no workarounds needed for europe-west2

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 will experience significantly increased failed requests.
*   Workaround: None at this time.

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers will experience high latency for all API requests. Some queues are not being loaded and they have stopped executing tasks. Customers may also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Workaround: None at this time.
*   Impact Mitigation time: 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers will be unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers will also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.

**Vertex AI online prediction**

*   Impact/Diagnosis: End user will experience timeouts
*   Workaround: None at this time.

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a currently inaccessible. Customers will also experience 100% packet loss to europe-west2-a. Customers are unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 12:49 US/Pacific

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: None at this time.

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming Listen and Write outage starting around 07:15 US/Pacific
*   Workaround: none
*   Impact Mitigation time: 09:55 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up may experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b , europe-west2-c or other regions.

We will provide more information by Tuesday, 2022-07-19 20:30 US/Pacific.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

19 Jul 2022 16:07 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is affecting multiple Cloud services.

Cooling system restoration in europe-west2-a has been completed.

GCP product teams are working on restoring their services. ETA to be determined.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region will see elevated errors when attempting to read objects
*   Workaround: None at this time.
*   Impact Mitigation time: 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets will be unavailable for the impacted locations.
*   Workaround: None at this time.
*   Impact Mitigation time: 13:43 US/Pacific

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers may experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Workaround: Customers are advised to fail over to another region if possible.
*   Impact Mitigation time: 11:05 US/Pacific

**DataFlow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may be stuck. New Dataflow jobs may not start.
*   Workaround: Customers are advised to run their jobs in another region.
*   Impact Mitigation time: 12:38 US/Pacific

**Cloud Composer:**

*   Impact/Diagnosis: All operations will experience downgraded performance in europe-west2.
*   Workaround: None at this time.

**Cloud Filestore:**

*   Impact/Diagnosis: europe-west2-a is unavailable for all customers
*   Workaround: Customers are advised to fail over to another region if possible.

**Cloud SQL:**

*   Impact/Diagnosis: Non-HA instances backed by europe-west2-a are hard-down in europe-west2-a. HA instances that were in europe-west2-a when the incident started, are down with stuck failovers.
*   Workaround: None at this time.

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management api (create,update,delete,etc), and cache flushes
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.

**Persistent Disk:**

*   Impact/Diagnosis: Existing PD devices in europe-west2-a may be unavailable. Replicated PD devices in europe-west2 may see some replica unavailability.
*   Workaround: Create PD devices in a different region/zone.
*   Impact mitigated: Starting at 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c.

**API Gateway:**

*   Impact/Diagnosis: Customers may see elevated 5xx errors in europe-west2
*   Workaround: None at this time.
*   Impact Mitigation time: 10:07 US/Pacific

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers will not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region, 45% for Multi Region . to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max) . Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can chose to route to europe-west1, no workarounds needed for europe-west2

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 will experience significantly increased failed requests.
*   Workaround: None at this time.

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers will experience high latency for all API requests. Some queues are not being loaded and they have stopped executing tasks. Customers may also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Workaround: None at this time.
*   Impact Mitigation time: 13:24 US/Pacific

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers will be unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers will also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.

**Vertex AI online prediction**

*   Impact/Diagnosis: End user will experience timeouts
*   Workaround: None at this time.

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a currently inaccessible. Customers will also experience 100% packet loss to europe-west2-a. Customers are unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.
*   Impact Mitigation time: 12:49 US/Pacific

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: None at this time.

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming Listen and Write outage starting around 07:15 US/Pacific
*   Workaround: none
*   Impact Mitigation time: 09:55 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up may experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b , europe-west2-c or other regions.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

19 Jul 2022 14:39 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: A cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2 is affecting multiple Cloud services.

Our engineering team is currently working to restore the cooling system in europe-west2-a.

Cooling system restoration in europe-west2-a is expected to be complete by Tuesday, 2022-07-19 16:30 US/Pacific.

Once the cooling system is restored, GCP product teams will work on restoring their services. ETA to be determined.

We will provide more information by Tuesday, 2022-07-19 16:30 US/Pacific.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region will see elevated errors when attempting to read objects
*   Workaround: None at this time.
*   Impact Mitigation time: 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets will be unavailable for the impacted locations.
*   Workaround: None at this time.

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers may experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Workaround: Customers are advised to fail over to another region if possible.
*   Impact Mitigation time: 11:05 US/Pacific

**DataFlow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may be stuck. New Dataflow jobs may not start.
*   Workaround: Customers are advised to run their jobs in another region.

**Cloud Composer:**

*   Impact/Diagnosis: All operations will experience downgraded performance in europe-west2.
*   Workaround: None at this time.

**Cloud Filestore:**

*   Impact/Diagnosis: europe-west2-a is unavailable for all customers
*   Workaround: Customers are advised to fail over to another region if possible.

**Cloud SQL:**

*   Impact/Diagnosis: Non-HA instances backed by europe-west2-a are hard-down in europe-west2-a. HA instances that were in europe-west2-a when the incident started, are down with stuck failovers.
*   Workaround: None at this time.

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management api (create,update,delete,etc), and cache flushes
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.

**Persistent Disk:**

*   Impact/Diagnosis: Existing PD devices in europe-west2-a may be unavailable. Replicated PD devices in europe-west2 may see some replica unavailability.
*   Workaround: Create PD devices in a different region/zone.
*   Impact mitigated: Starting at 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c.

**API Gateway:**

*   Impact/Diagnosis: Customers may see elevated 5xx errors in europe-west2
*   Workaround: None at this time.

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers will not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region, 45% for Multi Region . to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max) . Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can chose to route to europe-west1, no workarounds needed for europe-west2

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 will experience significantly increased failed requests.
*   Workaround: None at this time.

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers will experience high latency for all API requests. Some queues are not being loaded and they stop eeurope-west2-bcuting tasks. Customers may also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Workaround: None at this time.

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers will be unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers will also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.

**Vertex AI online prediction**

*   Impact/Diagnosis: End user will experience timeouts
*   Workaround: None at this time.

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a currently inaccessible. Customers will also experience 100% packet loss to europe-west2-a. Customers are unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: None at this time.

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming Listen and Write outage starting around 07:15 US/Pacific
*   Workaround: none
*   Impact Mitigation time: 09:55 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up may experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b , europe-west2-c or other regions.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

19 Jul 2022 14:18 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: Mitigation work is currently underway by our engineering team.

We do not have an ETA for mitigation at this point.

We will provide more information by Tuesday, 2022-07-19 15:00 US/Pacific.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region will see elevated errors when attempting to read objects
*   Workaround: None at this time.
*   Impact Mitigation time: 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets will be unavailable for the impacted locations.
*   Workaround: None at this time.

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers may experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Workaround: Customers are advised to fail over to another region if possible.
*   Impact Mitigation time: 11:05 US/Pacific

**DataFlow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may be stuck. New Dataflow jobs may not start.
*   Workaround: Customers are advised to run their jobs in another region.

**Cloud Composer:**

*   Impact/Diagnosis: All operations will experience downgraded performance in europe-west2.
*   Workaround: None at this time.

**Cloud Filestore:**

*   Impact/Diagnosis: europe-west2-a is unavailable for all customers
*   Workaround: Customers are advised to fail over to another region if possible.

**Cloud SQL:**

*   Impact/Diagnosis: Non-HA instances backed by europe-west2-a are hard-down in europe-west2-a. HA instances that were in europe-west2-a when the incident started, are down with stuck failovers.
*   Workaround: None at this time.

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management api (create,update,delete,etc), and cache flushes
*   Workaround: None at this time.

**MemoryStore for Memcached**

*   Impact/Diagnosis: Existing instances in europe-west2-a may be unavailable. Additionally, customers may experience degraded performance in europe-west2. Instance creation is not affected.
*   Workaround: None at this time.

**Persistent Disk:**

*   Impact/Diagnosis: Existing PD devices in europe-west2-a may be unavailable. Replicated PD devices in europe-west2 may see some replica unavailability.
*   Workaround: Create PD devices in a different region/zone.
*   Impact mitigated: Starting at 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c.

**API Gateway:**

*   Impact/Diagnosis: Customers may see elevated 5xx errors in europe-west2
*   Workaround: None at this time.

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers will not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region, 45% for Multi Region . to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max) . Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can chose to route to europe-west1, no workarounds needed for europe-west2

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 will experience significantly increased failed requests.
*   Workaround: None at this time.

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers will experience high latency for all API requests. Some queues are not being loaded and they stop eeurope-west2-bcuting tasks. Customers may also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Workaround: None at this time.

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers will be unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers will also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.

**Vertex AI online prediction**

*   Impact/Diagnosis: End user will experience timeouts
*   Workaround: None at this time.

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a currently inaccessible. Customers will also experience 100% packet loss to europe-west2-a. Customers are unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: None at this time.

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming Listen and Write outage starting around 07:15 US/Pacific
*   Workaround: none
*   Impact Mitigation time: 09:55 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up may experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b , europe-west2-c or other regions.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

19 Jul 2022 14:08 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: Mitigation work is currently underway by our engineering team.

We do not have an ETA for mitigation at this point.

We will provide more information by Tuesday, 2022-07-19 15:00 US/Pacific.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region will see elevated errors when attempting to read objects
*   Workaround: None at this time.
*   Impact Mitigation time: 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets will be unavailable for the impacted locations.
*   Workaround: None at this time.

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers may experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Workaround: Customers are advised to fail over to another region if possible.
*   Impact Mitigation time: 11:05 US/Pacific

**DataFlow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may be stuck. New Dataflow jobs may not start.
*   Workaround: Customers are advised to run their jobs in another region.

**Cloud Composer:**

*   Impact/Diagnosis: All operations will experience downgraded performance in europe-west2.
*   Workaround: None at this time.

**Cloud Filestore:**

*   Impact/Diagnosis: europe-west2-a is unavailable for all customers
*   Workaround: Customers are advised to fail over to another region if possible.

**Cloud SQL:**

*   Impact/Diagnosis: Non-HA instances backed by europe-west2-a are hard-down in europe-west2-a. HA instances that were in europe-west2-a when the incident started, are down with stuck failovers.
*   Workaround: None at this time.

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management api (create,update,delete,etc), and cache flushes
*   Workaround: None at this time.

**Persistent Disk:**

*   Impact/Diagnosis: Existing PD devices in europe-west2-a may be unavailable. Replicated PD devices in europe-west2 may see some replica unavailability.
*   Workaround: Create PD devices in a different region/zone.
*   Impact mitigated: Starting at 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c.

**API Gateway:**

*   Impact/Diagnosis: Customers may see elevated 5xx errors in europe-west2
*   Workaround: None at this time.

**Cloud Spanner**

*   Impact/Diagnosis: Most Cloud Spanner customers will not see any impact, but customers should stay below our recommended CPU usage 65% for Single Region, 45% for Multi Region . to not experience any increased latency. [https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max](https://cloud.google.com/spanner/docs/cpu-utilization#recommended-max) . Multi Region clients running in europe-west2 will see added latency.
*   Workaround: For eur5 customers can chose to route to europe-west1, no workarounds needed for europe-west2

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 will experience significantly increased failed requests.
*   Workaround: None at this time.

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers will experience high latency for all API requests. Some queues are not being loaded and they stop eeurope-west2-bcuting tasks. Customers may also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Workaround: None at this time.

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers will be unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers will also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.

**Vertex AI online prediction**

*   Impact/Diagnosis: End user will experience timeouts
*   Workaround: None at this time.

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a currently inaccessible. Customers will also experience 100% packet loss to europe-west2-a. Customers are unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: None at this time.

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming Listen and Write outage starting around 07:15 US/Pacific
*   Workaround: none
*   Impact Mitigation time: 09:55 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up may experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b , europe-west2-c or other regions.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

19 Jul 2022 13:40 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: Mitigation work is currently underway by our engineering team.

We do not have an ETA for mitigation at this point.

We will provide more information by Tuesday, 2022-07-19 14:10 US/Pacific.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region will see elevated errors when attempting to read objects
*   Workaround: None at this time.
*   Impact Mitigation time: 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets will be unavailable for the impacted locations.
*   Workaround: None at this time.

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers may experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Workaround: Customers are advised to fail over to another region if possible.
*   Impact Mitigation time: 11:05 US/Pacific

**DataFlow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may be stuck. New Dataflow jobs may not start.
*   Workaround: Customers are advised to run their jobs in another region.

**Cloud Composer:**

*   Impact/Diagnosis: All operations will experience downgraded performance in europe-west2.
*   Workaround: None at this time.

**Cloud Filestore:**

*   Impact/Diagnosis: europe-west2-a is unavailable for all customers
*   Workaround: Customers are advised to fail over to another region if possible.

**Cloud SQL:**

*   Impact/Diagnosis: Non-HA instances backed by europe-west2-a are hard-down in europe-west2-a. HA instances that were in europe-west2-a when the incident started, are down with stuck failovers.
*   Workaround: None at this time.

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management api (create,update,delete,etc), and cache flushes
*   Workaround: None at this time.

**Persistent Disk:**

*   Impact/Diagnosis: Existing PD devices in europe-west2-a may be unavailable. Replicated PD devices in europe-west2 may see some replica unavailability.
*   Workaround: Create PD devices in a different region/zone.
*   Impact mitigated: Starting at 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c.

**API Gateway:**

*   Impact/Diagnosis: Customers may see elevated 5xx errors in europe-west2
*   Workaround: None at this time.

**Cloud Spanner**

*   Impact/Diagnosis: After further investigation by our Cloud Spanner Engineering, we confirmed that there is no known impact to external customers.

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 will experience significantly increased failed requests.
*   Workaround: None at this time.

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers will experience high latency for all API requests. Some queues are not being loaded and they stop eeurope-west2-bcuting tasks. Customers may also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Workaround: None at this time.

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers will be unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers will also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.

**Vertex AI online prediction**

*   Impact/Diagnosis: End user will experience timeouts
*   Workaround: None at this time.

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a currently inaccessible. Customers will also experience 100% packet loss to europe-west2-a. Customers are unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: None at this time.

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming Listen and Write outage starting around 07:15 US/Pacific
*   Workaround: none
*   Impact Mitigation time: 09:55 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up may experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b , europe-west2-c or other regions.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

19 Jul 2022 13:28 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: Mitigation work is currently underway by our engineering team.

We do not have an ETA for mitigation at this point.

We will provide more information by Tuesday, 2022-07-19 14:00 US/Pacific.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region will see elevated errors when attempting to read objects
*   Workaround: None at this time.
*   Impact Mitigation time: 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets will be unavailable for the impacted locations.
*   Workaround: None at this time.

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers may experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Workaround: Customers are advised to fail over to another region if possible.
*   Impact Mitigation time: 11:05 US/Pacific

**DataFlow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may be stuck. New Dataflow jobs may not start.
*   Workaround: Customers are advised to run their jobs in another region.

**Cloud Composer:**

*   Impact/Diagnosis: All operations will experience downgraded performance in europe-west2.
*   Workaround: None at this time.

**Cloud Filestore:**

*   Impact/Diagnosis: europe-west2-a is unavailable for all customers
*   Workaround: Customers are advised to fail over to another region if possible.

**Cloud SQL:**

*   Impact/Diagnosis: Non-HA instances backed by europe-west2-a are hard-down in europe-west2-a. HA instances that were in europe-west2-a when the incident started, are down with stuck failovers. Failing ops include: backups, creates, DMS migrations, deletes, restarts, exports.
*   Workaround: None at this time.

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management api (create,update,delete,etc), and cache flushes
*   Workaround: None at this time.

**Persistent Disk:**

*   Impact/Diagnosis: Existing PD devices in europe-west2-a may be unavailable. Replicated PD devices in europe-west2 may see some replica unavailability.
*   Workaround: Create PD devices in a different region/zone.
*   Impact mitigated: Starting at 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c.

**API Gateway:**

*   Impact/Diagnosis: Customers may see elevated 5xx errors in europe-west2
*   Workaround: None at this time.

**Cloud Spanner**

*   Impact/Diagnosis: Customers that stay below our recommended CPU usage 65% SR, 45% MR should not see impact. Customers that are above the limit might see latency impact.
*   Workaround: For eur5 customers can chose to route to europe-west1, no workarounds needed for europe-west2

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 will experience significantly increased failed requests.
*   Workaround: None at this time.

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers will experience high latency for all API requests. Some queues are not being loaded and they stop eeurope-west2-bcuting tasks. Customers may also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Workaround: None at this time.

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers will be unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers will also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.

**Vertex AI online prediction**

*   Impact/Diagnosis: End user will experience timeouts
*   Workaround: None at this time.

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a currently inaccessible. Customers will also experience 100% packet loss to europe-west2-a. Customers are unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: None at this time.

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming Listen and Write outage starting around 07:15 US/Pacific
*   Workaround: none
*   Impact Mitigation time: 09:55 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up may experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b , europe-west2-c or other regions.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

19 Jul 2022 13:11 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: Mitigation work is currently underway by our engineering team.

We do not have an ETA for mitigation at this point.

We will provide more information by Tuesday, 2022-07-19 14:00 US/Pacific.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region will see elevated errors when attempting to read objects
*   Workaround: None at this time.
*   Impact Mitigation time: 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets will be unavailable for the impacted locations.
*   Workaround: None at this time.

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers may experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Workaround: Customers are advised to fail over to another region if possible.
*   Impact Mitigation time: 11:05 US/Pacific

**DataFlow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may be stuck. New Dataflow jobs may not start.
*   Workaround: Customers are advised to run their jobs in another region.

**Cloud Composer:**

*   Impact/Diagnosis: All operations will experience downgraded performance in europe-west2.
*   Workaround: None at this time.

**Cloud Filestore:**

*   Impact/Diagnosis: europe-west2-a is unavailable for all customers
*   Workaround: Customers are advised to fail over to another region if possible.

**Cloud SQL:**

*   Impact/Diagnosis: Non-HA instances backed by europe-west2-a are hard-down in europe-west2-a. HA instances that were in europe-west2-a when the incident started, are down with stuck failovers. Failing ops include: backups, creates, DMS migrations, deletes, restarts, exports.
*   Workaround: None at this time.

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management api (create,update,delete,etc), and cache flushes
*   Workaround: None at this time.

**Persistent Disk:**

*   Impact/Diagnosis: Existing PD devices in europe-west2-a may be unavailable. Replicated PD devices in europe-west2 may see some replica unavailability.
*   Workaround: Create PD devices in a different region/zone.
*   Impact mitigated: Starting at 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c.

**API Gateway:**

*   Impact/Diagnosis: Customers may see elevated 5xx errors in europe-west2
*   Workaround: None at this time.

**Cloud Spanner**

*   Impact/Diagnosis: Customers that stay below our recommended CPU usage 65% SR, 45% MR should not see impact. Customers that are above the limit might see latency impact.
*   Workaround: For eur5 customers can chose to route to europe-west1, no workarounds needed for europe-west2

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 will experience significantly increased failed requests.
*   Workaround: None at this time.

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers will experience high latency for all API requests. Some queues are not being loaded and they stop eeurope-west2-bcuting tasks. Customers may also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Workaround: None at this time.

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2europe-west2-bDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2europe-west2-bDSqLtJZUvcH)
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2europe-west2-bDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2europe-west2-bDSqLtJZUvcH)

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers will be unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers will also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.

**Vertex AI online prediction**

*   Impact/Diagnosis: End user will experience timeouts
*   Workaround: None at this time.

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a currently inaccessible. Customers will also experience 100% packet loss to europe-west2-a. Customers are unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: None at this time.

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming Listen and Write outage starting around 07:15 US/Pacific
*   Workaround: none
*   Impact Mitigation time: 09:55 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up may experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b , europe-west2-c or other regions.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

19 Jul 2022 12:43 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: Mitigation work is currently underway by our engineering team.

We do not have an ETA for mitigation at this point.

We will provide more information by Tuesday, 2022-07-19 13:30 US/Pacific.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region will see elevated errors when attempting to read objects
*   Workaround: None at this time.
*   Impact Mitigation time: 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets will be unavailable for the impacted locations.
*   Workaround: None at this time.

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers may experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Workaround: Customers are advised to fail over to another region if possible.
*   Impact Mitigation time: 11:05 US/Pacific

**DataFlow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may be stuck. New Dataflow jobs may not start.
*   Workaround: Customers are advised to run their jobs in another region.

**Cloud Composer:**

*   Impact/Diagnosis: All operations will experience downgraded performance in europe-west2.
*   Workaround: None at this time.

**Cloud Filestore:**

*   Impact/Diagnosis: europe-west2-a is unavailable for all customers
*   Workaround: Customers are advised to fail over to another region if possible.

**Cloud SQL:**

*   Impact/Diagnosis: Non-HA instances backed by europe-west2-a are hard-down in europe-west2-a. Some HA instances are down in europe-west2-b/c (those with standby in europe-west2-a backed by europe-west2-a). Following operations are failing: Backups, creates, DMS migrations, updates, deletes, recreates (internal), restarts, exports.
*   Workaround: None at this time.

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management api (create,update,delete,etc), and cache flushes
*   Workaround: None at this time.

**Persistent Disk:**

*   Impact/Diagnosis: Existing PD devices in europe-west2-a may be unavailable. Replicated PD devices in europe-west2 may see some replica unavailability.
*   Workaround: Create PD devices in a different region/zone.
*   Impact mitigated: Starting at 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c.

**API Gateway:**

*   Impact/Diagnosis: Customers may see elevated 5xx errors in europe-west2
*   Workaround: None at this time.

**Cloud Spanner**

*   Impact/Diagnosis: Customers that stay below our recommended CPU usage 65% SR, 45% MR should not see impact. Customers that are above the limit might see latency impact.
*   Workaround: For eur5 customers can chose to route to europe-west1, no workarounds needed for europe-west2

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 will experience significantly increased failed requests.
*   Workaround: None at this time.

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers will experience high latency for all API requests. Some queues are not being loaded and they stop eeurope-west2-bcuting tasks. Customers may also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Workaround: None at this time.

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2europe-west2-bDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2europe-west2-bDSqLtJZUvcH)
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2europe-west2-bDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2europe-west2-bDSqLtJZUvcH)

**Cloud Data Fusion:**

*   Impact/Diagnosis : Service unavailability in europe-west2 for a very few customers.
*   Workaround: None at this time.

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers will be unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers will also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.

**Vertex AI online prediction**

*   Impact/Diagnosis: End user will experience timeouts
*   Workaround: None at this time.

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a currently inaccessible. Customers will also experience 100% packet loss to europe-west2-a. Customers are unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: None at this time.

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming Listen and Write outage starting around 07:15 US/Pacific
*   Workaround: none
*   Impact Mitigation time: 09:55 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up may experience elevated error rate in europe-west2-a.
*   Workaround: Customers can choose a europe-west2-b , europe-west2-c or other regions.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

19 Jul 2022 12:32 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: We are experiencing an issue with multiple cloud products beginning on Tuesday, 2022-07-19 06:33 US/Pacific.

Our engineering team continues to investigate the issue.

We will provide an update by Tuesday, 2022-07-19 13:15 US/Pacific with current details.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region will see elevated errors when attempting to read objects
*   Workaround: None at this time.
*   Impact Mitigation time: 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets will be unavailable for the impacted locations.
*   Workaround: None at this time.

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers may experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Workaround: Customers are advised to fail over to another region if possible.
*   Impact Mitigation time: 11:05 US/Pacific

**DataFlow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may be stuck. New Dataflow jobs may not start.
*   Workaround: Customers are advised to run their jobs in another region.

**Cloud Composer:**

*   Impact/Diagnosis: All operations will experience downgraded performance in europe-west2.
*   Workaround: None at this time.

**Cloud Filestore:**

*   Impact/Diagnosis: europe-west2-a is unavailable for all customers
*   Workaround: Customers are advised to fail over to another region if possible.

**Cloud SQL:**

*   Impact/Diagnosis: Non-HA instances backed by europe-west2-a are hard-down in europe-west2-a. Some HA instances are down in europe-west2-b/c (those with standby in europe-west2-a backed by europe-west2-a). Following operations are failing: Backups, creates, DMS migrations, updates, deletes, recreates (internal), restarts, exports.
*   Workaround: None at this time.

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management api (create,update,delete,etc), and cache flushes
*   Workaround: None at this time.

**Persistent Disk:**

*   Impact/Diagnosis: Existing PD devices in europe-west2-a may be unavailable. Replicated PD devices in europe-west2 may see some replica unavailability.
*   Workaround: Create PD devices in a different region/zone.
*   Impact mitigated: Starting at 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c.

**API Gateway:**

*   Impact/Diagnosis: Customers may see elevated 5xx errors in europe-west2
*   Workaround: None at this time.

**Cloud Spanner**

*   Impact/Diagnosis: Customers that stay below our recommended CPU usage 65% SR, 45% MR should not see impact. Customers that are above the limit might see latency impact.
*   Workaround: For eur5 customers can chose to route to europe-west1, no workarounds needed for europe-west2

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 will experience significantly increased failed requests.
*   Workaround: None at this time.

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers will experience high latency for all API requests. Some queues are not being loaded and they stop eeurope-west2-bcuting tasks. Customers may also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Workaround: None at this time.

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2europe-west2-bDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2europe-west2-bDSqLtJZUvcH)
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2europe-west2-bDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2europe-west2-bDSqLtJZUvcH)

**Cloud Data Fusion:**

*   Impact/Diagnosis : Customers in europe-west2 may see service disruption when they shut down europe-west2-a and may also see pipeline failures.
*   Workaround: None at this time.

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers will be unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers will also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.

**Vertex AI online prediction**

*   Impact/Diagnosis: End user will experience timeouts
*   Workaround: None at this time.

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a currently inaccessible. Customers will also experience 100% packet loss to europe-west2-a. Customers are unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.

**Bigtable:**

*   Impact/Diagnosis: Service unavailability and elevated latency for some customers in europe-west2.
*   Workaround: None at this time.

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming Listen and Write outage starting around 07:15 US/Pacific
*   Workaround: none
*   Impact Mitigation time: 09:55 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up may experience elevated error rate in europe-west2-a and europe-west2-c.
*   Workaround: Customers can choose a europe-west2-b or other regions.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

19 Jul 2022 12:21 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: We are experiencing an issue with multiple cloud products beginning on Tuesday, 2022-07-19 06:33 US/Pacific.

Our engineering team continues to investigate the issue.

We will provide an update by Tuesday, 2022-07-19 13:00 US/Pacific with current details.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region will see elevated errors when attempting to read objects
*   Workaround: None at this time.
*   Impact Mitigation time: 08:53 US/Pacific

**Google BigQuery:**

*   Impact/Diagnosis: Datasets will be unavailable for the impacted locations.
*   Workaround: None at this time.

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers may experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Workaround: Customers are advised to fail over to another region if possible.
*   Impact Mitigation time: 11:05 US/Pacific

**DataFlow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may be stuck. New Dataflow jobs may not start.
*   Workaround: Customers are advised to run their jobs in another region.

**Cloud Composer:**

*   Impact/Diagnosis: All operations will experience downgraded performance in europe-west2.
*   Workaround: None at this time.

**Cloud Filestore:**

*   Impact/Diagnosis: europe-west2-a is unavailable for all customers
*   Workaround: Customers are advised to fail over to another region if possible.

**Cloud SQL:**

*   Impact/Diagnosis: Non-HA instances backed by europe-west2-a are hard-down in europe-west2-a. Some HA instances are down in europe-west2-b/c (those with standby in europe-west2-a backed by europe-west2-a). Following operations are failing: Backups, creates, DMS migrations, updates, deletes, recreates (internal), restarts, exports.
*   Workaround: None at this time.

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management api (create,update,delete,etc), and cache flushes
*   Workaround: None at this time.

**Persistent Disk:**

*   Impact/Diagnosis: Existing PD devices in europe-west2-a may be unavailable. Replicated PD devices in europe-west2 may see some replica unavailability.
*   Workaround: Create PD devices in a different region/zone.
*   Impact mitigated: Starting at 11:18 US/Pacific, customers should be able to create new PD devices in europe-west2-b and europe-west2-c.

**API Gateway:**

*   Impact/Diagnosis: Customers may see elevated 5xx errors in europe-west2
*   Workaround: None at this time.

**Cloud Spanner**

*   Impact/Diagnosis: Customers that stay below our recommended CPU usage 65% SR, 45% MR should not see impact. Customers that are above the limit might see latency impact.
*   Workaround: For eur5 customers can chose to route to europe-west1, no workarounds needed for europe-west2

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 will experience significantly increased failed requests.
*   Workaround: None at this time.

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers will experience high latency for all API requests. Some queues are not being loaded and they stop eeurope-west2-bcuting tasks. Customers may also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Workaround: None at this time.

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2europe-west2-bDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2europe-west2-bDSqLtJZUvcH)
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2europe-west2-bDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2europe-west2-bDSqLtJZUvcH)

**Cloud Data Fusion:**

*   Impact/Diagnosis : Customers in europe-west2 may see service disruption when they shut down europe-west2-a and may also see pipeline failures.
*   Workaround: None at this time.

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers will be unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers will also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.

**Vertex AI online prediction**

*   Impact/Diagnosis: End user will experience timeouts
*   Workaround: None at this time.

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a currently inaccessible. Customers will also experience 100% packet loss to europe-west2-a. Customers are unable to make any control plane changes in europe-west2-a.
*   Workaround: None at this time.

**Bigtable:**

*   Impact/Diagnosis: Service unavailability for very few customers in europe-west2.
*   Workaround: None at this time.

**Cloud Firestore:**

*   Impact/Diagnosis: Webchannel use of Firestore streaming Listen and Write outage starting around 07:15 US/Pacific
*   Workaround: none
*   Impact Mitigation time: 09:55 US/Pacific

**Cloud Dataproc:**

*   Impact/Diagnosis: Dataproc cluster creation and scale up may experience elevated error rate in europe-west2-c.
*   Workaround: Customers can choose a different zone.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

19 Jul 2022 11:48 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: We are experiencing an issue with multiple cloud products beginning on Tuesday, 2022-07-19 06:33 US/Pacific.

Our engineering team continues to investigate the issue.

We will provide an update by Tuesday, 2022-07-19 12:20 US/Pacific with current details.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region will see elevated errors when attempting to read objects
*   Workaround: None at this time.
*   Impact mitigated at 08:53 US/Pacific for Google Cloud Storage

**Google BigQuery:**

*   Impact/Diagnosis: Datasets will be unavailable for the impacted locations.
*   Workaround: None at this time.

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers may experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Workaround: Customers are advised to fail over to another region if possible.

**DataFlow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may be stuck. New Dataflow jobs may not start.
*   Workaround: Customers are advised to run their jobs in another region.

**Cloud Composer:**

*   Impact/Diagnosis: All operations will experience downgraded performance in europe-west2.
*   Workaround: None at this time.

**Cloud Filestore:**

*   Impact/Diagnosis: europe-west2-a is unavailable for all customers
*   Workaround: Customers are advised to fail over to another region if possible.

**Cloud SQL:**

*   Impact/Diagnosis: Non-HA instances are hard-down in europe-west2-a. HA instances are down in europe-west2-a (pending zonal failover). Following ops are failing in all three zones, but mostly in europe-west2-a: Backups, creates, DMS migrations, updates, deletes, recreates (internal), restarts, exports.
*   Workaround: None at this time.

**MemoryStore for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management api (create,update,delete,etc), and cache flushes
*   Workaround: None at this time.

**Persistent Disk:**

*   Impact/Diagnosis: Customers won't be able to create any PD device in europe-west2. PD devices in europe-west2-a may be unavailable.
*   Workaround: Create PD devices in a different region/zone.

**API Gateway:**

*   Impact/Diagnosis: Customers may see elevated 5xx errors in europe-west2
*   Workaround: None at this time.

**Cloud Spanner**

*   Impact/Diagnosis: Customers that stay below our recommended CPU usage 65% SR, 45% MR should not see impact. Customers that are above the limit might see latency impact.
*   Workaround: For eur5 customers can chose to route to europe-west1, no workarounds needed for europe-west2

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 will experience significantly increased failed requests.
*   Workaround: None at this time.

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers will experience high latency for all API requests. Some queues are not being loaded and they stop executing tasks. Customers may also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Workaround: None at this time.

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)

**Cloud Data Fusion:**

*   Impact/Diagnosis : Customers in europe-west2 may see service disruption when they shut down europe-west2-a and may also see pipeline failures.
*   Workaround: None at this time.

**Managed Service for Microsoft Active Directory:**

*   Impact/Diagnosis: Customers will be unable to perform any operations on Managed Active Directory (AD) domains which are single region (europe-west2). Customers will also experience a degraded experience if one domain controller is unavailable due to zonal impact.
*   Workaround: None at this time.

**Vertex AI online prediction**

*   Impact/Diagnosis: End user will experience timeouts
*   Workaround: None at this time.

**VPC (Traffic Virtnet)**

*   Impact/Diagnosis: VPCs in europe-west2-a currently inaccessible. Customers will also experience 100% packet loss to europe-west2-a.
*   Workaround: None at this time.

**Bigtable:**

*   Impact/Diagnosis: Service unavailability for very few customers in europe-west2.
*   Workaround: None at this time.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

19 Jul 2022 11:05 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: We are experiencing an issue with multiple cloud products beginning on Tuesday, 2022-07-19 06:33 US/Pacific.

Our engineering team continues to investigate the issue.

We will provide an update by Tuesday, 2022-07-19 11:45 US/Pacific with current details.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region will see elevated errors when attempting to read objects
*   Workaround: None at this time.
*   Impact mitigated at 08:53 US/Pacific for Google Cloud Storage

**Google BigQuery:**

*   Impact/Diagnosis: Datasets will be unavailable for the impacted locations.
*   Workaround: None at this time.

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers may experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Workaround: Customers are advised to fail over to another region if possible.

**DataFlow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may be stuck. New Dataflow jobs may not start.
*   Workaround: Customers are advised to run their jobs in another region.

**Cloud Composer:**

*   Impact/Diagnosis: All operations will experience downgraded performance in europe-west2.
*   Workaround: None at this time.

**Cloud Filestore:**

*   Impact/Diagnosis: europe-west2-a is unavailable for all customers
*   Workaround: Customers are advised to fail over to another region if possible.

**Cloud SQL:**

*   Impact/Diagnosis: Non-HA instances are hard-down in europe-west2-a. HA instances are down in europe-west2-a (pending zonal failover). Following ops are failing in all three zones, but mostly in europe-west2-a: Backups, creates, DMS migrations, updates, deletes, recreates (internal), restarts, exports.
*   Workaround: None at this time.

**Memory Store for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management api (create,update,delete,etc), and cache flushes
*   Workaround: None at this time.

**Persistent Disk:**

*   Impact/Diagnosis: Customers won't be able to create any PD device in europe-west2. PD devices in europe-west2-a may be unavailable.
*   Workaround: Create PD devices in a different region/zone.

**API Gateway:**

*   Impact/Diagnosis: Customers may see elevated 5xx errors in europe-west2
*   Workaround: None at this time.

**Cloud Spanner**

*   Impact/Diagnosis: Customers that stay below our recommended CPU usage 65% SR, 45% MR should not see impact. Customers that are above the limit might see latency impact.
*   Workaround: For eur5 customers can chose to route to europe-west1, no workarounds needed for europe-west2

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 will experience significantly increased failed requests.
*   Workaround: None at this time.

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers will experience high latency for all API requests. Some queues are not being loaded and they stop executing tasks. Customers may also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Workaround: None at this time.

**Google Compute Engine:**

*   Impact/Diagnosis: The impact details for GCE are externalized at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)
*   Workaround: Please refer to GCE posting at [https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH](https://status.cloud.google.com/incidents/XVq5om2XEDSqLtJZUvcH)

**Cloud Data Fusion:**

*   Impact/Diagnosis : Customers in europe-west2 will not be able to create new pipelines
*   Workaround: None at this time.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

19 Jul 2022 10:35 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: We are experiencing an issue with multiple cloud products beginning on Tuesday, 2022-07-19 06:33 US/Pacific.

Our engineering team continues to investigate the issue.

We will provide an update by Tuesday, 2022-07-19 11:10 US/Pacific with current details.

We apologize to all who are affected by the disruption.

Product Impact:

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region will see elevated errors when attempting to read objects
*   Workaround: None at this time.
*   Impact mitigated at 08:53 Us/pacific for Google Cloud Storage

**Google BigQuery:**

*   Impact/Diagnosis: Datasets will be unavailable for the impacted locations.
*   Workaround: None at this time.

_Google App Engine and Cloud Functions:_\*

*   Impact/Diagnosis: Customers may experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Workaround: Fail over to another region if your service supports it.

**DataFlow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may be stuck. New Dataflow jobs may not start.
*   Workaround: Customers are advised to run their jobs in another region.

**Cloud Composer:**

*   Impact/Diagnosis: All operations will experience downgraded performance in europe-west2.
*   Workaround: None at this time.

**Cloud Filestore:**

*   Impact/Diagnosis: europe-west2-a is unavailable for all customers
*   Workaround: Fail over to another region if your service supports it

**Cloud SQL:**

*   Impact/Diagnosis: Non-HA instances are hard-down in europe-west2-a. HA instances are down in europe-west2-a (pending zonal failover). Following ops are failing in all three zones, but mostly in europe-west2-a: Backups, creates, DMS migrations, updates, deletes, recreates (internal), restarts, exports.
*   Workaround: None at this time.

**Memory Store for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management api (create,update,delete,etc), and cache flushes
*   Workaround: None at this time.

**Persistent Disk:**

*   Impact/Diagnosis: Customers won't be able to create any PD device in europe-west2. PD devices in europe-west2-a are unavailable.
*   Workaround: Create PD devices in a different region/zone.

**API Gateway:**

*   Impact/Diagnosis: Customers may see elevated 5xx errors in europe-west2
*   Workaround: None at this time.

**Cloud Spanner**

*   Impact/Diagnosis: Customers that stay below our recommended CPU usage 65% SR, 45% MR should not see impact. Customers that are above the limit might see latency impact.
*   Workaround: For eur5 customers can chose to route to europe-west1, no workarounds needed for europe-west2

**GKE:**

*   Impact/Diagnosis: Certain portions of clusters with presence in europe-west2-a may be unavailable
*   Workaround: Customers are encouraged to move their workloads to other regions if possible.

**Looker:**

*   Impact/Diagnosis: Customers hosted in europe-west2 will experience significantly increased failed requests.
*   Workaround: None at this time.

**Google Cloud Tasks:**

*   Impact/Diagnosis: Customers will experience high latency for all API requests. Some queues are not being loaded and they stop executing tasks. Customers may also see high delivery latency for their tasks, depending on which cell is serving their queues.
*   Workaround: None at this time.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

19 Jul 2022 10:07 PDT

Summary: Multiple Cloud products experiencing elevated error rates, latencies or service unavailability in europe-west2

Description: We are experiencing an issue with multiple cloud products beginning on Tuesday, 2022-07-19 06:33 US/Pacific.

Our engineering team continues to investigate the issue.

We will provide an update by Tuesday, 2022-07-19 10:40 US/Pacific with current details.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region will see elevated errors when attempting to read objects
*   Workaround: None at this time.
*   Impact mitigated at 08:53 Us/pacific for Google Cloud Storage

**Google BigQuery:**

*   Impact/Diagnosis: Datasets will be unavailable for the impacted locations.
*   Workaround: None at this time.

_Google App Engine and Cloud Functions:_\*

*   Impact/Diagnosis: Customers may experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Workaround: Fail over to another region if your service supports it.

**DataFlow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may be stuck. New Dataflow jobs may not start.
*   Workaround: Customers are advised to run their jobs in another region.

**Cloud Composer:**

*   Impact/Diagnosis: All operations will experience downgraded performance in europe-west2.
*   Workaround: None at this time.

**Cloud Filestore:**

*   Impact/Diagnosis: europe-west2-a is unavailable for all customers
*   Workaround: Fail over to another region if your service supports it

**Cloud SQL:**

*   Impact/Diagnosis: Non-HA instances are hard-down in europe-west2-a. HA instances are down in europe-west2-a (pending zonal failover). Following ops are failing in all three zones, but mostly in europe-west2-a: Backups, creates, DMS migrations, updates, deletes, recreates (internal), restarts, exports.
*   Workaround: None at this time.

**Memory Store for Redis:**

*   Impact/Diagnosis: Customers will experience timeouts for management api (create,update,delete,etc), and cache flushes
*   Workaround: None at this time.

**Persistent Disk:**

*   Impact/Diagnosis: Customers won't be able to create any PD device in europe-west2. PD devices in europe-west2-a are unavailable.
*   Workaround: Create PD devices in a different region/zone.

**API Gateway:**

*   Impact/Diagnosis: Customers may see elevated 5xx errors in europe-west2
*   Workaround: None at this time.

**Cloud Spanner**

*   Impact/Diagnosis: Customers that stay below our recommended CPU usage 65% SR, 45% MR should not see impact. Customers that are above the limit might see latency impact.
*   Workaround: For eur5 customers can chose to route to europe-west1, no workarounds needed for europe-west2

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

19 Jul 2022 09:38 PDT

Summary: Multiple Cloud products experiencing elevated error rates in europe-west2

Description: We are experiencing an issue with multiple cloud products beginning on Tuesday, 2022-07-19 06:33 US/Pacific.

Our engineering team continues to investigate the issue.

We will provide an update by Tuesday, 2022-07-19 10:10 US/Pacific with current details.

We apologize to all who are affected by the disruption.

**Product Impact:**

**Google Cloud Storage:**

*   Impact/Diagnosis: Customers with data in the impacted region will see elevated errors when attempting to read objects
*   Workaround: None at this time.

**Google BigQuery:**

*   Impact/Diagnosis: Datasets will be unavailable for the impacted cells.
*   Workaround: None at this time.

**Cloud Filestore:**

*   Impact/Diagnosis: europe-west2-a is unavailable for all customers
*   Workaround: Fail over to another region if your service supports it

**Google App Engine and Cloud Functions:**

*   Impact/Diagnosis: Customers may experience high error rates for Google App Engine and Cloud Functions in europe-west2 for requests coming through Cloud Pubsub, Eventarc, Cloud Tasks, and Cloud Scheduler. Pubsub, Eventarc, and Cloud Tasks traffic will be automatically retried once the outage is resolved. Cloud Scheduler requests will need to be resubmitted.
*   Workaround: Fail over to another region if your service supports it.

**DataFlow:**

*   Impact/Diagnosis: Dataflow Streaming jobs already running in the impacted area may be stuck. New Dataflow jobs may not start.
*   Workaround: Customers are advised to run their jobs in another region.

**Cloud Composer:**

*   Impact/Diagnosis: All operations will experience downgraded performance in europe-west2.
*   Workaround: None at this time.

Diagnosis: Product specific symptoms are in the issue description.

Workaround: Product specific workaround are in the issue description.

19 Jul 2022 09:15 PDT

Summary: Multiple Cloud products experiencing elevated error rates in europe-west2

Description: We are experiencing an issue with Google Cloud Storage, Google BigQuery, Google App Engine, Google Cloud Functions, Google Cloud Dataflow beginning at Tuesday, 2022-07-19 06:33 US/Pacific .

Our engineering team continues to investigate the issue.

We will provide an update by Tuesday, 2022-07-19 09:45 US/Pacific with current details.

We apologize to all who are affected by the disruption.

We apologize to all who are affected by the disruption.

Diagnosis: None at this time.

Workaround: None at this time.

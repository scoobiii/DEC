# Google Cloud Service Health 
29 Jul 2022 14:05 PDT

For a full Incident Report, please refer to [https://status.cloud.google.com/incidents/fmEL9i2fArADKawkZAa2](https://status.cloud.google.com/incidents/fmEL9i2fArADKawkZAa2)

21 Jul 2022 12:06 PDT

For a preliminary Incident Report (Mini IR) , please refer to [https://status.cloud.google.com/incidents/fmEL9i2fArADKawkZAa2](https://status.cloud.google.com/incidents/fmEL9i2fArADKawkZAa2)

19 Jul 2022 20:45 PDT

There was a cooling related failure in one of our buildings that hosts a portion of capacity for zone europe-west2-a for region europe-west2 that is now resolved. GCE, Persistent Disk and Autoscaling impacts have been addressed. Customers can launch VMs in all zones of europe-west2. A small number of HDD backed Persistent Disk volumes are still experiencing impact and will exhibit IO errors. If you are continuing to experience issues with these services, please contact Google Cloud Product Support and reference this message.

The issue has been resolved for all affected users as of Tuesday, 2022-07-19 20:43 US/Pacific.

We thank you for your patience while we worked on resolving the issue.

19 Jul 2022 15:29 PDT

Summary: Cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2.

Description: Mitigation work is currently underway by our engineering team.

We do not have an ETA for mitigation at this point.

There is a cooling related failure in one of our buildings that hosts a portion of capacity for zone europe-west2-a for region europe-west2 that is not yet completely resolved. GCE, Persistent Disk and Autoscaling impacts have been mitigated. Most customers can launch VMs in all zones of europe-west2. A minority of customers might continue to see failures in europe-west2-a. We continue to work on bringing back previously impacted capacity in europe-west2-a to fully restore GCE, Persistent Disk and Autoscaling. If you are continuing to experience issues with these services, please contact Google Cloud Product Support and reference this message.

Customers who launched new VMs in zones europe-west2-b and europe-west2-c and would like to delete their previously running VMs in europe-west2-a, can delete VMs via the console or GCE APIs. There might be a delay in processing the deletion and all deletions will be fully processed when all issues in europe-west2-a are resolved.

Diagnosis: Customers impacted by this issue would have seen abnormal VM terminations for a small set of their VMs.

Workaround: Customers who launched new VMs in zones europe-west2-b and europe-west2-c and would like to delete their previously running VMs in europe-west2-a, can delete VMs via the console or GCE APIs. There might be a delay in processing the deletion and all deletions will be fully processed when all issues in europe-west2-a are resolved.

19 Jul 2022 14:08 PDT

Summary: Cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2.

Description: Mitigation work is currently underway by our engineering team.

We do not have an ETA for mitigation at this point.

There is a cooling related failure in one of our buildings that hosts a portion of capacity for zone europe-west2-a for region europe-west2 that is not yet completely resolved.

GCE, Persistent Disk and Autoscaling impacts have been mitigated. Most customers can launch VMs in all zones of europe-west2.

A minority of customers might continue to see failures in europe-west2-a.

We continue to work on bringing back previously impacted capacity in europe-west2-a to fully restore GCE, Persistent Disk and Autoscaling. If you are continuing to experience issues with these services, please contact Google Cloud Product Support and reference this message.

Please refer to workaround section for additional information about VM deletion.

We will provide more information by Tuesday, 2022-07-19 16:00 US/Pacific.

Diagnosis: Customers impacted by this issue would have seen abnormal VM terminations for a small set of their VMs.

Workaround: Customers who launched new VMs in zones europe-west2-b and europe-west2-c and would like to delete their previously running VMs in europe-west2-a, can delete VMs via the console or GCE APIs. There might be a delay in processing the deletion and all deletions will be fully processed when all issues in europe-west2-a are resolved.

19 Jul 2022 12:29 PDT

Summary: Cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2. Europe-west2-b and europe-west2-c are not impacted for VMs. We have fixed the previously occurring issues when creating new Persistent Disk devices. Zonal autoscaling for europe-west2-a is impacted for customers who suffered VM terminations.

Description: We are experiencing an issue with Google Compute Engine beginning at Tuesday, 2022-07-19 08:10 US/Pacific.

Our engineering team continues to investigate the issue.

There has been a cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2. This caused a partial failure of capacity in that zone, leading to VM terminations and a loss of machines for a small set of our customers. We’re working hard to get the cooling back on-line and create capacity in that zone. We do not anticipate further impact in zone europe-west2-a and currently running VMs should not be impacted. A small percentage of replicated Persistent Disk devices are running in single redundant mode.

In order to prevent damage to machines and an extended outage, we have powered down part of the zone and are limiting GCE preemptible launches. We are working to restore redundancy for any remaining impacted replicated Persistent Disk devices.

We will provide an update by Tuesday, 2022-07-19 15:00 US/Pacific with current details. We apologize to all who are affected by the disruption.

Diagnosis: Customers impacted by this issue would have seen abnormal VM terminations for a small set of their VMs. We’re working on identifying and communicating with all affected customers.

Workaround: For customers who have experienced a VM failure in europe-west2-a, we recommend launching new instances in europe-west2-b and europe-west2-c. Customers who currently have running instances in europe-west2-a can continue to use Auto Scaling in europe-west2-a. Regional Auto Scaling continues to work.

19 Jul 2022 12:05 PDT

Summary: Cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2. Europe-west2-b and europe-west2-c are not impacted for VMs. We have fixed the previously occurring issues when creating new Persistent Disk devices. Zonal autoscaling for europe-west2-a is impacted.

Description: We are experiencing an issue with Google Compute Engine beginning at Tuesday, 2022-07-19 08:10 US/Pacific.

Our engineering team continues to investigate the issue.

There has been a cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2. This caused a partial failure of capacity in that zone, leading to VM terminations and a loss of machines for a small set of our customers. We’re working hard to get the cooling back on-line and create capacity in that zone. We do not anticipate further impact in zone europe-west2-a and currently running VMs should not be impacted. A small percentage of replicated Persistent Disk devices are running in single redundant mode.

In order to prevent damage to machines and an extended outage, we have powered down part of the zone and are limiting GCE preemptible launches. We are working to restore redundancy for any remaining impacted replicated Persistent Disk devices.

We will provide an update by Tuesday, 2022-07-19 14:00 US/Pacific with current details. We apologize to all who are affected by the disruption.

Diagnosis: Customers impacted by this issue would have seen abnormal VM terminations for a small set of their VMs. We’re working on identifying and communicating with all affected customers.

Workaround: For customers who have experienced a VM failure in europe-west2-a, we recommend launching new instances in europe-west2-b and europe-west2-c.

19 Jul 2022 11:36 PDT

Summary: Cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2. The other zones are not impacted for VMs. We are seeing regional impact for a small proportion of newly launched Persistent Disk volumes.

Description: We are experiencing an issue with Google Compute Engine beginning at Tuesday, 2022-07-19 08:10 US/Pacific.

Our engineering team continues to investigate the issue.

There has been a cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2. This caused a partial failure of capacity in that zone, leading to VM terminations and a loss of machines for a small set of our customers. We’re working hard to get the cooling back on-line and create capacity in that zone. We do not anticipate further impact in zone europe-west2-a and currently running VMs should not be impacted. A small percentage of replicated Persistent Disk devices are now running in single redundant mode.

In order to prevent damage to machines and an extended outage, we have powered down part of the zone and are limiting GCE preemptible launches. We are seeing regional impact for a small proportion of newly launched Persistent Disk volumes and are working to restore redundancy for the impacted replicated Persistent Disk devices.

We will provide an update by Tuesday, 2022-07-19 13:45 US/Pacific with current details. We apologize to all who are affected by the disruption.

Diagnosis: Customers impacted by this issue would have seen abnormal VM terminations for a small set of their VMs. We’re working on identifying and communicating with all affected customers.

Workaround: For customers who have experienced a VM failure in europe-west2-a, we recommend launching new instances in other zones of europe-west2.

19 Jul 2022 10:41 PDT

Summary: Cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2. The other zones are not impacted for VMs. We are seeing regional impact for a small proportion of newly launched Persistent Disk volumes.

Description: We are experiencing an issue with Google Compute Engine beginning at Tuesday, 2022-07-19 08:10 US/Pacific.

Our engineering team continues to investigate the issue.

There has been a cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2. This caused a partial failure of capacity in that zone, leading to VM terminations and a loss of machines for a small set of our customers. We’re working hard to get the cooling back on-line and create capacity in that zone. We do not anticipate further impact in zone europe-west2-a and currently running VMs should not be impacted. A small percentage of replicated Persistent Disk devices are now running in single redundant mode.

In order to prevent damage to machines and an extended outage, we have powered down part of the zone and are limiting GCE preemptible launches. We are seeing regional impact for a small proportion of newly launched Persistent Disk volumes and are working to restore redundancy for the impacted replicated Persistent Disk devices.

We will provide an update by Tuesday, 2022-07-19 11:45 US/Pacific with current details.

We apologize to all who are affected by the disruption.

Diagnosis: Customers impacted by this issue would have seen abnormal VM terminations for a small set of their VMs. We’re working on identifying and communicating with all affected customers.

Workaround: For customers who have experienced a VM failure in europe-west2-a, we recommend launching new instances in other zones of europe-west2.

19 Jul 2022 10:13 PDT

Summary: Cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2. The other zones are not impacted.

Description: We are experiencing an issue with Google Compute Engine beginning at Tuesday, 2022-07-19 08:10 US/Pacific.

Our engineering team continues to investigate the issue.

There has been a cooling related failure in one of our buildings that hosts zone europe-west2-a for region europe-west2. This caused a partial failure of capacity in that zone, leading to VM terminations and a loss of machines for a small set of our customers. We’re working hard to get the cooling back on-line and create capacity in that zone. We do not anticipate further impact in zone europe-west2-a and currently running VMs should not be impacted. A small percentage of replicated Persistent Disk devices are now running in single redundant mode.

In order to prevent damage to machines and an extended outage, we have powered down part of the zone and are limiting GCE preemptible launches. We are also working to restore redundancy for the impacted replicated Persistent Disk devices.

We will provide an update by Tuesday, 2022-07-19 11:45 US/Pacific with current details.

We apologize to all who are affected by the disruption.

Diagnosis: Customers impacted by this issue would have seen abnormal VM terminations for a small set of their VMs. We’re working on identifying and communicating with all affected customers.

Workaround: For customers who have experienced a VM failure in europe-west2-a, we recommend launching new instances in other zones of europe-west2.

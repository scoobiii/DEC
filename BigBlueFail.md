![](Aspose.Words.b2d177f1-63df-486e-aa01-604355f066f3.001.png)

**Customer Incident Report**

**Details![ref1]**

**Release Date:** 10 December 2020 **Outage Start: Reference number:** INC3218712 **Outage End: Revision: Outage Duration:**

**Environment:** Sao Paulo 01 DATACENTER **Recurring Event: Reported Service Impact:** Cooling Infrastructure Power Failure - SAO01 *(Note: All referenced times are in UTC)*

07 December 2020 07:23 08 December 2020 01:00 17 Hours 36 Minutes

No

**Incident Description![ref1]**

During this incident, IBM Cloud services were not available because of a cooling problem in a non-IBM owned building that houses the IBM Cloud data center in Sao Paulo, Brazil. Note: This is a multi-tenant building that included other Cloud service providers, and other non-IBM co-lo clients. All were impacted by this event. 

On 07 December 2020 at 07:13, a local IBM technician noticed an increase in temperature in SAO01. IBM Cloud DC Operations began to execute its incident response plan and actions. At 07:23, the Data Center Aggregate Routers (DARS) auto-shutdown due to high temperature. 

Investigation by the building owner discovered an issue with the main water supply that feeds the building's cooling towers. In an attempt to avert service impact, Data Center Specialists deployed fans. The building vendor then worked to move cooling to a secondary system, however it was also impacted by the water supply issue. 

It took several hours for the building owner to make repairs to the water supply valves, refill the cooling system, and reduce the interior temperature to a level that would allow IBM Cloud employees back inside the data center. Once back in the building, the data center infrastructure had to be validated, and brought back online, ensuring there was no loss of data, and that customer environments were fully protected.

**Root Cause Information![ref1]**

Root cause investigation determined that a failed building vendor maintenance for the data center cooling system resulted in the initial service impact. Additionally, alarms were not detected by the building vendor to mitigate this issue. 

During a building bi-monthly fan coil maintenance on 04 December 2020, two valves for fan coils were left open at the end of the Method Of Procedure (MOP), affecting water supply to the cooling towers. This allowed the dual water tanks to begin to drain. Following the maintenance, the building vendor did not verify that the valves were closed and did not detect the water tank level alarms that were triggered. These alarms indicated a continuous loss of water in the dual water tanks.  

Research into the alerting issue indicated that alarms remained open and were not properly programmed to send escalating messages to the building vendor operations team. The building vendor operations and maintenance teams reacted to the event when they were notified by a chiller pressure alarm.

**Timeline![ref1]**

This timeline only reflects the alerting, troubleshooting, and mitigation of the top-level service degradation or disruption. Additional dependent services or unique customer environments are not reflected here, and these might have experienced impact outside the times listed. 

The following sequence of events occurred: 

1. 07 December 2020 07:13 - IBM Cloud DC Specialists alerts the IBM Site Manager about temperature alarms. Coordinated response begins.
1. 07 December 2020 07:23 - Internal connectivity to the data center was lost. DARS auto shutdown. Incident started.
1. 07 December 2020 07:23 - IBM Site Manager notified Building Owner of rising temperature alarms. Building Management informed IBM that they were aware of the issue and were working on it.
1. 07 December 2020 07:30 – IBM Cloud DC Specialists deployed fans in an attempt to reduce the server room temperature.
1. 07 December 2020 09:04 – Building Vendor updated the Site Manager and moved cooling to a secondary system. A brief temperature drop was observed.
1. 07 December 2020 10:20 – Decision was made to start shutting down bare metal servers to reduce cooling load
1. 07 December 2020 10:30 – Water leak was repaired and water trucks dispatched to the site to replenish lost water.
1. 07 December 2020 11:20 - IBM Cloud personnel were vacated from the building because of elevated temperature.
1. 07 December 2020 11:53 – Building Owner requested authorization to power down Sao01, based on elevated temperatures and ETA of 2.5 hrs.
1. 07 December 2020 13:50 - IBM Data Center personnel were allowed to return to DC (visual inspection started).
1. 07 December 2020 15:30 - Network service team began recovery efforts.
1. 07 December 2020 15:46 - Network connectivity to DC reestablished.
1. 07 December 2020 18:00 - Initial Compute Services were restored.
1. 07 December 2020 20:30 - Storage internal management switches were online.
1. 07 December 2020 21:50 - Full network cleanup completed.
1. 07 December 2020 22:17 - All fabric services restored.
1. 08 December 2020 00:07 - Storage restoration completed.
1. 08 December 2020 00:15 - IKS services restored.
1. 08 December 2020 01:00 - ROKS restoration completed. All Compute services were restored. 
1. 08 December 2020 01:00 - Data center incident ended.

**Service Restoration![ref2]**

The Building owner hosting the IBM Cloud Data Center worked with the building vendor to close the valves and refill the cooling tanks. IBM Cloud Specialists then restored Network and Cloud services, mitigating the impact and ending the incident on 08 December 2020 at 01:00.

**Completed and Future Actions![ref2]**

The IBM Cloud team has analyzed this incident for areas of improvement, including issue detection, identification and future mitigation. The following specific improvements were identified:

**Description Due Date** Enhance customer communication for critical events 15 January 2021 Enhance checks and verification in the vendor MOP for data center maintenance 31 December 2020 Validate with the provider that appropriate thresholds and alerting is established and 31 December 2020

monitored

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

10 December 2020 PRB0055726

[ref1]: Aspose.Words.b2d177f1-63df-486e-aa01-604355f066f3.002.png
[ref2]: Aspose.Words.b2d177f1-63df-486e-aa01-604355f066f3.003.png

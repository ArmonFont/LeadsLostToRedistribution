# LeadsLostToRedistribution
KPI query used to see if AEs are attacking fresh web leads aggressively.

Fresh web leads are forms recently submitted by potential customers to Account Executives (AEs)

The goal of this query is to monitor our AE’s ability to address fresh web leads quickly and frequently. The CRM system used in this scenario is designed to reassign fresh web leads 4 hours after it is assigned to/dialed by the AE (within business hours). 

AEs are expected to maintain a low unassign rate. 


Leads contacted over 6 times are not counted as unassigned. Leads that are transferred to another department are not counted as unassigned. 

Credit pulls are also monitored. It was observed that AEs unassigned rate and credit pull rates were inversely related. 

The names of database tables/ other internal info were renamed.

At 07:00 UTC on October 17, 2026, ConnectWise disabled the seven application identities listed below and
revoked the associated credentials as part of our containment actions. These vendor-side actions do not change
the configuration of enterprise applications within individual partner or customer tenants.
Verify the local enterprise-application setting in every tenant you administer. Vendor-side containment and local
tenant settings are different controls; this table does not document a local change in every tenant. Previously
issued tokens may remain usable until they expire or are otherwise invalidated. Separate unauthorized
credentials, grants, or identities require investigation and remediation.

Application Application client ID
ConnectWise Cloud Backup 506e08f9-6e9a-4da4-942b-a099a03644ae
ConnectWise Cloud Management 2399d300-6fca-4e6a-a686-399fefe75b83
Cloud Management - MS Teams 24205314-2b89-4e07-a2a4-579e898a5640
Cloud Management - Exchange Online b78f6a77-d4e5-46fb-87e5-d1648bd02519
Cloud Management - MS Graph 0de14c77-b9b0-4137-9360-34878a929827
Cloud Backup technical investigation and remediation guide | 3
Application Application client ID
Cloud Management - Partner Center 412dcf96-176d-4b3a-a1b0-4981ad66971e
Cloud Management - SharePoint Online b2b55f5b-7758-4719-8d8f-88ae172e6bdb
Older installations may display SkyKick Cloud Manager names. Match on Application ID rather than display
name, and record the local Object ID as well. Do not assume a similarly named application has the same identity
or status.

https://aviatrix.ai/threat-research-center/hackers-compromised-connectwise-credential-173-microsoft-365-tenants-2026/
https://www.todyl.com/blog/investigation-unauthorized-access-through-the-connectwise-cloud-backup-microsoft-365-app
https://www.mitiga.io/blog/connectwise-breach-entra-app-abuse
https://specterops.io/blog/2026/10/17/connectwise-skykick-breach-microsoft-365/
https://www.reddit.com/r/msp/comments/1x7gw61/cwskykick_breach/


LM : The blast radius depends on how many tenants trusted the ConnectWise app, not on how much effort the threat actor spent in each one. A single stolen credential reaches every tenant where the app was consented.

Priv Escalation : App-only access isn’t tied to a single user, so the ConnectWise app can reach every mailbox, OneDrive, SharePoint site and Teams chat in the tenant. 

Exfil : Data leaves through the same Graph API path as legitimate backups. Microsoft Graph activity logs are the best evidence, if they were already enabled.

FP : You may also find other ConnectWise apps in the customer’s environment that the vendor has not flagged as compromised. Check with the customer whether they use them, and treat them as false positives for now.

Unknown : I found sign-in log entries from a private IP address, which I believe belongs to Microsoft’s internal infrastructure. I’m not sure why this is happening. Based on the public blog post, this appears to be legitimate traffic from the ConnectWise Cloud Backup app incase the customer is usin that app for their backup. However, the customer I’m investigating doesn’t use this service, yet these sign-ins still appear in their logs. It may be worth working with the Entra team to understand this better.



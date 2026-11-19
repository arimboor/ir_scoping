Recommendations – Teams Phishing Incident
1. Rebuild the affected device
Reimage the affected device from a known-good image rather than attempting to clean it. The threat actor had interactive remote access, so further changes beyond the identified persistence cannot be ruled out. Preserve the disk image and investigation package as evidence before rebuilding.
2. Restrict Microsoft Teams external access
The attack began with an external Teams contact impersonating IT support. Restrict external access to an allowlist of trusted domains, or block unknown external tenants. External access is enabled by default.
Reference: Manage external meetings and chat with people and organizations using Microsoft identities
3. Disable or restrict Quick Assist
Quick Assist provided the threat actor with remote access to the device. If it is not used for internal support, disable it by blocking traffic to the Quick Assist service endpoint, or remove the app across the estate using Intune.
Reference: Use Quick Assist to help users
4. Enable attack surface reduction (ASR) rules
ASR rules target behaviours seen in this incident, including scripts launching downloaded content, obfuscated scripts, and executables running from user folders. Deploy the relevant rules in block mode after an audit period.
Reference: Attack surface reduction rules reference
5. Implement application control
Prevent unapproved executables, installer packages and script hosts from running out of user-writable locations such as AppData and Downloads. This would have prevented the renamed Node.js binary and the VBScript from executing. Microsoft recommends App Control for Business over AppLocker where possible.
Reference: App Control and AppLocker overview
6. Block indicators of compromise
Block the identified command-and-control hostnames, IP address and blockchain RPC endpoints at the web proxy and as Microsoft Defender custom indicators. Add file hashes for the malicious installer, the renamed binary and the payload.
Reference: Create indicators in Microsoft Defender for Endpoint
7. Enable Defender network protection
Network protection enables blocking of connections to malicious domains and IP addresses at the network layer, from any process rather than only browsers. Combined with custom indicators, it would block command-and-control and blockchain lookup traffic.
Reference: Use network protection
8. Restrict outbound traffic and segment the network
Force outbound traffic through the proxy, and block direct connections to uncategorised cloud-hosted infrastructure and public blockchain RPC services unless there is a business need. Restrict workstation-to-workstation SMB, RDP and WinRM traffic to limit lateral movement following the domain and session discovery observed.
Reference: Enterprise access model
9. Reset credentials and revoke sessions
For all affected users, reset passwords, revoke sign-in sessions and refresh tokens, and review registered MFA methods. Credential exposure cannot be ruled out given the clipboard access and suspicious library load observed.
Reference: Revoke user access in Microsoft Entra ID
10. Use Defender device risk in Conditional Access
Link the Microsoft Defender device risk score to Intune compliance, so that devices exceeding the risk threshold are marked noncompliant and automatically lose access to company resources.
Reference: Configure Conditional Access with Microsoft Defender for Endpoint
11. Enable risk-based Conditional Access
Use sign-in risk and user risk policies to block or challenge suspicious sign-ins if stolen credentials are reused. These policies require Microsoft Entra ID Protection (Entra ID P2).
Reference: Microsoft Entra Conditional Access overview
12. Deploy Windows LAPS
Ensure each device has a unique, automatically rotated local administrator password, so that a single compromised device cannot be used to access others.
Reference: Windows LAPS overview
13. Apply privileged access tiering
Ensure administrative accounts never sign in to standard user workstations, so that a compromised workstation cannot expose privileged credentials.
Reference: AD DS tier model
14. Investigate the other affected devices
Hunt for the same artifacts on the devices of the other affected users, including the staging folder, the renamed Node.js binary, the Startup folder shortcut and connections to the identified C2 infrastructure. Rebuild any device confirmed as compromised.
15. User awareness
Remind staff that IT will never contact them through external Teams calls asking them to start remote access sessions. Provide a simple process to verify and report suspicious contacts.
16. Review network flow and web proxy logs
Based on the endpoint telemetry reviewed, no evidence of data exfiltration was identified. However, the device communicated with threat actor-controlled infrastructure, and endpoint telemetry alone cannot fully confirm the volume or content of data transferred. We recommend reviewing network flow and web proxy logs to gain further insight into any data transfer between the compromised device and this infrastructure.

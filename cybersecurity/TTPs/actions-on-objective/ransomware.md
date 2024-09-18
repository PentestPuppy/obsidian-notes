
# Ransomware
Ransomware is a type of malware which *encrypts files on a device*, rendering them completely useless to the targeted organization. Normally, threat actors use ransomware *to demand a ransom* from the org. If the ransom is paid, they promise to share the key, allowing the victim to decrypt their data.

Ransomware is a common attack because *it is extremely lucrative* for attackers and ransomware groups. Victims of ransomware normally have three options:
-  Restore from backups
- Accept the loss
- Pay the ransom
## Evolution
Ransomware used to be primarily done by a single individual, but now are usually done by *large ransomware groups* which practice "Ransomware as a Service". These groups are well-organized, and will even include services like customer support lines which victims and response teams can contact 24/7.
### Ransomware as a Service
- Customer service lines
- PR
- Branding
- Software as a Service (such as portals where victims can communicate or engage in payment handling)
### Rebranding
Having too much heat on you under one brand name is *bad for business*, so groups which have caught too much hear from the FBI, etc, will *rebrand* but continue working with the same [TTPs](../../defense/threat-intelligence/TTPs.md) and IOCs.
**Example**: rebranding of "Darkside" to "Black Matter"
### Methods:
#### Double Extortion
Instead of just encrypting data on a victim's network, ransomware groups will now *also exfiltrate the data* so they have additional leverage against the victim (who now fears the data will be released if they don't pay).
#### Triple Extortion
In addition to encryption and exfiltration, some ransomware groups will add additional incentives such as *DDoSing the victim* until they pay, *targeting partner organizations*, *auctioning off data* that has been exfiltrated, publicizing fragments of the data, etc..
#### Coercion 
Attacking the victim and their brand organization by attempting to make them look bad in order to coerce them into paying. Will also *threaten the victim's customers*  to try to get the customers to hound the organization to pay. For example 'tell XYZ Organization to pay the ransom or *YOUR CUSTOMER DATA* will be released on the dark web.'
## Mitigation
These mitigations come from [CISA](https://www.cisa.gov/stopransomware/ransomware-guide) (the Joint Cybersecurity and Infrastructure Security Agency).
### Backups
Protect critical data by maintaining an encrypted, offline backup of it. These backups should be *regularly tested* for availability and integrity. Backup procedures should also be tested regularly.

Backups should be kept offline because ransomware *commonly attempts to find these backups* and encrypt them or make them inaccessible.
#### Golden Images
Critical systems should be kept in 'golden images' which include image templates w/ pre-configured [operating systems](computers/concepts/operating-system.md) and associated software/ applications. These images should be updated regularly and should *be able to deploy quickly to rebuild a system*.
#### Hardware
If rebuilding a compromised system is not preferred, then hardware should be kept in the chance that it needs to be used for restoration. Hardware should be stored offline so it isn't infected by malicious actors. It should also be *as up to date as possible* so it can be deployed quickly.
### [Incident Response](../../defense/incident-response/incident-response.md) Plan (IRP)
Create and maintain an incident response plan which includes procedures for establishing communication and response. There should be a *hard copy* of the plan in addition to the on-device one. The IRP should be *exercised regularly* so employees know what to do during a real incident.

The IRP should include an *up to date diagram of the network/s*. including descriptions of systems, data flows, and internal and external dependencies.
### Vulnerabilities and Misconfigurations
Companies should try to limit their own vulnerability to a ransomware attack by:
- Not exposing unnecessary services to the web, such as [RDP](networking/protocols/RDP.md). Additionally, services which are exposed should be protected from common exploits and other forms of abuse
- Regular vulnerability scanning should be done to identify CVEs, etc.
- Software and operating systems should be kept up to date w/ latest patches and versions
- Devices on premise should be properly configured w/ secure features enabled
- Limit the use of remote desktop services in general
- MFA should be enabled wherever possible
- Identity and Access Management (IAM) systems should be used
- Access control should use a [zero trust](cybersecurity/defense/zero-trust.md) methodology\
- Networks should be *segmented physically and technologically*
- etc.
## Detection & Analysis
In the event of a ransomware attack, the victim org should follow their IRP. The IRP should include the following steps (among others):
### Isolate Impacted Systems
This may include taking the entire network offline, including physically unplugging devices and powering them down. Critical systems which are essential to regular operations should be the priority.
### Communication
Communication is vulnerable to being compromised by the threat actor. Because of that *out of band* communications should be used during an incident.
### Triage
The devices and systems which should be prioritized for protection and recovery need to be identified beforehand  in the IRP. Additionally, systems which aren't believed to be impacted *should be monitored*.
### Threat Hunting
Threat hunting should be initialized once stability from the attack has been established. This can include anything from looking through logs, examining detection systems, etc.. Forensics can highlight *additional compromised systems* as well as other malware involved in the attack.

Pay attention to other suspicious activities such as the creation of new accounts w/ escalated privileges, suspicious logins, endpoint modifications, etc..

There is also the possibility that *data was exfiltrated from the network* which can also be used to *for ransoming*. Combining a ransomware attack w/ the ransoming of sensitive data is called *'Double Extortion'*.

> [!Resources]
> - [CISA: Ransomware Guide](https://www.cisa.gov/stopransomware/ransomware-guide)
> - Internship training/ education


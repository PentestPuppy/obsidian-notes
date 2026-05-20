
# WHOIS Command
Query for who a [domain](/networking/DNS/DNS.md) name is registered to according to *Domain Registrars*.
## Usage
### Basic
```
whois <target domain name>
```
### Advanced
#### Query by Company Name
```bash
whois -h whois.ripe.net "Company Name"
```
**NOTE:** This is querying for [RIPE](../networking/protocols/whois.md#RIPE) RIRs (Europe, Middle East, Central Asia). For ARIN RIRs, it would be `-h whois.arin.net`
#### Query by Organization ID
```bash
whois -h whois.ripe.net -- '-i org ORG-XXXX-RIPE'
```
#### Query by IP Range
```bash
whois -h whois.ripe.net "192.0.2.0"
```
#### Search for all Related Objects
```bash
whois -h whois.ripe.net -- '-i org ORG-XXXX-RIPE' -T aut-num
```
- `-T aut-num`: specifically filters for ASN objects


> [!Resources]
> - `man whois`

> [!Related]
> - [WHOIS protocol](../networking/protocols/WHOIS.md)


# DNS Record Types: The Essential Cheat Sheet

Understanding DNS (Domain Name System) records is crucial for anyone managing domains, websites, or network services. This cheat sheet breaks down the most common DNS record types, their purpose, and practical use cases.

---

## 01. A Record (Address Record)
- **Purpose:** Maps a domain name to an IPv4 address.
- **Format:** `example.com. IN A 192.0.2.1`
- **Use Case:** Directing users to your web server or any service hosted on an IPv4 address.
- **CLI Example (dig):** `dig A example.com`

---

## 02. AAAA Record (IPv6 Address Record)
- **Purpose:** Maps a domain name to an IPv6 address.
- **Format:** `example.com. IN AAAA 2001:0db8::1`
- **Use Case:** Directing users to services hosted on an IPv6 address, supporting modern network infrastructure.
- **CLI Example (dig):** `dig AAAA example.com`

---

## 03. CNAME Record (Canonical Name Record)
- **Purpose:** Maps an alias domain name to another canonical (primary) domain name.
- **Format:** `www.example.com. IN CNAME example.com.`
- **Use Case:** Pointing subdomains (e.g., `www`) or services (e.g., `blog.example.com`) to your main domain without needing to update IP addresses directly.
- **CLI Example (dig):** `dig CNAME www.example.com`

---

## 04. MX Record (Mail Exchange Record)
- **Purpose:** Specifies the mail servers responsible for accepting email messages on behalf of a domain name. Includes a preference value.
- **Format:** `example.com. IN MX 10 mail.example.com.`
- **Use Case:** Ensuring emails sent to `@example.com` are routed to the correct mail server.
- **CLI Example (dig):** `dig MX example.com`

---

## 05. TXT Record (Text Record)
- **Purpose:** Stores arbitrary text information, often used for verification or policy.
- **Format:** `example.com. IN TXT "v=spf1 include:_spf.google.com ~all"`
- **Use Case:**
    - **SPF (Sender Policy Framework):** Preventing email spoofing.
    - **DKIM (DomainKeys Identified Mail):** Email authentication.
    - **Domain Verification:** Proving domain ownership to services like Google, Microsoft, etc.
- **CLI Example (dig):** `dig TXT example.com`

---

## 06. SRV Record (Service Record)
- **Purpose:** Specifies the location (hostname and port number) of servers for specific services.
- **Format:** `_sip._tcp.example.com. IN SRV 0 5 5060 sipserver.example.com.`
- **Use Case:** Locating services like SIP (Voice over IP), XMPP (Jabber), or other specialized network services.
- **CLI Example (dig):** `dig SRV _sip._tcp.example.com`

---

## 07. NS Record (Name Server Record)
- **Purpose:** Delegates a domain or subdomain to a specific set of authoritative DNS servers.
- **Format:** `example.com. IN NS ns1.nameserver.com.`
- **Use Case:** Defining which DNS servers are authoritative for your domain, allowing them to manage its records.
- **CLI Example (dig):** `dig NS example.com`

---

## 08. SOA Record (Start of Authority Record)
- **Purpose:** Contains administrative information about a zone, including the primary name server, the email of the domain administrator, the domain serial number, and various timers relating to refreshing the zone's information.
- **Format:** `example.com. IN SOA ns1.nameserver.com. hostmaster.example.com. (...)`
- **Use Case:** Essential for proper DNS zone management and replication. Every zone must have one.
- **CLI Example (dig):** `dig SOA example.com`

---

## 09. PTR Record (Pointer Record)
- **Purpose:** Performs reverse DNS lookups, mapping an IP address back to a domain name.
- **Format:** `1.2.0.192.in-addr.arpa. IN PTR example.com.`
- **Use Case:** Email servers often use PTR records to verify the sender's domain, helping to prevent spam. Also used in logging and network troubleshooting.
- **CLI Example (dig):** `dig -x 192.0.2.1`

---

## Practical Troubleshooting Commands:

- **`dig <domain> <record_type>`:** The most versatile tool for querying DNS.
    - `dig example.com A`
    - `dig example.com MX`
    - `dig -x 192.0.2.1` (for PTR records)
- **`nslookup <domain>`:** A simpler, interactive DNS query tool.
    - `nslookup example.com`
- **`host <domain>`:** A straightforward utility for basic lookups.
    - `host example.com`

---

**Save this essential guide for your next domain configuration or network deep dive.**
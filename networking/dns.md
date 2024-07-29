# DNS

## DNS Resolution

The Domain Name System (or DNS) translates human readable domain names (for example, www.amazon.com) to machine readable IP addresses (for example, 192.0.2.44).

First, a user tries to go to a website like www.amazon.com. This request is routed to a recursive resolver, also known as a "DNS recursor". Recursive resolvers cache information from authoritative nameservers, and may just return the IP address on the spot.

If a recursive resolver does not know which IP address corresponds to the domain, they will issue requests to (1) a root nameserver, (2) a TLD nameserver, and (3) an authoritative nameserver. The resulting information is then cached for future requests.

Recursive resolvers are generally provided by ISPs.

They exist to cache information for authoritative nameservers, who don't want to get flooded with requests.

DNS root nameservers route queries to TLD (or "top-level domain") nameservers, based on the top-level domain extension of the website address (i.e. '.com', '.net', etc.). IANA/ICAAN manages TLD nameservers.

## DNS Records

| Record   | Description |
| -------- | -------     |
| A         |  A *host* record. Maps a domain/host to an IP address.           |
| AAAA         | An *IPv6 host* record.            |
| PTR         | A *pointer* record. Opposite of a host record. Maps an IP address to a domain/host.            |
| TXT          | Leveraged mainly for SPF/DKIM/DMARC for email valdation, but this is a generic record.          |
| SRV | For VoIP, instant messaging, etc. Any kinds of newer protocols. |
| NS | This is a pointer to one or more authoritative DNS servers, which contain DNS records. | 
| MX | A record that directs email to a mail server domain. | 
| SPF | A "sender policy framework" record, which is a type of DNS TXT record that lists servers authorized to send emails from a particular domain. This is a component of DMARC. This is also (allegedly) deprecated in favor of TXT-based DNS records in place of SPF. |
| PTR | This is a record that is used to look up hosts by IP address; used to help populate trace routes and reverse DNS lookups. | 

## Sources

CompTIA Network+ Study Guide (4e)

https://aws.amazon.com/route53/what-is-dns

https://umbrella.cisco.com/blog/what-is-the-difference-between-authoritative-and-recursive-dns-nameservers

https://www.cloudflare.com/learning/dns/what-is-recursive-dns/
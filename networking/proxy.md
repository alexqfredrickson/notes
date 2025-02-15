# Proxies

A *forward proxy* sits between users and the internet, intercepting requests from users, and *forwarding* them forward to the internet. They can be used to obfuscate a user's online identity, or to bypass a firewall. A system administrator may force requests through a forward proxy to block access to content (i.e. to apply filtering rules).

A *transparent proxy* automatically intercepts all TCP traffic, at the layer 4 switch level, and forwards it through.

A *reverse proxy* sits between the internet and *web applications*, intercepting requests from the internet, and forwarding those to application servers. They can be used to obfuscate an application's identity - so that a website is harder to DDOS. A reverse proxy can also act as a load balancer. A reverse proxy may also cache content in a local data store to quickly serve data. A reverse proxy may also handle SSL handshakes.
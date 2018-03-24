LupinTech uses IPv4 internally and externally.

It uses the 192.168.AAA.BBB private networking space for its hosts, with the following reservations:

AAA under 100 is allocated to hosts in Central Services. Each of the (4 currently) CS zones has a /22 network in this space.
AAA 101-200 is allocated to POP hosts, each having a /26 network in this range
AAA 201-205 is allocated to production VPN and office access.

There is not yet any IPv6 usage. The platform engineering team is interested in experimenting with overlay networks but has not done so yet.

Presently, services find each other using a mix of traditional DNS and SVR records (configured by Chef) and an API-driven dynamic DNS-based service registry. There's an intent to slowly migrate most services to the latter; right now it's still pretty new (and not well-tested yet).

OpenVPN and OpenVPN-compatible devices are used to build network links between physically remote parts of the production network. Some former OpenVPN developers work at the company and made this choice because they have extensive knowledge of how to debug and scale it.

## Office networks and access to prod
LupinTech has 2 main offices, in Dublin and New York City. There are about 400 employees at the company; office networks are isolated from production and do not share the same network space; engineers need to jump through bastions to reach prod. There are central bastions as well as emergency access public hosts providing access to each network segment.


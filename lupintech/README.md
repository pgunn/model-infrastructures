# Lupin Technologies

Lupin Technologies is a low-profile privately-owned tech company that makes and runs infrastructure for a large number of mobile apps, many games, many crowdsourced utilities. Most of the company’s customers don’t know the name of the company (they avoid branding on the mobile apps) and just know the apps they use.

An example application: Trailblaze is a mobile app for people to collectively rate trails and propose hiking partners; like most LupinTech apps this works on an eventual-consistency “best effort” model.

LupinTech’s infrastructure is divided into two tiers:

- Central Services - Central data storage and accounting. Never directly communicates with the clients
- POPs - Small, local sets of servers that run the API endpoints for apps to talk to

API Servers in each POP categorise their data using a Usenet-like model, and use NNTP as a transport mechanism for dataflows. For example, the example application above might turn a trail review into a post on clientposts.trailblaze.geo.idaho (perhaps crossposting to another group if it might need to be searchable in another way). The POP’s NNTP servers would use regular NNTP synchronisation to send it to CentralServices and to all the other POPs, which in turn would lazily parse it (deep, geographically-aware paths and crossposting helps make for efficient searches). 

## POP (Point of Presence) Infrastructure
- Currently there are around 25 POPs. They’re all physical infrastructure in leased space. Eventually will extend into AWS and GCE.
- Each POP is 5-30 machines
- Software running on the bare OS. Chef, Debian. Currently no plans to change this.
- Each host is running a NNTP server
- Each has a varnish serving static assets on one port
- And nginx, to handle login and user-specific content on yet another port
- And 3 API endpoint listeners running on three more ports (XMLRPC, Json, Protobufs). Different apps use different API endpoints. XMLRPC is considered deprecated.
- Each host has an independent Postgres server that lazily scrapes NNTP in response to data requests and returns/caches that information. Apps are semi-aware of what machine in a POP they’re talking to and the infrastructure tries to keep a consistent map of client to server-in-a-pop to make the cache hit more
- Hardware load balancers/health checkers sit in front of the entire POP
- POP systems are independent from each other (although they NNTP peer). While a client will attempt to reuse a “routing key” they got when building a connection, if the backing host is down or the data is likely no longer in cache, the hardware load balancers will route requests to a new host and tell the client to update its routing key. Client code usually will also try not to use stale routing keys.
- Wiping a host, including loss of data it hasn’t streamed up, is explicitly considered “no big deal”.

## Central Services Infrastructure
- Currently 4 machine rooms, 2 in the US (lupin-us-pgh and lupin-us-sea), one in the UK (lupin-uk-mk), 1 in South Africa (lupin-sa-cape). Never colocated with a POP.
- Also 2 GCE regions, one in Singapore (gce-asia-southeast-1), one in Israel (gce-middleeast-central-1)
- This makes for 6 zones.
- About 90 hosts apiece, hardware mostly homogenous. There is a mandate to shuffle services around and refactor them to allow 1-2 CS zones to go down without user impact; right now it is only believed (but untested) that some services are ready for this.
- Most hosts run Debian with software running on the bare OS. Some minor services are transitioning into containers on independent Kubernetes clusters running in each CS zone.
- CS hosts do not “stand alone” the way POP hosts do
- Running a larger variety of infrastructure pieces, some central/administrative, some doing processing for the POPs
- Some components:
  - Nagios for monitoring (within a Central Services zone only)
  - Logstash - some logs from the nearest POP flow in and join the logs for this CS zone
  - User accounts - Kerberos
  - Central NNTP servers - peer with the NNTP servers running in the POPs. Faster storage for NNTP groups that are for data most relevant to the geo-zone they’re in (rough metaphor is NUMA).
  - Digester - These parse incoming NNTP traffic, do predefined operations on that data, and push it back out onto a different group-path for POPs to get. Most processing is about local data.
  - OpenVPN to POPs
  - Hardware-based VPN between each CS Zone

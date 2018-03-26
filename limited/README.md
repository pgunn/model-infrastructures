# Limited Ltd

Limited is a 2 year old startup that runs a website (and makes an app) that lets people upload data and work with skilled data analysts on interpreting their data. It charges for the transfer and storage of data as well as the time of its analysts. It does a lot of work with private companies and academia, but also some work with individuals; typically its analysts develop a long-term relationship with their clients (similar to how accountants in an accountantcy firm work with the company's clients). It employs about 40 analysts, 15 SREs, and 25 others.

Limited's infrastructure has a public-private cloud model; analysts are willing to work with clients on GCE or AWS if their data is already there (or if it seems a good fit to what is needed), but the company also has its own hardware to meet the following needs:
* Clients have regulatory or privacy concerns that rule out public clouds
* The analysis pipeline needs specialised hardware (e.g. HPC) that is either unavailable or too expensive in public clouds
* There's a long-running operation that guarantees a resource usage for a reasonable amount of time (1 week or more)

Longer-term the company wants to reduce its cloud footprint and increase its machine-room footprint when it gets better at managing co-tenancy and making resource usage predictable; it has the numbers and anticipates significant cost-savings.

# Infrastructure goals
The company makes money in hooking its analysts and their tools/code up to client data. It charges in analyst time and in computer resource usage (active and storage). When analysts are working with clients, they're often using Matlab, Mathematica, R, Jupyter, or similar tools together in a shared space with the data already loaded; they also usually have voice and possibly video chat to the analyst; this must be seamless.

Customers usually upload their data before starting a call. This needs to be easy and pretty reliable. Limited tries to be flexible in allowing all kinds of upload methods; for some methods (e.g. google storage buckets or aws s3 buckets) it produces custom upload clients that do retries and parallelism.

Data processing usually happens outside of a client call, and usually does not have a time constraint; jobs must not be lost but they can be delayed (or restarted from a checkpoint if stopped or crashed). 

Not all clients need processed data out of the platform, but for those that do it's generally acceptable for this to be somewhat less immediate/available than the upload and the live consulting.

# General infrastructure
Work done in Limited's infrastructure is (believed to be) highly decoupled; work done for one client doesn't generally have any relationship to work done for other clients. Accounting and access need to be coherent across all infrastructure though, and having data access/transfers between different parts of the network is important.

Limited has two main offices, one in its headquarters in Boston USA, and one in its secondary office in Erlangen Germany. It intends to open a third office in the Asia-Pacific region in about a year (specifics not yet determined). For proximity to its staff, it maintains its (presently 2) datacentres nearby. It calls these pDC (physical DataCenter) and calls its presence in GCE and AWS vDC (virtual DataCenter). It does not maintain virtual networks or similar between its pDCs or vDCs.

Limited's infrastructure can be divided into a few host types:
* Systems that are used for interactive sessions between clients and analysts. In pDCs this happens in a VM (a given host runs many VMs). In vDCs usually this happens in an instance that's spun up specifically for the interaction. These systems only perform the most trivial of computation (they're mainly specced for running Matlab or Mathematica or Jupyter); real work is farmed off to the batch system (below)
* Datastores - These are proximate to the interactive systems and hold user-provided datasets for analysis (both interactive and batch). They're usually running inside containers, and have network plumbing restricting their exposed port to access from the VMs allocated to a particular client. They run a great variety of storage technologies (hBase, Postgres, MongoDB, whatever clients want). Some datastores are outside this (some are in S3 or GS, some are large filesystems, and occasionally clients continue to host their data and set up access across the networks)
* Batch System - This is a large and varied set of hosts that do work on data in the datastores. They're categorised as a grid across two dimensions: short versus long, and a variety of specific hardware profiles. The short queue runs shorter tasks in times reasonable for client-analyst interactions. The long queue runs longer tasks whenever resources are available. The batch system is designed to flexibly allow VM or container-based workflows (VMs are used to run user-provided binaries).

There's a much smaller number of additional internal services (including an internal knowledgebase based on MediaWiki and an internal Jira for tracking work). The main website for Limited is hosted on Cloudfront/S3.


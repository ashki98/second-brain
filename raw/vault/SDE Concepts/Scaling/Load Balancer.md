# Load Balancer

summary from yt video: 

**What is a Load Balancer? by IBM Technologies:** [https://www.youtube.com/watch?v=sCR3SAVdyCc](https://www.youtube.com/watch?v=sCR3SAVdyCc)

- **Need for Load Balancing:** When a website experiences high traffic, a single server isn't enough. Scaling out with multiple servers becomes necessary [[01:17](http://www.youtube.com/watch?v=sCR3SAVdyCc&t=77)]. A load balancer is crucial for distributing traffic among these servers [[02:12](http://www.youtube.com/watch?v=sCR3SAVdyCc&t=132)].
- **Load Balancer Definition:** It's a device (hardware or software) positioned between the internet and your application servers [[02:20](http://www.youtube.com/watch?v=sCR3SAVdyCc&t=140)]. It intercepts incoming traffic and directs it to an appropriate server [[02:26](http://www.youtube.com/watch?v=sCR3SAVdyCc&t=146)].
- **Load Balancer Functions:**
    - Distributes incoming traffic to available servers [[02:32](http://www.youtube.com/watch?v=sCR3SAVdyCc&t=152)].
    - Gathers information from servers, like their current load/utilization [[02:46](http://www.youtube.com/watch?v=sCR3SAVdyCc&t=166)].
    - Can initiate auto-scaling (adding/removing servers) based on the load [[02:51](http://www.youtube.com/watch?v=sCR3SAVdyCc&t=171)].
- **Role in Cloud-Native Architectures:** Load balancers are vital for ensuring traffic is efficiently distributed to application servers, which typically access a shared database [[03:22](http://www.youtube.com/watch?v=sCR3SAVdyCc&t=202)].
- **Common Load Balancing Methods:**
    - **Round Robin:** Sends traffic to servers in sequential order [[04:32](http://www.youtube.com/watch?v=sCR3SAVdyCc&t=272)].
    - **Smart Load Balancing:** The load balancer communicates with servers to identify the one with the least load [[05:22](http://www.youtube.com/watch?v=sCR3SAVdyCc&t=322)].
    - **Random Select:** Uses a random function to choose the server for incoming traffic [[07:12](http://www.youtube.com/watch?v=sCR3SAVdyCc&t=432)].

## Why not expose the application server directly?

- **Single point of failure** — one server, one outage. No failover if it crashes or needs a restart.
- **No health-aware routing** — the LB continuously health-checks servers and pulls unhealthy ones out of rotation. A directly exposed server has nothing doing that; it just goes down and stays down until someone notices.
- **Can't scale horizontally without breaking clients** — adding a second server means clients need a way to be spread across both. Without an LB you're stuck with DNS round-robin, which is slow to propagate, gets cached by resolvers, and has zero awareness of health or load.
- **No zero-downtime deploys** — an LB lets you drain connections from a server, take it out of rotation, deploy, and add it back (rolling deploys). No rotation to drain from means deploys drop connections.
- **Bigger attack surface** — the server's IP/port becomes directly internet-reachable, with no edge layer to absorb SYN floods, DDoS traffic, slowloris-style slow connections, or apply WAF/rate-limiting before it reaches app code.
- **Resource exhaustion** — every raw client connection (retries, bots, slow clients included) competes directly for the app server's thread/worker pool and OS connection limits, with no buffering layer in front.
- **Internal topology exposed** — an LB gives clients one stable public endpoint; behind it, servers can be resized, replaced, or re-IP'd freely. Expose the server directly and its identity *is* the public contract.

The LB is the resilience + security boundary between unpredictable internet traffic and actual app logic — skipping it couples every client directly to one fragile, exposed machine.

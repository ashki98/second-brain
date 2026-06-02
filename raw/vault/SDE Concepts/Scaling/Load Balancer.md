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

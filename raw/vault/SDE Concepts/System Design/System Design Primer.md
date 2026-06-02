# System Design Primer

[https://systemdesignschool.io/primer#component-breakdown](https://systemdesignschool.io/primer#component-breakdown)

Key concepts discussed in this chat:

- **RPS for app servers (ECS services)**
    - RPS (requests per second) is the main throughput metric for stateless services, analogous to how much “front-end” traffic a service can handle.geeksforgeeks+1
    - Actual RPS per ECS task/container depends on CPU, memory, code efficiency, and downstream dependencies; you scale horizontally with more tasks behind a load balancer to hit target total RPS.aws.amazon+1
- **Database metrics: QPS/TPS vs connections**
    - For databases, the “RPS equivalent” is **QPS/TPS** (queries per second / transactions per second) plus **latency**; these tell you real throughput.liquibase+2
    - **Connections** (active sessions vs `max_connections`) are capacity/pressure signals, not throughput by themselves; you care about QPS and latency *at* a certain concurrency.manageengine+2
    - Storage metrics like **IOPS, bandwidth throughput, and disk latency** often become the real bottlenecks under high QPS.simplyblock+3
- **Reasoning about QPS for a system**
    - Do a back-of-the-envelope: estimate daily operations from DAU × actions/user, then QPS≈ops per day/86,400\text{QPS} \approx \text{ops per day} / 86{,}400QPS≈ops per day/86,400, split into read/write QPS.dev+3
    - Measure real QPS using monitoring on APIs and DBs and compare to the capacity of a single node (from load tests or vendor guidance) to decide how many instances you need.grokipedia+2
- **When to introduce followers / read replicas**
    - Use **leader–follower (single-leader) replication** when you are read-heavy and a single primary cannot handle read QPS with good latency, but write load is still within a single-node limit.educative+3
    - Followers (read replicas) serve most reads; the leader handles all writes and maybe “must-be-fresh” reads. This trades strict consistency for scalability and availability (eventual consistency due to replication lag).cloud.ibm+3
    - Read replicas also help with high availability and DR, since a follower can be promoted if the leader fails.aws.amazon+3
- **How leader–follower replication works (DB-level)**
    - The **leader/primary** writes to an internal log (WAL/binlog). Followers stream that log and apply changes in order (PostgreSQL streaming replication, MySQL replication, RDS/Aurora read replicas).dev+3
    - Usually set up as **asynchronous** replication; sometimes semi-sync for tighter durability guarantees. Async gives better latency but allows replicas to lag.enjoyalgorithms+2
    - Followers are typically read-only; attempts to write to them either fail or are blocked by configuration.notes.shichao+1
- **How apps use leader + followers (read/write splitting)**
    - Logical contract: **all writes → leader**, **most reads → followers**, except flows that require read-your-writes or strict consistency.leapcell+2
    - Common app pattern: two connection pools/DSNs, `DB_WRITE_URL` and `DB_READ_URL`; application code or an ORM/router decides which to use based on operation type.stackademic+2
    - For requests that must see their own writes (e.g., after creating an order), you route that specific read to the leader or use a strategy like “read from leader for N seconds after a write.”openmetal+2
- **Where routing logic can live (app vs infra)**
    - **Replication tools** (Postgres streaming replication, MySQL replication, RDS/Aurora replicas) handle copying data; they *do not* automatically know which queries are reads vs writes.aws.amazon+3
    - **Routing / proxy tools** centralize read/write splitting, so app logic can be minimal but usually not zero:
        - **Aurora / managed DBs**: expose a “writer endpoint” and a “reader endpoint”; you choose which endpoint to use, and AWS balances across replicas behind the reader endpoint.aws.amazon+3
        - **MySQL Router**: offers ports for read–write vs read-only traffic; it does read/write splitting and replica selection behind those ports.mysql+3
        - **PgBouncer** (and variants like pgbouncer-rr): can be configured to route to primary vs replicas via logical databases or extended routing rules; you still choose which DSN to hit from the app.enterprisedb+3
    - “Almost no app logic” setup is: app knows only “writer DSN” vs “reader DSN”; the proxy/managed service handles replica membership, failover, and balancing.hevodata+3
- **Operational considerations for follower setups**
    - **Provisioning**: replicas usually start from a snapshot/backup, then catch up via logs (WAL/binlog + LSN/position).dev+3
    - **Failover**: an orchestrator (cloud control plane, cluster manager, or custom HA tool) promotes a follower to leader and updates endpoints/proxies; clients reconnect via stable DNS or proxy endpoints.designgurus+3
    - **Consistency and UX**: because of replication lag, the system is typically *eventually consistent*; design flows so brief staleness is acceptable for most reads, while critical paths use stronger guarantees.leapcell+2

No explicit visual diagrams were exchanged in this chat, but conceptually the main mental models to remember are:

- **Service throughput**: many stateless app instances (e.g., ECS tasks) behind a load balancer handling aggregate RPS.nathanpeck+1
- **Leader–follower DB**: one leader node receiving all writes, multiple follower nodes replicating logs from the leader and serving most reads, with read/write routing done either in the application or via a proxy layer.aws.amazon+3
1. [https://www.geeksforgeeks.org/system-design/capacity-estimation-in-systems-design/](https://www.geeksforgeeks.org/system-design/capacity-estimation-in-systems-design/)
2. [https://systemdesignschool.io/fundamentals/qps](https://systemdesignschool.io/fundamentals/qps)
3. [https://docs.aws.amazon.com/AmazonECS/latest/developerguide/capacity-autoscaling-best-practice.html](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/capacity-autoscaling-best-practice.html)
4. [https://nathanpeck.com/amazon-ecs-scaling-best-practices/](https://nathanpeck.com/amazon-ecs-scaling-best-practices/)
5. [https://www.liquibase.com/blog/database-performance-monitoring-going-beyond-essential-metrics-with-observability](https://www.liquibase.com/blog/database-performance-monitoring-going-beyond-essential-metrics-with-observability)
6. [https://www.manageengine.com/products/applications_manager/tech-topics/key-database-metrics-to-monitor.html](https://www.manageengine.com/products/applications_manager/tech-topics/key-database-metrics-to-monitor.html)
7. [https://grokipedia.com/page/Queries_per_second](https://grokipedia.com/page/Queries_per_second)
8. [https://last9.io/blog/database-monitoring-metrics/](https://last9.io/blog/database-monitoring-metrics/)
9. [https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-metrics.html](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-metrics.html)
10. [https://www.simplyblock.io/blog/database-performance-storage-limitations/](https://www.simplyblock.io/blog/database-performance-storage-limitations/)
11. [https://www.aptible.com/docs/core-concepts/managed-databases/managing-databases/database-tuning](https://www.aptible.com/docs/core-concepts/managed-databases/managing-databases/database-tuning)
12. [https://www.komprise.com/glossary_terms/iops/](https://www.komprise.com/glossary_terms/iops/)
13. [https://www.techtarget.com/searchstorage/definition/IOPS-input-output-operations-per-second](https://www.techtarget.com/searchstorage/definition/IOPS-input-output-operations-per-second)
14. [https://dev.to/zeeshanali0704/how-i-calculate-capacity-for-systems-design-399o](https://dev.to/zeeshanali0704/how-i-calculate-capacity-for-systems-design-399o)
15. [https://www.educative.io/answers/how-to-estimate-requests-per-second-of-a-server](https://www.educative.io/answers/how-to-estimate-requests-per-second-of-a-server)
16. [https://bytebytego.com/courses/system-design-interview/back-of-the-envelope-estimation](https://bytebytego.com/courses/system-design-interview/back-of-the-envelope-estimation)
17. [https://docs.oracle.com/en-us/iaas/application-integration/doc/calculate-requests-second.html](https://docs.oracle.com/en-us/iaas/application-integration/doc/calculate-requests-second.html)
18. [https://stackoverflow.com/questions/67798031/how-to-calculate-how-many-servers-are-needed-for-an-specific-qps-queries-per-se](https://stackoverflow.com/questions/67798031/how-to-calculate-how-many-servers-are-needed-for-an-specific-qps-queries-per-se)
19. [https://www.educative.io/answers/leader-and-follower-replication](https://www.educative.io/answers/leader-and-follower-replication)
20. [https://www.geeksforgeeks.org/system-design/leader-follower-pattern-in-distributed-systems/](https://www.geeksforgeeks.org/system-design/leader-follower-pattern-in-distributed-systems/)
21. [https://dev.to/the_infinity/database-replication-encyclopaedia-single-leader-replication-13-2l5c](https://dev.to/the_infinity/database-replication-encyclopaedia-single-leader-replication-13-2l5c)
22. [https://leapcell.io/blog/scaling-reads-and-writes-with-database-replication](https://leapcell.io/blog/scaling-reads-and-writes-with-database-replication)
23. [https://cloud.ibm.com/docs/databases-for-mysql?topic=databases-for-mysql-read-replicas](https://cloud.ibm.com/docs/databases-for-mysql?topic=databases-for-mysql-read-replicas)
24. [https://aws.amazon.com/rds/features/read-replicas/](https://aws.amazon.com/rds/features/read-replicas/)
25. [https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-read-replicas](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-read-replicas)
26. [https://docs.oracle.com/iaas/mysql-database/doc/overview-read-replica.html](https://docs.oracle.com/iaas/mysql-database/doc/overview-read-replica.html)
27. [https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.html](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.html)
28. [https://www.enjoyalgorithms.com/blog/master-slave-replication-databases/](https://www.enjoyalgorithms.com/blog/master-slave-replication-databases/)
29. [https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Replication.html](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Replication.html)
30. [https://www.designgurus.io/blog/database-replication](https://www.designgurus.io/blog/database-replication)
31. [https://notes.shichao.io/dda/ch5/](https://notes.shichao.io/dda/ch5/)
32. [https://blog.stackademic.com/routing-queries-to-read-or-write-replica-in-spring-boot-simplified-c055e085e4cb](https://blog.stackademic.com/routing-queries-to-read-or-write-replica-in-spring-boot-simplified-c055e085e4cb)
33. [https://www.enterprisedb.com/blog/taking-advantage-write-only-and-read-only-connections-pgbouncer-django](https://www.enterprisedb.com/blog/taking-advantage-write-only-and-read-only-connections-pgbouncer-django)
34. [https://openmetal.io/resources/blog/leader-based-vs-leaderless-replication/](https://openmetal.io/resources/blog/leader-based-vs-leaderless-replication/)
35. [https://hevodata.com/learn/amazon-rds-read-replicas/](https://hevodata.com/learn/amazon-rds-read-replicas/)
36. [https://stackoverflow.com/questions/63880798/is-it-necessary-to-manage-read-replica-details-on-application-side-for-auroradb](https://stackoverflow.com/questions/63880798/is-it-necessary-to-manage-read-replica-details-on-application-side-for-auroradb)
37. [https://www.projectpro.io/article/amazon-aurora/853](https://www.projectpro.io/article/amazon-aurora/853)
38. [https://dev.mysql.com/doc/mysql-router/8.2/en/router-read-replicas.html](https://dev.mysql.com/doc/mysql-router/8.2/en/router-read-replicas.html)
39. [https://blogs.oracle.com/mysql/mysql-routing-guidelines-introduction](https://blogs.oracle.com/mysql/mysql-routing-guidelines-introduction)
40. [https://dev.mysql.com/doc/mysql-router/9.0/en/router-read-write-splitting.html](https://dev.mysql.com/doc/mysql-router/9.0/en/router-read-write-splitting.html)
41. [https://www.mydbops.com/blog/mysql-transparent-read-write-splitting-with-mysql-router-8-2](https://www.mydbops.com/blog/mysql-transparent-read-write-splitting-with-mysql-router-8-2)
42. [https://www.pgbouncer.org/config.html](https://www.pgbouncer.org/config.html)
43. [https://planetscale.com/docs/postgres/connecting/pgbouncer](https://planetscale.com/docs/postgres/connecting/pgbouncer)
44. [https://www.pgbouncer.org/faq.html](https://www.pgbouncer.org/faq.html)
45. [https://systemdesignschool.io/primer#component-breakdown](https://systemdesignschool.io/primer#component-breakdown)

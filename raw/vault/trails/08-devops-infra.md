# Trail: DevOps & Infrastructure

**Objective:** Understand how modern software gets built, packaged, and deployed automatically and reliably.
**Prerequisites:** Trail 04 — Backend Patterns (for what you're shipping)
**Review time:** ~20 min
**Nodes:** 4

---

1. [[Docker Container vs Registry]]
   > Start with the unit of deployment: a container. This note covers what Docker containers are (isolated processes with their own filesystem), how they differ from VMs, and what a container registry is (where images are stored and pulled from). After this note, "ship it in a container" is a concrete action, not a buzzword.

2. [[Jenkins & CI CD Walkthrough]]
   > Automate the build-test-deploy cycle. This note walks through a Jenkins pipeline: triggers, stages (build, test, deploy), agents, and how the pipeline integrates with your version control. CI/CD is what turns "it works on my machine" into "it ships reliably every time."

3. [[Github Gitlab CI CD]]
   > The git-native version: CI/CD defined in a YAML file alongside your code. This note covers GitHub Actions and GitLab CI — the workflows your team most likely uses day-to-day. After the Jenkins conceptual foundation, this is the practical tool.

4. [[Kubernetes]]
   > Orchestrate containers at scale: scheduling pods across nodes, health checks, rolling deployments, and service discovery. Kubernetes is the platform that containers run on in production. This note covers the core objects (Pod, Deployment, Service, Ingress) and connects back to the load balancing concepts from Trail 06.

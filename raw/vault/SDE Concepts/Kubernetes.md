# Kubernetes

https://notebooklm.google.com/notebook/4203adc6-be82-48eb-9f2a-419879d89c68

![[NotebookLM_Mind_Map_(1).png]]

This summary serves as a comprehensive "cheat sheet" for the **Kubernetes Crash Course** by TechWorld with Nana. It covers the core architecture, essential components, and the hands-on workflow.

### **1. Core Concepts & "Why Kubernetes?"**

- **Definition:** An open-source container orchestration framework (originally by Google) used to manage hundreds or thousands of containers across different environments [[01:45](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=105)].
- **Key Problems Solved:**
    - **High Availability:** Ensures no downtime for users [[03:32](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=212)].
    - **Scalability:** Easily scales applications up or down based on load [[03:41](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=221)].
    - **Disaster Recovery:** Mechanisms to back up and restore data to the latest state [[04:03](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=243)].

### **2. Kubernetes Architecture (The Cluster)**

A cluster consists of at least one **Master Node** and multiple **Worker Nodes** [[04:40](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=280)].

- **Worker Node:** Where applications run. Contains:
    - **Kubelet:** A process that allows the node to communicate with the cluster and execute tasks [[04:49](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=289)].
- **Master Node (Control Plane):** Manages the cluster state. Key processes:
    - **API Server:** The entry point for all clients (UI, CLI, API) [[05:41](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=341)].
    - **Controller Manager:** Monitors the cluster and handles repairs (e.g., restarting dead containers) [[06:12](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=372)].
    - **Scheduler:** Decides which worker node should run a new container based on resources [[06:27](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=387)].
    - **etcd:** A key-value store that acts as the "cluster brain," holding the current state/configuration [[06:54](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=414)].
- **Virtual Network:** Spans all nodes, allowing them to communicate and act as one powerful machine [[07:36](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=456)].

### **3. Main Kubernetes Components**

- **Pod:** The smallest unit. An abstraction over a container. Usually runs one application container [[09:48](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=588)].
- **Service:** A permanent IP address for pods. It acts as a **Load Balancer** and ensures communication persists even if a pod dies and gets a new internal IP [[12:21](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=741)].
    - **Internal Service:** For database/back-end pods [[13:15](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=795)].
    - **External Service:** Opens communication to the public (e.g., via NodePort) [[13:08](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=788)].
- **Ingress:** Routes traffic into the cluster using secure protocols and domain names instead of just IP/ports [[14:03](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=843)].
- **ConfigMap & Secret:**
    - **ConfigMap:** External configuration (URLs, endpoints) so you don't have to rebuild images for small changes [[15:37](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=937)].
    - **Secret:** Similar to ConfigMap but for sensitive data (passwords, certs), stored in Base64 [[16:39](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=999)].
- **Volumes:** Attaches physical storage to pods to ensure data persistence if a container restarts [[18:28](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=1108)].
- **Deployments vs. StatefulSets:**
    - **Deployment:** A blueprint for **stateless** apps. Manages replicas and scaling [[21:25](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=1285)].
    - **StatefulSet:** Used for **stateful** apps like databases (MySQL, MongoDB) to manage data consistency and synchronization [[23:09](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=1389)].

### **4. Configuration & Deployment Workflow**

- **YAML Files:** Configuration is **declarative**—you define the "desired state," and Kubernetes works to match the "actual state" (self-healing) [[28:01](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=1681)].
- **Selectors & Labels:** Labels are key-value pairs used to identify and group components. **Selectors** are used by Services and Deployments to find the specific pods they manage [[49:09](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=2949)].
- **MiniKube & Kubectl:**
    - **MiniKube:** A tool to run a one-node K8s cluster locally for testing [[33:46](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=2026)].
    - **Kubectl:** The Command Line Interface (CLI) to interact with any K8s cluster [[34:25](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=2065)].

### **5. Useful Kubectl Commands (Quick Reference)**

- `kubectl apply -f [filename].yaml`: Create or update components [[01:04:55](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=3895)].
- `kubectl get all`: List pods, services, and deployments [[01:05:48](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=3948)].
- `kubectl describe [type] [name]`: Get detailed info about a specific component [[01:07:43](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=4063)].
- `kubectl logs [pod_name]`: View container logs for debugging [[01:08:36](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=4116)].
- `minikube ip`: Get the IP of your local cluster to access external services [[01:09:35](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=4175)].

---

**Conversation Highlights & Clarifications:**

- **Stateful vs. Stateless:** You might have wondered why we don't just use Deployments for everything. The video emphasizes that **StatefulSets** are necessary for databases because they ensure that even if a pod is moved or restarted, it maintains its specific identity and connects to the correct storage [[23:09](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=1389)].
- **The Three Parts of YAML:** Every K8s config has **Metadata**, **Specification** (your desired state), and **Status** (automatically generated by K8s to track actual state) [[28:42](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=1722)].
- **External Access:** To see your app in a browser, you must use a **NodePort** (range 30000–32767) or **Ingress** [[01:02:41](http://www.youtube.com/watch?v=s_o8dwzRlu4&t=3761)].

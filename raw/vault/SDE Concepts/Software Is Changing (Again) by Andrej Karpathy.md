# "Software Is Changing (Again)" by Andrej Karpathy

[https://www.youtube.com/watch?v=LCEmiRjPEtQ](https://www.youtube.com/watch?v=LCEmiRjPEtQ)

![[image 7.png]]

Here's a summary of the video "Software Is Changing (Again)" by Andrej Karpathy, covering the key concepts in bullet points:

- **Evolution of Software**
    - **Software 1.0:** Traditional, human-written code (e.g., C++ in Tesla's autopilot) [[01:42:01](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=6121)], [[04:49:10](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=17350)].
    - **Software 2.0:** Neural networks with "code" as weights, tuned by data and optimizers (e.g., AlexNet) [[01:47:04](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=6424)], [[02:46:27](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=9987)]. Software 2.0 replaced much of Software 1.0 in Tesla's autopilot [[05:26:08](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=19568)]. Hugging Face is the "GitHub of Software 2.0" [[02:16:03](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=8163)].
    - **Software 3.0:** Large Language Models (LLMs) programmed via natural language prompts [[03:19:15](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=11955)], viewed as a new kind of computer [[03:06:23](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=11183)].
- **LLMs as New Utilities, Fabs, and Operating Systems**
    - **Utilities:** LLMs require significant investment (capex) and metered API access (opex) [[06:44:17](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=24257)], demanding low latency and high uptime [[07:02:45](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=25365)].
    - **Fabs:** LLMs resemble fabrication plants due to high capital costs and rapid technological advancements [[08:04:48](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=29088)].
    - **Operating Systems:** LLMs are complex software ecosystems, similar to operating systems [[09:12:49](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=33169)], with closed-source (like Windows/Mac OS) and open-source (like Linux/Llama) parallels [[09:32:00](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=34320)]. The LLM is the "CPU," context windows are "memory," orchestrating memory and compute [[10:14:14](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=36854)].
    - **Early Computing Era:** LLM compute is compared to the 1960s, expensive and centralized [[11:02:18](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=39738)], with personal computing revolution yet to fully materialize [[11:39:56](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=41996)].
    - **Technology Diffusion:** LLMs uniquely started with consumer adoption [[12:57:38](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=46658)].
- **Psychology of LLMs**
    - LLMs are "stochastic simulations of people" with emergent human-like psychology [[14:49:28](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=53368)].
    - **Superpowers:** Encyclopedic knowledge and memory [[15:30:11](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=55811)].
    - **Cognitive Deficits:**
        - **Hallucination:** LLMs make up information [[16:07:07](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=58027)].
        - **Jagged Intelligence:** Superhuman in some areas, basic mistakes in others [[16:20:25](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=58825)].
        - **Anterograde Amnesia:** LLMs don't natively consolidate knowledge [[16:43:18](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=60198)].
        - **Gullibility/Security Risks:** Susceptible to prompt injection and data leaks [[17:44:03](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=63843)].
- **Opportunities and Building LLM Apps**
    - **Partial Autonomy Apps:** Integrate LLMs (e.g., Cursor, Perplexity) [[18:25:27](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=66327)].
    - **Key Properties of LLM Apps:**
        - Context Management [[19:24:43](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=69883)].
        - Orchestration of Multiple LLMs [[19:32:00](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=70320)].
        - Application-Specific GUI for auditing [[19:44:56](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=71096)].
        - Autonomy Slider to control AI level [[20:26:02](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=73562)].
    - **Human-AI Cooperation:** Fast generation and verification loop [[22:13:21](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=80001)], with fast verification (GUIs) and controlled AI output [[22:24:29](http://www.youtube.com/watch?v=LCEmiRjPEtQ&t=80669)].
    - **Best Practices:** Be concrete in prompts [24:30:42].
    - **Education with AI:** Separate apps for teachers/students with auditable "course" artifact [25:05:08].
    - **Analogy to Autopilot:** Partial autonomy with GUI and autonomy slider [25:52:16]. "Decade of agents," not just a year [27:36:13].
    - **Iron Man Suit Analogy:** Focus on augmentation/partial autonomy ("Iron Man suits") rather than full autonomy ("Iron Man robots") [27:54:27].
- **Everyone is a Programmer (Vibe Coding)**
    - Natural language interface makes everyone a programmer [29:11:13].
    - **Vibe Coding:** Building apps by "winging it" with LLMs [31:07:05].
    - **Real-World Challenges:** Making applications "real" (authentication, payments) remains difficult [32:30:17].
- **Building for Agents**
    - Software infrastructure needs to adapt to agents [33:40:24].
    - `lm.txt` for LLM website behavior [34:12:35].
    - LLM-friendly documentation (markdown, API commands) [34:37:37].
    - Ingestion Tools (e.g., get-ingest, DeepWiki) [36:33:04].
    - Make information easily accessible to LLMs [37:32:00].

Karpathy emphasizes that it's an exciting time to be in the software industry due to the transformative potential of AI and LLMs

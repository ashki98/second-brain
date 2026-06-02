# LLMs- Fine Tuning, RAG

## YT Video IBM **RAG vs. Fine Tuning** [https://www.youtube.com/watch?v=00Q0G84kq3M](https://www.youtube.com/watch?v=00Q0G84kq3M)

The video explains the differences between Retrieval Augmented Generation (RAG) and fine-tuning, two methods used to enhance large language models (LLMs) [[00:00](https://www.youtube.com/watch?v=00Q0G84kq3M&t=0)].

Here's a breakdown:

- **Retrieval Augmented Generation (RAG):**
    - RAG enhances a model's capabilities by retrieving external, up-to-date information and using it to augment the original prompt [[01:07](https://www.youtube.com/watch?v=00Q0G84kq3M&t=67)].
    - It uses a "corpus" of information (like documents and spreadsheets) to provide context to the model [[01:38](https://www.youtube.com/watch?v=00Q0G84kq3M&t=98)].
    - RAG is useful for dynamic data sources and providing sources for information, increasing trust and transparency [[03:44](https://www.youtube.com/watch?v=00Q0G84kq3M&t=224)].
- **Fine-tuning:**
    - Fine-tuning specializes a foundational model in a specific domain using labeled, targeted data [[02:33](https://www.youtube.com/watch?v=00Q0G84kq3M&t=153)].
    - It allows the model to adopt a specific tone or style [[02:53](https://www.youtube.com/watch?v=00Q0G84kq3M&t=173)].
    - Fine-tuning "bakes in" context and intuition into the model's weights, improving speed and inference cost [[03:12](https://www.youtube.com/watch?v=00Q0G84kq3M&t=192)].
- **RAG vs. Fine-tuning:**
    - RAG is ideal for dynamic data sources, while fine-tuning is better for industries with specific language nuances [[05:59](https://www.youtube.com/watch?v=00Q0G84kq3M&t=359)].
    - RAG provides source transparency, while fine-tuning leverages past data for better outputs [[06:46](https://www.youtube.com/watch?v=00Q0G84kq3M&t=406)].
- **Combining RAG and Fine-tuning:**
    - The best approach often involves combining both methods [[07:34](https://www.youtube.com/watch?v=00Q0G84kq3M&t=454)].
    - For example, a financial news reporting service could be fine-tuned for financial terminology and use RAG to provide up-to-date news sources [[07:40](https://www.youtube.com/watch?v=00Q0G84kq3M&t=460)].

## YT Video: **RAG vs. CAG by IBM Technologies**

[https://www.youtube.com/watch?v=HdafI0t3sEY](https://www.youtube.com/watch?v=HdafI0t3sEY)

![[Screenshot_2025-04-02_at_15.53.19.png]]

The YouTube video discusses two main techniques for enhancing the knowledge of Large Language Models (LLMs): Retrieval Augmented Generation (RAG) and Cache Augmented Generation (CAG).

**Retrieval Augmented Generation (RAG)** [[00:34](https://www.youtube.com/watch?v=HdafI0t3sEY&t=34)]

- RAG is a two-phase process involving an offline phase for knowledge indexing [[02:30](https://www.youtube.com/watch?v=HdafI0t3sEY&t=150)] and an online phase for retrieving and generating answers [[02:36](https://www.youtube.com/watch?v=HdafI0t3sEY&t=156)].
- It works by fetching relevant documents from an external knowledge base to provide context to the LLM for generating more accurate responses [[00:48](https://www.youtube.com/watch?v=HdafI0t3sEY&t=48)].
- RAG's modular design allows for flexibility in choosing and replacing components like the database or embedding model [[04:48](https://www.youtube.com/watch?v=HdafI0t3sEY&t=288)].

**Cache Augmented Generation (CAG)** [[01:24](https://www.youtube.com/watch?v=HdafI0t3sEY&t=84)]

- CAG involves preloading the entire knowledge base into the LLM's context window [[01:47](https://www.youtube.com/watch?v=HdafI0t3sEY&t=107)].
- The LLM then processes this information and stores it in a Key-Value (KV) cache [[05:31](https://www.youtube.com/watch?v=HdafI0t3sEY&t=331)].
- When a user asks a question, the LLM uses the pre-processed information in the KV cache to generate the answer [[06:25](https://www.youtube.com/watch?v=HdafI0t3sEY&t=385)].

**Comparison of RAG and CAG** [[07:55](https://www.youtube.com/watch?v=HdafI0t3sEY&t=475)]

The video compares the two methods based on several factors:

- **Accuracy:** RAG's accuracy is tied to the effectiveness of the retrieval mechanism, while CAG's accuracy depends on the LLM's ability to extract relevant information from a large context [[07:55](https://www.youtube.com/watch?v=HdafI0t3sEY&t=475)].
- **Latency:** CAG is generally faster than RAG because it eliminates the retrieval step during the online phase [[09:07](https://www.youtube.com/watch?v=HdafI0t3sEY&t=547)].
- **Scalability:** RAG is more scalable as it can handle much larger knowledge bases compared to CAG, which is limited by the LLM's context window size [[09:36](https://www.youtube.com/watch?v=HdafI0t3sEY&t=576)].
- **Data Freshness:** RAG allows for easier updates to the knowledge base by simply updating the index, whereas CAG requires recomputing the entire cache when the underlying data changes [[10:45](https://www.youtube.com/watch?v=HdafI0t3sEY&t=645)].

**Use Cases** [[11:31](https://www.youtube.com/watch?v=HdafI0t3sEY&t=691)]

The video illustrates the suitability of each method with practical examples:

- **CAG:** An IT help desk bot using a product manual would benefit from CAG due to the relatively small and static nature of the knowledge base [[11:45](https://www.youtube.com/watch?v=HdafI0t3sEY&t=705)].
- **RAG:** A research assistant for a law firm would be better suited to RAG because of the potentially vast and frequently updated legal documents [[12:43](https://www.youtube.com/watch?v=HdafI0t3sEY&t=763)].
- **Hybrid Approach:** A clinical support system for hospitals could potentially use a combination of both RAG and CAG to leverage the strengths of each method [[13:54](https://www.youtube.com/watch?v=HdafI0t3sEY&t=834)].

In conclusion, the choice between RAG and CAG depends on the specific requirements and characteristics of the use case, such as the size and volatility of the knowledge base, the importance of latency, and the need for accuracy [[15:17](https://www.youtube.com/watch?v=HdafI0t3sEY&t=917)].

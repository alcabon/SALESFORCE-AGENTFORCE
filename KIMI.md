### Quick English Glossary:  
- **Semantic search** = the *process* that understands meaning/intent and produces relevance scores.  
- **Top-K snippets** or **Top-K chunks** = the *result* of that process: the **K highest-scoring passages** that are actually inserted into the LLM’s context window.

---

### What “Semantic search → Top-K chunks” means
1. **Semantic search**  
   - Uses NLP (transformers like BERT or GPT) to convert both the **user query** and every **document chunk** into vectors.  
   - Computes **cosine similarity** (or another distance) to rank chunks by **meaning**, not keywords .

2. **Top-K chunks / Top-K snippets**  
   - After ranking, the system picks the **K best-scoring passages** (where K might be fixed—e.g. 5—or dynamically tuned ).  
   - These **K passages** (plain text) are concatenated and fed to the LLM as additional context.  
   - They are often called **snippets** when they come from live web search, and **chunks** when they come from a pre-segmented document.

---

### Key difference in one sentence  
> *Semantic search* is the **method** that ranks by meaning; *Top-K chunks/snippets* are the **concrete text blocks** it selects for the LLM.

```mermaid

%%{init:{'theme':'base'}}%%
flowchart TD
    A([User Input]) --> B{Input Type?}

    %% 1. Pasted / short text
    B -->|Paste / Type| C1([Full text kept<br>in context])
    C1 --> Z([LLM])

    %% 2. Uploaded large file
    B -->|Upload large file| D1([Store file<br>temporarily])
    D1 --> D2([Semantic search<br>→ Top-K chunks])
    D2 --> Z

    %% 3. Kimi external search
    B -->|Ask Kimi to search| E1([Live web search])
    E1 --> E2([Plain-text snippets])
    E2 --> E3([Top-K snippets])
    E3 --> Z

    %% Styling
    classDef green fill:#c2e0c6,stroke:#006100
    classDef yellow fill:#fce8b4,stroke:#8f7f00
    classDef blue  fill:#dae8fc,stroke:#002060
    class C1 green
    class D2,E3 yellow
    class Z blue
```

Cosine distance is simply how “close” two vectors (embeddings) are in direction.

1.  Each chunk and the user query are turned into **embedding vectors**—long lists of numbers.  
2.  **Cosine similarity** = the cosine of the angle between the two vectors.  
   • 0° → vectors point the same way → similarity = 1 (perfect match).  
   • 90° → vectors are orthogonal → similarity = 0 (no match).  
3.  **Cosine distance** = 1 − cosine similarity.  
   • 0 means identical direction (very close).  
   • 1 means opposite direction (very far).

So lower cosine distance ⇒ higher semantic relevance ⇒ chunk is chosen as one of the **Top-K**.

What the numbers in an embedding vector actually **mean**

1.  **Each position is a “latent feature”**  
    •  A 384-dim vector (MiniLM-L6-v2) has 384 **real numbers** (float32).  
    •  Position 57 might roughly encode “technicality”, position 123 “legality”, etc., but **there is no human label**; they are learned automatically during training.

2.  **Values are usually small decimals**  
    •  Typical range: –1 to +1, often around –0.2 … +0.4.  
    •  They are **unit-normalized** (length=1) so cosine similarity only measures **direction**, not magnitude.

3.  **Together they form a “semantic coordinate”**  
    •  Sentences with similar meaning end up close in this 384-D space.  
    •  Changing one word can shift many coordinates slightly, nudging the vector toward a new meaning cluster.

4.  **No single dimension is interpretable**  
    •  Unlike a database column (“price = 42”), no individual value has standalone meaning.  
    •  Only the **whole pattern** (all 384 numbers) captures the semantic content.

Think of the vector as a **compressed “meaning fingerprint”**: useless alone, powerful when compared to others.

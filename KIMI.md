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

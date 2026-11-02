# Medical Q&A Assistant with Retrieval-Augmented Generation

*Completed as part of the Post Graduate Program in Generative AI for Business Applications, UT Austin (2025).*

A question-answering assistant that answers clinical questions from the *Merck Manual of Diagnosis and Therapy* and cites the pages it used. The project then tests whether retrieval actually beats a plain LLM, using an LLM-as-judge evaluation.

**Stack:** Python · LangChain · ChromaDB · OpenAI GPT-4o-mini and text-embedding-3-small · pandas · Google Colab

## Problem

Clinicians need fast answers they can trust and trace back to a source. A general-purpose LLM answers fluently but can't show where an answer came from, which matters for clinical audit and liability.

## Approach

I answered the same four clinical questions (sepsis management, appendicitis, patchy hair loss, traumatic brain injury) three ways:

1. **Base LLM:** GPT-4o-mini answering directly.
2. **Prompt-engineered LLM:** the same model with a prompt tuned for structured, readable answers.
3. **RAG:** the manual is loaded with `PyPDFLoader`, split into 900-character chunks with 150-character overlap, embedded with `text-embedding-3-small` and stored in Chroma. The top 4 chunks are retrieved for each question, and the answer cites their page numbers.

An LLM judge then scored every answer from 1 to 5 against the retrieved evidence on five criteria. Groundedness was weighted 1.5× in the overall score.

## Results

| Criterion | Base LLM | Prompt-engineered | RAG |
|---|---|---|---|
| Relevance | 5.00 | 4.75 | 5.00 |
| Groundedness | 5.00 | 4.75 | 5.00 |
| Completeness | 4.25 | 4.00 | **4.75** |
| Specificity | 4.75 | 4.50 | 4.75 |
| Safety | 5.00 | 4.50 | 5.00 |
| **Weighted overall** | 4.81 | 4.53 | **4.91** |

**What this shows:**

- RAG scored highest overall, mostly because its answers were more complete.
- The base LLM came close. These conditions are well covered in its training data, so it can *look* grounded even when it is drawing on general knowledge.
- The deciding difference is **traceability**. Only RAG shows the page each claim came from, and it stays current when the source document is updated and re-embedded, without waiting for a new model.
- With four questions the score differences are small, so I read them as directional rather than conclusive.

## Recommendations

- Use RAG as the default for production, with prompt engineering layered on top for consistent formatting and inline citations.
- Add disclaimers ("general guidance, not individualized advice").
- Next steps: add more sources (hospital protocols, WHO and specialty guidelines), add a clinician feedback loop for unclear answers, and plan API rate limits and quotas up front, which was the hardest practical constraint in this project.

## Running it

Open `MedicalAssistant.ipynb` in Google Colab. You'll need your own OpenAI API key in a `config.json` file and a copy of the Merck Manual PDF. Neither is included in this repo.

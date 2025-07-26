**Setup Guide**

\!pip install \-U langchain langchain-community transformers accelerate sentence-transformers faiss-cpu pypdf pymupdf

\!apt install poppler-utils \-y

\!apt install tesseract-ocr \-y

\!apt install tesseract-ocr-ben \-y

\!pip install pytesseract pdf2image

Mount Google Drive and run the script in Google Colab. Upload the pdf in google drive and provide the pdf paths in code accordingly. 

**Tools and Libraries Used:**

* **OCR**: pytesseract with tesseract-ocr-ben  
* **PDF to Image**: pdf2image  
* **Embedding Model**: sentence-transformers/paraphrase-multilingual-mpnet-base-v2  
* **Similarity Search**: FAISS  
* **QA Model**: deepset/xlm-roberta-large-squad2  
* **Evaluation**: cosine\_similarity from sklearn

**Evaluation Matrix**

* Metric: **Cosine Similarity** of true vs predicted answers (embedding space)  
* Average Similarity on test set: 0.9629

**1\. What method or library did you use to extract the text, and why? Did you face any formatting challenges with the PDF content?**

**Library Used**: pytesseract (with tesseract-ocr-ben for Bangla) \+ pdf2image

The source PDF was image-based (scanned document), not text-based. So OCR (Optical Character Recognition) was required.

**Challenges**:

* OCR errors due to poor scan quality (misrecognized characters, merged words).  
* Formatting loss: paragraphs, headings, and punctuation were inconsistently detected.  
* Required cleaning steps like whitespace normalization and newline trimming.

**2\. What chunking strategy did you choose (e.g. paragraph-based, sentence-based, character limit)? Why do you think it works well for semantic retrieval?**

**Strategy**: Fixed-size, word-level chunking — 100 words per chunk with 20-word overlap.

* Ensures each chunk has enough semantic context.  
* Overlap reduces context loss across boundaries.  
* More stable than sentence-based chunking for noisy OCR output where punctuation is unreliable.

**Effectiveness**: Works well for semantic retrieval, especially when paragraphs or sentence boundaries are unclear.

**3\. What embedding model did you use? Why did you choose it? How does it capture the meaning of the text?**

**Model**: sentence-transformers/paraphrase-multilingual-mpnet-base-v2

Chose this one because-

* Supports multilingual input including Bangla.  
* Optimized for semantic similarity tasks.  
* Pretrained on large multilingual datasets, capturing cross-lingual paraphrasing.

It works by converting each sentence or chunk into a dense vector (embedding). Also, similar meanings produce vectors that are close in embedding space.

**4\. How are you comparing the query with your stored chunks? Why did you choose this similarity method and storage setup?**

**Comparison Method**: Cosine similarity between query embedding and chunk embeddings.

**Storage**: FAISS (IndexFlatL2) for efficient approximate nearest neighbor (ANN) search.

* Cosine similarity is effective for measuring semantic closeness in embedding space.  
* FAISS allows fast similarity search across large sets of vectors with low latency.  
* Scalable and widely used in production retrieval systems.

**5\. How do you ensure that the question and the document chunks are compared meaningfully? What would happen if the query is vague or missing context?**

* Semantic embeddings align the meaning of questions and document chunks, even in different languages.  
* Retrieved chunks and previous Q\&A pairs (short-term memory) are passed together to the QA model for context.

If the Query is Vague:

* Retrieved chunks may be off-topic.  
* QA model may return incorrect or generic answers.  
* In such cases, prompting user clarification or using a fallback strategy (e.g., query expansion) is necessary.

**6\. Do the results seem relevant? If not, what might improve them (e.g. better chunking, better embedding model, larger document)?**

Most of the times, the results are accurate but sometimes can be irrelevant due to:

1. OCR Errors: Inaccurate text extraction can introduce noise, leading to irrelevant answers.  
2. Chunking Limitations: Fixed-size word chunks with overlap may not always capture enough context, splitting important information.  
3. Embedding Model: The model might miss nuanced meanings in complex or less frequent language constructs, like in Bangla.  
4. Short-Term Memory: Limited context in short-term memory could cause the system to miss important details across multi-turn conversations.

Improvements:

* **Better OCR Processing:** Use spell correction and denoising techniques.  
* **Refined Chunking:** Switch to paragraph-based chunking or use dynamic chunk sizes  
* **Embedding Model**: Fine-tune for Bangla or use a more context-aware model.  
* **Larger Corpus:** Include more documents for richer retrieval.  
* **Enhanced Memory:** Expand the context window for more accurate multi-turn responses.


# RAG and LangChain Notebook Guide

This guide maps the project notebooks to the concepts they demonstrate, what
each workflow does, and why you might use it. The notebooks are learning
examples; exact package APIs and model availability can vary by environment.

## Recommended learning order

1. Learn how to load and parse source files in `0-DataIngestionAndParsing/`.
2. Learn what embeddings are and how text becomes vectors in
   `1-EmbeddingAndDatabases/`.
3. Store and retrieve chunks with Chroma or FAISS in `vector/`.
4. Compare semantic chunking approaches in `src/ragai/SemanticChunk/`.
5. Combine lexical and semantic retrieval in `src/ragai/search/`.

## Data ingestion and parsing

Loaders convert files into LangChain `Document` objects. A document usually
contains `page_content` plus `metadata` such as a source path, row number, or
page number. Parsing is the first step before splitting, embedding, indexing,
or retrieval.

| Notebook | What it demonstrates | Why it is useful |
|---|---|---|
| [1-dataingestion.ipynb](./0-DataIngestionAndParsing/1-dataingestion.ipynb) | Introduction to ingestion and common text-splitting approaches. | Shows how to prepare long text for downstream retrieval and why chunk boundaries matter. |
| [2-dataingestionpdf.ipynb](./0-DataIngestionAndParsing/2-dataingestionpdf.ipynb) | Loading a PDF with PyMuPDF and trying text splitters. | Extracts searchable text from PDFs and breaks it into manageable passages. |
| [3-smart_pdf_ingestion.ipynb](./0-DataIngestionAndParsing/3-smart_pdf_ingestion.ipynb) | A reusable PDF-ingestion class with text cleanup, splitting, and validation-oriented structure. | Keeps repeated PDF processing steps together and makes ingestion behavior easier to reuse. |
| [4-docs_data_parsing.ipynb](./0-DataIngestionAndParsing/4-docs_data_parsing.ipynb) | Parsing Word documents with `Docx2txtLoader` and an Unstructured loader. | Demonstrates alternatives for extracting text or structured elements from DOCX files. |
| [5-csv_parsing.ipynb](./0-DataIngestionAndParsing/5-csv_parsing.ipynb) | Creating a CSV and loading it with `CSVLoader`. | Provides a straightforward route for turning tabular rows into documents. |
| [6-excel_parsing.ipynb](./0-DataIngestionAndParsing/6-excel_parsing.ipynb) | Creating and parsing an Excel workbook with `UnstructuredExcelLoader`. | Shows how spreadsheet content can be made available to document-oriented pipelines. |
| [7-csv_intelligent_parsing.ipynb](./0-DataIngestionAndParsing/7-csv_intelligent_parsing.ipynb) | Custom row-wise CSV parsing and document metadata. | Gives control over row representation and preserves useful identifiers such as row index and file path. |
| [8-json_parsing.ipynb](./0-DataIngestionAndParsing/8-json_parsing.ipynb) | Custom parsing of JSON data into documents for entities such as employees and projects. | Structures JSON records as focused documents instead of embedding one large JSON blob. |
| [9_sqllite_data_parsing.ipynb](./0-DataIngestionAndParsing/9_sqllite_data_parsing.ipynb) | Connecting to SQLite and loading/querying database content. | Demonstrates access to structured relational data as input to a LangChain workflow. |

## Embeddings

An embedding model maps text to a numeric vector. Similar meanings tend to have
vectors that are close under a similarity measure. Embeddings support semantic
search; they are not generated answers and are not usually human-readable.

| Notebook | What it demonstrates | Why it is useful |
|---|---|---|
| [1_huggingface-sentence-embedding.ipynb](./1-EmbeddingAndDatabases/1_huggingface-sentence-embedding.ipynb) | Creating sentence embeddings with a Hugging Face model. | Builds intuition for vectors, vector dimensions, and semantic similarity. |
| [2_huggingface-embedding-textfile.ipynb](./1-EmbeddingAndDatabases/2_huggingface-embedding-textfile.ipynb) | Loading and splitting a text file before embedding its content. | Connects raw-file ingestion and chunking to the embedding stage. |

## Vector stores and RAG

RAG (retrieval-augmented generation) retrieves relevant documents for a query
and supplies them as context to a language model. A vector store indexes
embeddings and supports similarity-based retrieval. These notebooks compare
Chroma and FAISS and show how retrieval can be combined with generation.

| Notebook | What it demonstrates | Why it is useful |
|---|---|---|
| [1-chromadb.ipynb](./vector/1-chromadb.ipynb) | Conversational RAG using Chroma, chunking, embeddings, retrieval, question rewriting, and answer generation. | Demonstrates how chat history can clarify follow-up queries while retrieved context grounds answers. |
| [2-faiss.ipynb](./vector/2-faiss.ipynb) | Building and saving/loading a FAISS index, retrieving documents, and constructing a conversational RAG chain. | Shows a local vector index workflow and how its retrieval step fits into a RAG pipeline. |
| [practice_with_pdf_resume.ipynb](./vector/practice_with_pdf_resume.ipynb) | Applying PDF loading, chunking, Chroma, and conversational retrieval to a resume. | Provides an end-to-end example using a real document rather than hand-written sample passages. |

## Semantic chunking

Chunking determines which text is embedded and retrieved together. Fixed-size
splitters use character or token limits. Semantic chunking instead uses
sentence meaning or embedding similarity to choose boundaries. It can preserve
coherent ideas, but its output depends on the input granularity, model, and
threshold.

| Notebook | What it demonstrates | Why it is useful |
|---|---|---|
| [1-semantic-chunk-python.ipynb](./src/ragai/SemanticChunk/1-semantic-chunk-python.ipynb) | A manual sentence-embedding and cosine-similarity approach for grouping adjacent sentences. | Makes the semantic-boundary logic visible and tunable rather than hiding it inside a library. |
| [2-custom-semantic-chunk-rag-pipeline.ipynb](./src/ragai/SemanticChunk/2-custom-semantic-chunk-rag-pipeline.ipynb) | Packaging custom semantic chunking in a class and using the resulting chunks in a FAISS RAG flow. | Connects a custom chunker to indexing, retrieval, prompting, and answer generation. |
| [3-semantic-chunk-langchain.ipynb](./src/ragai/SemanticChunk/3-semantic-chunk-langchain.ipynb) | Using LangChain Experimental's `SemanticChunker` with an embeddings provider. | Offers a library-managed alternative to implementing boundary logic manually. |

## Hybrid search

| Notebook | What it demonstrates | Why it is useful |
|---|---|---|
| [hybrid-search.ipynb](./src/ragai/search/hybrid-search.ipynb) | Combining dense vector retrieval (FAISS) with sparse BM25 retrieval through an ensemble retriever, then passing results into a retrieval QA chain. | Dense search can find paraphrases and related concepts; BM25 can match exact terms, names, or identifiers. Combining them can improve recall when either method alone misses relevant passages. |

## Choosing an approach

- Use **standard character/recursive chunking** as a simple baseline. It is
  predictable and does not require sentence embeddings to choose boundaries.
- Try **semantic chunking** when a fixed-size split regularly separates ideas
  that should remain together. Compare retrieval quality rather than assuming
  semantic chunks are always better.
- Use a **vector store** when semantic similarity is important and you need an
  index over many chunks.
- Use **BM25** when exact words, names, acronyms, or identifiers matter.
- Use **hybrid search** when both exact lexical matches and semantic matches
  are useful.
- Use **conversational RAG** when questions depend on earlier turns. Store chat
  history per conversation, rewrite follow-up questions for retrieval, and
  verify the retrieved context when an answer is incomplete.

## Practical notes

- Inspect the chunks and retrieved documents before diagnosing an answer
  generation problem. The model cannot ground an answer in material that was
  not retrieved.
- Rebuild or use a fresh vector-store collection after changing chunking or
  source documents; an existing persisted index does not automatically adopt
  new chunks.
- Keep metadata that helps trace a result back to its source.
- Some notebooks call hosted embedding or chat APIs and need credentials;
  others download local embedding models and need those model dependencies.
- Avoid exposing personal or confidential source documents when sharing
  notebook outputs.

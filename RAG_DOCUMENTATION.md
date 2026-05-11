# RAG Application Documentation

Welcome to the documentation for your **Retrieval-Augmented Generation (RAG)** project! This document explains how the entire system works, the purpose of each file and folder, and the architecture behind the application.

## Overview

This project implements a RAG pipeline. RAG is a technique that enhances Large Language Models (LLMs) by giving them access to an external knowledge base (in this case, PDF documents) so they can answer questions accurately based on *your specific data* without hallucinations.

The application has two main interfaces:
1. **Command Line Interface (CLI):** Driven by `main.py`
2. **Web Interface:** Driven by `app.py` using Streamlit.

Both interfaces use Google's Gemini models (`gemini-2.5-flash` for text generation and `gemini-embedding-001` for vector embeddings) through the LangChain framework.

---

## Folder & File Structure

Here is a breakdown of what is stored in each folder and file within your project:

### 📂 Folders
* **`chroma_db/`**
  * **What it stores:** The Chroma vector database.
  * **Explanation:** When your PDFs are processed, their text is converted into numbers (embeddings) and saved here locally on your hard drive. This allows the application to perform fast semantic searches across your documents without needing to process the PDF every time.
* **`document loaders/`**
  * **What it stores:** Source PDF files.
  * **Explanation:** This is where local documents like `deeplearning.pdf` are stored before they are processed by the database creation script.
* **`venv/`**
  * **What it stores:** Python Virtual Environment.
  * **Explanation:** Contains all the isolated Python packages and dependencies installed for this specific project.

### 📄 Files
* **`create_database.py`**
  * **Purpose:** A standalone Python script to manually populate the vector database from a local PDF.
  * **How it works:** It loads `deeplearning.pdf` from the `document loaders/` folder, splits the text into manageable chunks, converts those chunks into embeddings using Google's embedding model, and saves them into the `chroma_db` folder. *(Note: Currently, it's set to only process the first 10 chunks as a test).*
* **`main.py`**
  * **Purpose:** The Command Line Interface (CLI) application.
  * **How it works:** You run this in your terminal. It loads the existing database from `chroma_db`. It sets up a loop asking for your input (`You:`). When you ask a question, it searches the database for relevant document chunks, injects them into a prompt template, and sends them to Gemini to get a factual answer.
* **`app.py`**
  * **Purpose:** The Streamlit Web Application.
  * **How it works:** Provides a graphical user interface (GUI) in your browser. It allows you to dynamically upload *any* PDF through the web page. It handles the entire pipeline: saving the file temporarily, chunking, creating the vector database, and finally providing a chat interface to ask questions about the newly uploaded document.
* **`.env`**
  * **Purpose:** Environment variables file.
  * **Explanation:** Securely stores your secret API keys (like `GOOGLE_API_KEY`) so they aren't hardcoded into your scripts. The `python-dotenv` package loads these automatically.
* **`requirements.txt`**
  * **Purpose:** Dependency list.
  * **Explanation:** A list of all the Python packages (like `langchain`, `streamlit`, `chromadb`, etc.) required to run the project. You install them using `pip install -r requirements.txt`.

---

## How the RAG Pipeline Works (Step-by-Step)

Regardless of whether you use `app.py` or `main.py`, the core RAG architecture follows these 5 steps:

### Phase 1: Ingestion (Preparing the Knowledge Base)
1. **Document Loading:** 
   The application uses `PyPDFLoader` to read the raw text from your PDF file.
2. **Text Splitting (Chunking):** 
   Since LLMs have a limit on how much text they can read at once, `RecursiveCharacterTextSplitter` breaks the large document into smaller chunks (1000 characters each, with a 200-character overlap so sentences aren't abruptly cut off).
3. **Embedding & Storage:** 
   `GoogleGenerativeAIEmbeddings` takes each chunk of text and converts it into a high-dimensional vector (an array of numbers representing the semantic meaning of the text). These vectors are then stored persistently in the `chroma_db` vector store.

### Phase 2: Retrieval & Generation (Answering Questions)
4. **Retrieval:** 
   When a user asks a question, the application converts the question into a vector using the same embedding model. It then performs a similarity search inside `chroma_db` using an algorithm called MMR (Maximal Marginal Relevance) to fetch the top most relevant chunks of text from the PDF.
5. **Generation:** 
   The retrieved text chunks (the "Context") and the user's original question are combined into a strict Prompt Template. This prompt instructs the `gemini-2.5-flash` model to act as an AI assistant and answer the question **ONLY** using the provided context. If the answer isn't in the context, it gracefully replies that it couldn't find the answer.

---

## Summary of Technologies Used
* **LangChain:** The overarching framework used to tie all the LLM components together (loaders, splitters, retrievers, prompts).
* **Streamlit:** The framework used in `app.py` to create the web-based user interface.
* **Google Generative AI:** Provides the intelligence for the app. Uses `gemini-embedding-001` for embeddings and `gemini-2.5-flash` for the chat responses.
* **Chroma DB:** The local, open-source vector database used to store and search the document embeddings.

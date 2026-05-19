# VisiRAG

VisiRAG (**Visual Retrieval-Augmented Generation**) is a multimodal RAG system that allows users to upload PDFs containing text, tables, and images, and then interactively query the documents. The system retrieves relevant information and generates responses enriched with both text and images, providing a seamless multimodal experience.

The project is split into two main components:

* [`backend.py`](https://www.google.com/search?q=backend.py) – Handles PDF parsing, image extraction, embeddings, vector store creation, and query processing with LangChain and Azure OpenAI.
* [`frontend.py`](https://www.google.com/search?q=frontend.py) – A Streamlit-based user interface for uploading PDFs, chatting with the RAG engine, and viewing inline images in responses.

---

## ✨ Features

* 📄 **PDF ingestion**: Extracts text, tables, and images from PDFs using PyMuPDF, pdfplumber, and pdfminer.
* 🖼 **Image descriptions**: Generates AI-powered descriptions for extracted images (via Azure OpenAI).
* 🔍 **Semantic search**: Uses HuggingFace embeddings with Chroma vector store for retrieval.
* 🤖 **RAG workflow**: Combines document retrieval with Azure OpenAI’s LLM for contextual answers.
* 💬 **Conversational memory**: Maintains chat history for context-aware conversations.
* 🎨 **Streamlit UI**: Upload PDFs, ask questions, and view inline images in responses interactively.

---

## 📐 System Architecture

The workflow engine coordinates file ingestion, external model indexing, state tracking, and UI compilation across a decoupled framework:

```mermaid
flowchart TD
    %% Styling configurations
    classDef inputStyle fill:#e3f2fd,stroke:#1565c0,stroke-width:2px,color:#000;
    classDef engineStyle fill:#f1f8e9,stroke:#558b2f,stroke-width:2px,color:#000;
    classDef storageStyle fill:#fff3e0,stroke:#ef6c00,stroke-width:2px,color:#000;
    classDef apiStyle fill:#f3e5f5,stroke:#6a1b9a,stroke-width:2px,color:#000;
    classDef uiStyle fill:#fafafa,stroke:#37474f,stroke-width:2px,color:#000;

    %% Data Input Tier
    subgraph Input_Tier [Document Input]
        pdf["PDF Document File"]
    end

    %% Hybrid Loader Subgraph
    subgraph Loaders [Hybrid Loader - backend.py]
        pdf_miner["pdfminer.six<br>Text Engine"]
        pdf_plumber["pdfplumber<br>Table and BBox Engine"]
        pymupdf["PyMuPDF<br>Image Engine"]
    end

    %% Orchestration Pipeline Subgraph
    subgraph Orchestration [RAG Pipeline Orchestration]
        splitter["Recursive Character Splitter<br>Isolated Image Chunks"]
        img_mgr["Image Manager<br>Multi-format Mapping"]
        retrieval_chain["Conversational Retrieval Chain<br>LangChain and Memory"]
    end

    %% Storage & Artifacts Subgraph
    subgraph Storage [Vector Database & Directory Storage]
        chroma_db[("Chroma DB<br>rag_collection")]
        extracted_imgs[("Extracted Images Folder")]
    end

    %% External Interfaces Subgraph
    subgraph External_APIs [External Cloud APIs]
        azure_openai["Azure OpenAI<br>GPT Vision / LLM"]
        hf_embeddings["HuggingFace<br>e5-large-v2"]
    end

    %% Presentation Layer Subgraph
    subgraph Frontend [Streamlit UI - frontend.py]
        st_ui["Streamlit UI Interface"]
        st_state["Session State<br>Engine and Chat History"]
        st_render["Dynamic Layout Renderer<br>Inline Text and Images"]
    end

    %% Execution and Processing Flows
    pdf --> Loaders
    
    pymupdf -->|Saves Raw Bytes| extracted_imgs
    pymupdf -->|Sends Base64 Payload| azure_openai
    azure_openai -->|Returns Structured Description| pymupdf
    
    pdf_miner --> splitter
    pdf_plumber --> splitter
    pymupdf --> splitter
    
    splitter --> hf_embeddings
    hf_embeddings -->|Dense Vector Insertions| chroma_db
    
    Loaders -->|Passes Doc Metadata| img_mgr
    img_mgr -->|Maps System File Paths| extracted_imgs
    
    %% User Query & Runtime Inference Flows
    st_ui --> st_state
    st_state -->|Invokes query pipeline| retrieval_chain
    
    retrieval_chain -->|Vector Similarity Search| chroma_db
    retrieval_chain -->|Context and Query Prompt| azure_openai
    retrieval_chain -->|Regex Image Reference Lookups| img_mgr
    
    retrieval_chain -->|Returns Structured JSON Payload| st_render
    st_render --> st_ui

    %% Apply Styles
    class pdf inputStyle;
    class pdf_miner,pdf_plumber,pymupdf,splitter,img_mgr,retrieval_chain engineStyle;
    class chroma_db,extracted_imgs storageStyle;
    class azure_openai,hf_embeddings apiStyle;
    class st_ui,st_state,st_render uiStyle;

```

---

## ⚡ Installation

Clone this repository:

```bash
git clone https://github.com/your-username/VisiRAG.git
cd VisiRAG

```

Create and activate a virtual environment:

```bash
python -m venv venv
source venv/bin/activate   # On Linux/Mac
venv\Scripts\activate      # On Windows

```

Install dependencies with **uv**:

```bash
uv pip install -e .

```

Or, if you don’t want editable mode:

```bash
uv pip install .

```

This will install everything defined in your `pyproject.toml`.

> **Note:** Make sure you have Python **3.12** installed.

---

## 🔑 Environment Variables

Before running, create a `.env` file in the root directory with the following:

```env
AZURE_OPENAI_DEPLOYMENT_NAME=your-deployment
AZURE_OPENAI_API_KEY=your-api-key
AZURE_OPENAI_API_VERSION=your-api-version
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/

```

---

## 🚀 Usage

Run the Streamlit frontend:

```bash
streamlit run frontend.py

```

Steps:

1. Upload a PDF file.
2. Ask natural language questions about the content.
3. View responses with inline text and extracted images.

---

## 🗂 Project Structure

```text
VisiRAG/
│── .env              # Environment variables
│── .python-version   # Python version specification
│── LICENSE           # License file
│── backend.py        # Core RAG engine (PDF processing, embeddings, query handling)
│── frontend.py       # Streamlit-based chat UI
│── pyproject.toml    # Project dependencies and metadata
│── uv.lock           # Lockfile for reproducible installs
│── README.md         # Project documentation

```

---

## 📜 MIT License

```text
MIT License

Copyright (c) 2025 Aadarsh Pandey D

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.

```

---

## 🙌 Acknowledgments

* LangChain
* Streamlit
* Chroma
* Azure OpenAI
* PyMuPDF
* [pdfplumber]()
* [pdfminer.six]()

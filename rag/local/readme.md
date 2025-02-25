# Building a RAG application with Local Compute

## Overview

Sample RAG application that shows how you can use local compute to build and run your RAG application

1. Dataset in your machine; Sample includes a cat facts data provided in folder [here](./data/cat-facts.txt)
2. Load the dataset
3. Create embeddings for your dataset that will serve as the knowledge source for the RAG application
   1. Create chunks from your dataset (in this example, each entry line in the dataset is treated as a chunk)
   2. Choose a text embedding model
   3. Convert each chunk into an embedding vector
   4. Store the chunk and its corresponding vector into a list
4. User Input: Get user query
5. Retrieval:
   1. Create an embedding for the user query.
   2. Calculate the similarity between the input query and the vectors in the knowledge store
   3. Find the top N similar vectors
6. Generation: Use LLM model and a system instruction to generate a response to the user query based on the retrieved knowledge

**Language**:

- Python

**Tools:**

- ollama

**Models:**

- Embedding model: hf.co/CompendiumLabs/bge-base-en-v1.5-gguf
- Language model: hf.co/bartowski/Llama-3.2-1B-Instruct-GGUF

**Dataset:**

- [cat facts](https://huggingface.co/ngxson/demo_simple_rag_py/blob/main/cat-facts.txt)

## Getting Started

1. Install ollama from [ollama.com](https://ollama.com/)
2. Run the embedding and language models (mentioned in the above section) using the following command:
    `ollama pull hf.co/CompendiumLabs/bge-base-en-v1.5-gguf`
    `ollama pull hf.co/bartowski/Llama-3.2-1B-Instruct-GGUF`
3. pip install the requirements
4. Follow step by step instructions in the [notebook](byo-rag.ipynb) or use the python file `python byo-rag.py`; Make sure you are in the rag/local folder while running this command

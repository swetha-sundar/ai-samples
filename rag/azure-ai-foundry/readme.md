# Overview

Provides a simple build-your-own rag with Azure AI Foundry, AI Search and Azure OpenAI

## Setup

TODO: Add infra setup instructions

### Detailed Steps

1. Add your data to your storage (local is used in this example); Optionally you can push it to Storage Account and add a simple logic to read data from blob
2. Exploratory Data Analysis - Inspect and understand your data
3. Indexing your data using Azure AI Search
   1. CONFIGURATION:
      1. Configuration your Azure AI Project Client
      2. Create client for embedding model (i.e., the embedding inference client)
      3. Retrieve a search connection object from AI Project instance
      4. Create search client for indexing using the connection object above
   2. INDEX DEFINITION:
      1. Set the right parameters/properties for the index for the data
      2. Choose the algorithm (cosine distance HNSW algo chosen here) for similarity; Uses semantic ranking
   3. INDEX CREATION:
      1. Using the specified index definition and embedding model, create an index
      2. Loads the data, generates embeddings/vectors
      3. Uploads the vectorized data into pre-defined search index
      4.
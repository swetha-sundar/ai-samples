# Overview

Provides a simple build-your-own rag with Azure AI Foundry, AI Search and Azure OpenAI

## Setup

TODO: Add infra setup instructions

## Explanations for each step in the RAG

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
   4. Verify index is created
4. RETRIEVAL
   1. CONFIGURATION: similar as above to fetch model and search clients
   2. Receive input query from user
   3. Map user query to an intent (intent mapping - refer to the prompt used); this provides us the search query
   4. Create embedding or vectorize the search query
   5. Search the index for matches on the search query
   6. For each match, retrieve the document/content
   7. Return the document(s) to the user
5. AUGMENTATION + GENERATION
   1. Configure/Fetch the Azure AI Project Client
   2. Retrieve the Azure Open AI Chat Completions model inference client
   3. Use incoming user query to retrieve related product documents
   4. Use this knowledge to populate a "grounded" chat prompt template
   5. Call the chat completions inference client with the grounded prompt and generate a response for the user query
6. TEST/TRY IT OUT QUERY SAMPLES for the data
   1. "I need a new tent for 4 people, what would you recommend?"
   2. "Which tent is good for bug protection and rainy days?"
   3. "What food do cats like?" --> The response should be like "Sorry, I can only answer queries realted to camping gear.."

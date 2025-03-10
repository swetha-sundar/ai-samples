# Overview

Provides a simple build-your-own rag with Azure AI Foundry, AI Search and Azure OpenAI

## Setup

TODO: Add infra setup instructions/scripts

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
   3. "What food do cats like?" --> The response should be like "Sorry, I can only answer queries related to camping gear.."

## Getting Started

1. Create a virtual environment
    `python3 -m venv .aifoundryenv`
    `source .aifoundryenv/bin/activate`
2. Install dependencies - `pip install -r requirements.txt`
3. Use portal or infra scripts to create Azure AI Hub and Project Resources
4. .....
5. Check the "Authentication Type" on the workspaceblobstore and workspaceartifactstore. If it is "Account Key" based authentication, then ensure that the Storage Account has "Account Key Access" Enabled in the Configuration settings
6. Ensure the user has Blob Data Contributor access on the Storage Account; Follow below steps to set it right
    4a. Fetch the users `az ad user list`
    4b. Copy the object id for the respective user
    4c. Assign the role for the chosen user
        `az role assignment create --role "Storage Blob Data Contributor" --scope /subscriptions/<mySubscriptionID>/resourceGroups/<myResourceGroupName> --assignee-principal-type User --assignee-object-id "<user-id>"`

        <mySubscriptionID>: Subscription ID of the Azure AI Studio hub's linked storage account (available in Azure AI hub resource view in Azure Portal).
        <myResourceGroupName>: Resource group of the Azure AI Studio hub's linked storage account.
        <user-id>: User object ID for role assignment (retrieve with "az ad user show" command).
7. Run az login
8. Ensure you copy .env.template file to .env
9. Populate the AI project connection string  (You can fetch this from Project Overview page with AI Foundry Portal)
10. Ensure the dependencies are installed from step 2.
11. Configuration: `python 1-config.py`
12. Search Index Definition and Creation: `python 2-create-search-index.py`
13. Retrieve Product Documents: `python 3-get-product-documents.py` You will be prompted for a user query; optionally you can pass it in as a command line argument; see code file
14. Chat with Products (Generate a response for user query instead of list of documents): `python 4-chat-with-products.py`; optionally you can pass it in as a command line argument; see code file

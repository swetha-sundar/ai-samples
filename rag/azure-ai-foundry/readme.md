# Overview

Provides a simple build-your-own rag with Azure AI Foundry, AI Search and Azure OpenAI

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

1. Use portal or infra scripts to create an `Azure AI Foundry` resource, which will generate an AI Hub.
2. After creating the resource, from the Overview page you can launch the *Azure AI Foundry portal*.
3. In the AI Foundry portal, create a new `AI Project`.
4. Deploy these 2 models, used for embedding and chat completions (and customizable in `.env` file). Ensure the models you deploy are available in your region (check [official documentation](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/models?tabs=global-standard%2Cstandard-chat-completions#model-summary-table-and-region-availability))
   * `text-embedding-ada-002`
   * `gpt-4o-mini`

   ![image](assets/deployModels.png)

   *Note: If you change one of the models, remember to adjust the model names used in your .env file*.

5. In Azure portal, create a new `Azure AI Search` resource, in the same resource group as the AI Foundry resource.
6. In VSCode, go to *rag/azure-ai-foundry* and ensure you copy `.env.template` to a new `.env` file.
7. Populate the `AIPROJECT_CONNECTION_STRING` - You can fetch this from Project Overview page in AI Foundry Portal.
8. Create a virtual environment by running the following commands in VSCode terminal:

   ```bash
   python3 -m venv .aifoundryenv
   source .aifoundryenv/bin/activate
   ```

9. Install dependencies:

   ```bash
   cd rag/azure-ai-foundry/
   pip install -r requirements.txt
   ```

10. Login to your Azure subscription:

   ```bash
   az login
   ```

11. Run the configuration file, to setup telemetry for all troubleshooting and logging:

   ```bash
   python config.py
   ```

12. There will be two **`Connected Resources`** in you AI Project, namely a `workspaceblobstore` and a `workspaceartifactstore`; Check the "Authentication Type" for both by selecting each one of them in the Overview page of your AI Project. By default, it should be *Credential based*. If it's *Account Key based* authentication, then ensure that the Storage Account has *Account Key Access* Enabled in the Configuration settings.

13. Ensure the user has *Blob Data Contributor* access on the Storage Account, and *Search Index Data Contributor* access on the Azure AI Search resource. Follow below steps to set it right:
    1. Fetch the id of your user

        ```bash
            az ad signed-in-user show
        ```

      Note: if you need the id of a different user, you can use the following command, replacing the email address with the proper one: `az ad user list --filter "mail eq 'your-email@microsoft.com'" --output json`

    2. Copy the object id for the respective user
    3. Assign the roles to the user, replacing the placeholder items as explained below

      ```bash
      az role assignment create --role "Storage Blob Data Contributor" --scope /subscriptions/<mySubscriptionID>/resourceGroups/<myResourceGroupName>/providers/Microsoft.Storage/storageAccounts/<myStorageAccountName> --assignee-principal-type User --assignee-object-id "<user-id>"

      az role assignment create --role "Search Index Data Contributor" --scope /subscriptions/<mySubscriptionID>/resourceGroups/<myResourceGroupName>/providers/Microsoft.Search/searchServices/<mySearchServiceName> --assignee-principal-type User --assignee-object-id "<user-id>"
      ```

      `mySubscriptionID`: Subscription ID of the Azure AI Studio Hub's linked storage account (available in Azure AI Hub resource view in Azure Portal).

      `myResourceGroupName`: Resource group of the Azure AI Studio Hub's linked storage account.

      `user-id`: User object ID for role assignment, retrieved on step 4a.

      `myStorageAccountName`: Storage account name of the Azure AI Studio Hub's linked storage account.

      `mySearchServiceName`: Azure AI Search service name of the Azure AI Studio Hub's linked search service.

14. In the AI Foundry portal, link the Azure AI Search resource to the AI Project, by clicking on the `New Connection` button in the `Connections` tab of the AI Project. Select the `Azure AI Search` connection type and you should automatically see your resource. If not, you can manually provide the required information.

   ![image](assets/newConnection.png)

15. Search Index Definition and Creation:

   ```bash
   python create_search_index.py
   ```

   Once the command completes, you should see the index created in the Azure AI Search resource.

   ![image](assets/index.png)
   ![image](assets/indexSearch.png)

16. Retrieve Product Documents:

   ```bash
   python get_product_documents.py
   ```

   Optionally you can pass a query in as a command line argument.

17. Chat with Products (Generate a response for user query instead of list of documents):

   ```bash
   python chat_with_products.py

   # optionally you can pass query as a command line argument
   python chat_with_products.py --query "What is the best tent for families?"
   ```

## Troubleshooting guide

* Error message when you try to achieve the model you deployed in AI Foundry portal:

   ```text
   azure.core.exceptions.HttpResponseError: (None): The server had an error while processing your request. Sorry about that!
   ```

   *This might appear when you run `python get_product_documents.py` and/or `python chat_with_products.py`*

   **Troubleshooting steps**

   1. Ensure that the model names in your `.env` file match the model names in the Azure AI Foundry portal.

   2. Ensure that the model is available in your region. You can check the availability of the model in your region by visiting the [official documentation](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/models?tabs=global-standard%2Cstandard-chat-completions#model-summary-table-and-region-availability).

   3. Try to deploy a different model that is available in your region, and update the model names in your `.env` file accordingly.

# Indexing file contents and metadata in Azure Cognitive Search

This sample demonstrates how to use [multiple indexers](https://learn.microsoft.com/azure/search/search-indexer-overview#indexer-scenarios-and-use-cases) in [Azure Cognitive Search](https://learn.microsoft.com/azure/search/search-what-is-azure-search) to create a single [search index](https://learn.microsoft.com/azure/search/search-what-is-an-index) from files in [Blob storage](https://learn.microsoft.com/azure/storage/blobs/storage-blobs-overview), with additional file metadata in [Table storage](https://learn.microsoft.com/azure/storage/tables/table-storage-overview).

## Getting Started

By following this sample, you will create an Azure Storage account which has a number of files uploaded to blob storage. You will then create an Azure Cognitive Search service which indexes these files so that you can search for information contained within them. By combining a search indexer for [Azure Blob Storage](https://learn.microsoft.com/azure/search/search-howto-indexing-azure-blob-storage) and a separate indexer for [Azure Table Storage](https://learn.microsoft.com/azure/search/search-howto-indexing-azure-tables), each document in the search index will contain the joint metadata from both places.

In this sample, the search index contains `author`, `document_type` and `business_impact` metadata fields. The `author` is retrieved directly from the files in Blob storage, whereas the `document_type` and `business_impact` values come from a row in Table storage for each associated blob.

By default, a few basic text files are uploaded to Blob storage from the `samplefiles` directory in this repository. You can put additional files in the subdirectories, as long as they have one of the [supported document formats](https://learn.microsoft.com/azure/search/search-howto-indexing-azure-blob-storage#supported-document-formats). The directory name of the file (for example, `paper` or `report`) is used as the `document_type` metadata value, so you can also create additional subdirectories to have more document types to search for. If you'd like to use more interesting files to index, you can find sample data sets in the [Azure Cognitive Search Sample Data
 repository](https://github.com/Azure-Samples/azure-search-sample-data).

### Prerequisites

- An active Azure subscription (you can get a [free Azure account](https://azure.microsoft.com/free/) if you don't have one).
- The [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) installed to create the Azure resources.
- A Bash shell to run the deployment commands. Note that you can use the [Azure Cloud Shell](https://learn.microsoft.com/azure/cloud-shell/overview) for this as well; it even has the Azure CLI pre-installed.
- Optional: if you're using [Visual Studio Code](https://code.visualstudio.com/), you can use the [Azure CLI Tools extension](https://github.com/microsoft/vscode-azurecli) to light up additional features for `.azcli` files.

### Quickstart

In Bash, go through the commands in the [`deploy.azcli`](deploy.azcli) file to deploy and configure everything. You only have to change the first few lines to set the Azure region and resource group name.

In order to create the index, data sources and indexers in Azure Cognitive Search, this sample uses a few template files. The most relevant files for this sample are:

- [`deploy-index.json`](deploy-index.json) defines the structure of the search index.
  - It includes the typical information from blob storage (the content as well as file name, full path, file size, etc.).
  - It also defines the additional fields for the metadata values (`author`, `document_type` and `business_impact`).
- [`deploy-blob-indexer.json`](deploy-blob-indexer.json) defines the indexer for Blob storage.
  - The `dataToExtract` is set to `contentAndMetadata` so that metadata from blobs is included in the search index.
  - The `metadata_storage_path` (which is used as the document key by default) is base64-encoded to ensure legal key names.
- [`deploy-table-indexer.json`](deploy-table-indexer.json) defines the indexer for Table storage.
  - This uses the same `targetIndexName` as the Blob indexer so that the same destination index is used by both indexers.
  - It maps the `metadata_storage_path` column in Table storage to the `metadata_storage_path` field in the index; it's also base64-encoded to make sure that it refers to the same document key as the Blob indexer.

## Demo

The final line in the [`deploy.azcli`](deploy.azcli) file performs a search against the index. After a few seconds, this should start returning a JSON structure with the most interesting information for a single document: the search performs a `$top=1` to limit the results and `$select` to only return the file name and the relevant metadata values.

The important outcome here is that a *single* document will contain both the `author` metadata (coming from Blob storage) as well as the `document_type` and `business_impact` values (coming from Table storage).

You can also interactively search for documents in the Azure Portal by using the [Search explorer](https://learn.microsoft.com/azure/search/search-explorer#start-search-explorer).

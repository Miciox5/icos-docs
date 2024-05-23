# Quickstart

This quickstart shows how to run RAG application. It allows you to familiarize yourself with the workflow and gain a basic understanding of the lifecycle. Specifically, you will perform the following tasks in this tutorial:

- Serve application with Docker
- Manage your local data
- Ask the application about your data
- Trace your application with LangSmith


## Run services with Docker
[Docker](https://www.docker.com/) simplifies application deployment by packaging software and its dependencies into containers, ensuring consistent operation across different environments. It enables developers to build, ship, and run applications seamlessly across various platforms.

### Available services

=== "icos-mongodb"
    ```shell
    cd services/
    docker compose up icos-mongodb  -d
    ```
    This service provides a [MongoDB](https://www.mongodb.com/) database environment. Data stored within this ```MongoDB``` instance is mapped to a local data directory, likely for persistence. It serves as a backend data store for applications that require a NoSQL database solution.

=== "mongo-express"
    ```shell
    cd services/
    docker compose up mongo-express  -d
    ```
    This service is a web-based [MongoDB administration interface](https://github.com/mongo-express/mongo-express). It is connected to the icos-mongodb service, likely to provide a graphical user interface (GUI) for managing MongoDB databases, collections, and documents. It facilitates easier interaction and management of MongoDB data without directly using command-line tools.

=== "langfuse-server"
    ```shell
    cd services/
    docker compose up langfuse-server  -d
    ```
    [LangFuse](https://langfuse.com/) service is use for traces, evals, prompt management and metrics to debug and improve your LLM RAG application.

=== "redis-stack-chat-histories"
    ```shell
    cd services/
    docker compose up redis-stack-chat-histories  -d
    ```
    This service provides a [Redis](https://redis.io/) database environment for storing chat histories. ```Redis``` is often used as a fast, in-memory data store, commonly employed for caching, session management, and real-time data processing.

=== "openllm"
    ```shell
    cd services/
    docker compose up openllm  -d
    ```
    This service runs an [OpenLLM](https://github.com/bentoml/OpenLLM) (Language Model Server) service. It is configured with environment variables and GPU capabilities, used for serving large language models with support for GPU acceleration. Specific resource reservations and shared memory size are configured, optimized for running deep learning models. Additionally, it is mapped to volumes for cache and model storage, that serves language processing or machine learning tasks.

```shell title="Run all services"
cd services/
docker compose up -d
```

!!! note 
    Docker compose file was built with default persistence, it means that within the respective folders, database data is saved.


## Manage data
Introduces the concept of a [Single Source of Truth (SSOT)](#single-source-of-truth) for data ownership. It outlines uploading data and provides metadata formatting guidelines. Additionally, it explains the data ingestion process.

### Single source of truth

When a new data is uploaded, you should assign a Single Source of Truth (SSOT) to it.  
The SSOT is the owner of that data, and only the SSOT can modify or mutate it.  
The new data will be loaded into a `Mongo` database.

### Upload data

!!! info "Partial path tree showing the organization of the documents directory"
    ```
    ICOS
    └── documents
        └──source
            └── language
                ├── .loaded
                │   ├── doc_name.txt
                │   ├── ....
                │   └── doc_name.pdf
                ├── doc_name.pdf
                ├── doc_name.pdf
                └── metadata
                    └── document_metadata.json
    ```

!!! info "Supported file formats"
    - PDF
    - TXT

#### Metadata

Metadata should be uploaded to the metadata directory.  

!!! info "Json file of the metadata should reflect this formatting"
    ```
    {
        "document_name.txt": {
            "attr": "value",
            "attr": "value"
        },
        "document_name_2.txt": {
            "attr": "value",
            "attr": "value"
        },
        ....
    }
    ```

The data must be uploaded to the appropriate Documents folder, as shown just above in the [Partial path tree](#upload-data).

To upload data to the server run the [scp](https://man7.org/linux/man-pages/man1/scp.1.html) command:

```shell title="command syntax"
scp [OPTION] [user@]SRC_HOST:]file1 [user@]DEST_HOST:]file2
```

After uploading data to server, how explain in section [Single Source of Truth (SSOT)](#single-source-of-truth) the data must be uploaded into a `Mongo` database.

Upload data into a `Mongo` database:

```shell
python load_documents.py
```

!!! note 
    Once the data is uploaded to Mongo it is moved to the subdirectory called `.loaded.`   
    This `.loaded` directory is useful for two reasons:

        - Caching data already uploaded on Mongo
        - Persistence of uploaded data to server


### Ingest data
It loads documents from a specified source, processes them, extracts features using embeddings, and ingests them into a vector store.

Ingest data:

```shell
python ingest.py
```

## Query your data
The core of the application resides in its chain which is exposed by the [LangServe](https://python.langchain.com/v0.1/docs/langserve/) service.

### Expose the chain
```LangServe``` sets up a [FastAPI](https://fastapi.tiangolo.com/) server. It loads various components such as embeddings, cross-encoder model, and callbacks. It then constructs a processing chain for language understanding and dialogue management. Finally, it adds routes for handling API requests related to chat functionality and runs the server using [uvicorn](https://www.uvicorn.org/) on ```localhost:8000```.

Expose the chain:
```shell
python run_server.py
```

### Run client
The client allows you to contact ```LangServe``` which provides the answer and associated context documents extracted from the chain.

Run the following command:
```shell
python run_client.py
```

## Trace application

### LangSmith
[LangSmith](https://smith.langchain.com/) is a platform for building production-grade LLM applications. It allows you to closely monitor and evaluate your application.

A ```Trace``` is a collection of runs that are related to a single operation. For example, if you have a user request that triggers a chain, and that chain makes a call to an ```LLM```, then to an output parser, and so on, all of these runs would be part of the same trace.



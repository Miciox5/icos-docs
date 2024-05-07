# Quickstart

This quickstart shows how to run RAG application. It allows you to familiarize yourself with the workflow and gain a basic understanding of the lifecycle. Specifically, you will perform the following tasks in this tutorial:

- Serve application with Docker
- Manage your local data
- Ask the application about your data
- Trace your application with LangSmith


## Run services with Docker

Run the follow command to start all services. 

```shell
cd services/
docker compose up -d
cd ..
```

!!! note 
    Docker compose file was built with default persistence, it means that within the respective folders, database data is saved.

## Upload your document(s)

### Single source of truth

When a new data is uploaded, you should assign a Single Source of Truth (SSOT) to it.  
The SSOT is the owner of that data, and only the SSOT can modify or mutate it.  
The new data will be uploaded to the `documents` folder.

### Supported file formats:

- PDF
- TXT

Run the following command to upload your document(s):

```shell
python uploader.py
```

Use help for more details.

```sh
python uploader.py --help
```

## Run auxiliary services

Run the follow command to start all services required by project. 
Before you run this command, consult ports in the next section and free them, please.

```shell
cd services/
docker compose up -d
cd ..
```

### List of services

List of services in docker-compose.yml:
- **icos-mongodb**: Mongo database in which documents are saved; 
  - http://localhost:27017
- **mongo-express**: Mongo UI for consulting the Mongo database locally;
  - http://localhost:8081  
- **postgres-langfuse**: Postgresql database needed for langfuse server.
  - http://localhost:5432  # No UI
- **langfuse-server**: Langfuse Server UI to consult logs and evaluations from the model.
  - http://localhost:3000 

>_NOTE_: Docker compose file was built with default persistence. 
> It means that within the respective folders, database data is saved.

### Integrate a remote database

In addition to maintaining the data locally, it is possible to integrate a remote database, and this allows us to  
get more description of the existing data through the integration of associated metadata.  
The application provides a `Mongo` database.
Check first if your locally port of mongodb (**27017**) is already occupied.
You can run Mongo database with docker compose instructions upstairs.

### Langfuse

Langfuse is an open source LLM engineering platform to help teams collaboratively debug, analyze and iterate on 
their LLM Applications. For more information, visit https://langfuse.com/docs.

### Insert data into database

To insert documents into the database make sure they exist locally, in the `documents` folder (SSOT).

```shell
python mongo.py
cd ..
```

Use help for more details.

```sh
python mongo.py --help
```

## Ingest Data

If it not exists, this will create a new folder called `chroma`, and use it for the newly created vector store.  
You can ingest as many documents as you want.

### Note

- When run this for the first time, it will need internet access to download the embedding model (
  default: `all-MiniLM-L6-v2`).
- By default, it will run on `cuda` device.
- For Mac user specify `mps` device.

### Ingest from local

Run the following command to ingest the data present into `documents` folder.

```sh
python ingest.py
```

### Ingest from remote database

Run the following command to ingest the data from remote database.

```sh
python ingest.py --remote
```

Use help for more details.

```sh
python ingest.py --help
```


## Ask questions locally

Configuration: before you run this command, configure your ```.env``` with your data:
```shell
cp .env .env.example
```
After that, insert in ```.env``` file your data about LangSmith/Langfuse service.

In order to chat with your documents, run the following command:

```shell
python chat.py
```

This will load the ingested vector store and embedding model. You will be presented with a prompt:

```
> Enter a query: 
```

After typing your question, hit enter.  
The LLM will take some time based on your hardware.
Once the answer is generated, you can then ask another question without re-running the script, just wait for the prompt
again.

### Note

- When you run this for the first time, it will need to download the LLM (default: `TheBloke/Llama-2-7b-Chat-GGUF`).
- By default, it will run on `cuda` device.
- For Mac user specify `mps` device.
- Type `exit` to finish the script.

### Extra Options

- Use `--show_sources` flag to show which chunks were retrieved by the embedding model.
- Use `--save_logs` flag to save the log to LangSmith platform.

# How to select different text splitter, embeddings and LLM models?

## Change the text splitter parameter

By default, the class `Chunker()` instantiate a text splitter with these parameters:

```
chunk_size=1024,
chunk_overlap=128,
length_function=len,
is_separator_regex=False
```

1. Open `ingest.py` in the editor of your choice.
2. Pass the custom parameters to `Chunker()` class.

## Change the embedding model

Open `constants.py` in the editor of your choice to see a list of available model.

1. Open `main.py` in the editor of your choice.
2. Pass chosen model to the class `EmbeddingLoader(..., model_name=chosen_model)`.

## Change the LLM model and parameter

Open `constants.py` in the editor of your choice to see a list of available model.

By default, the class `ModelLoader` instantiate a model with these parameters:

```py linenums="1" hl_lines="2 3" title="bubble_sort.py"
ModelLoader(
    model_id="TheBloke/Llama-2-7b-Chat-GGUF",
    model_basename="llama-2-7b-chat.Q4_K_M.gguf",
    context_window_size=2048,
    max_new_tokens=2048,
    n_batch=512,
    n_gpu_layers=1
)
```

1. Open `main.py` in the editor of your choice.
2. Pass your custom parameters to `ModelLoader(...)` class.

# Common errors

- If the ingest.py stuck, yuo should change the embedding model to a smaller one. 
- If you get a ***"not enough space in the buffer"*** error, you should:
  1. Open up `constants.py` in the editor of your choice.
  2. Reduce the values `CONTEXT_WINDOW_SIZE`, start with half of the original values and keep halving the value until the
     error stops appearing.
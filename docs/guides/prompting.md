This guide provide an overview of prompts and usage examples. The guide also includes tips, applications, limitations.

## Prompt Engineering Guide for Mixtral 8x7B

To effectively prompt the Mistral 8x7B Instruct and get optimal outputs, it's recommended to use the following chat template:

```
<s>[INST] Instruction [/INST] Model answer</s>[INST] Follow-up instruction [/INST]
```

Note that `<s>` and `</s>` are special tokens for beginning of string *(BOS)* and end of string *(EOS)* while *[INST]* and *[/INST]* are regular strings.


### Prompt types
The application has multiple prompts divided according to the task to be performed by the model.
Main tasks that the model executes during chain execution are as follows:

- [Standalone Question](#standalone-question)
- [Self Query](#self-query)
- [Question Answer](#question-answer)
- [Retry Output Parser](#retry-output-parser)
- [Chat history summarization](#chat-history-summarization)

#### Standalone Question
A standalone question in a RAG system improves context clarity and information retrieval, leading to more precise and relevant responses. It reduces ambiguity and enhances multi-turn dialogue performance. This is achieved by defining a sub-chain that reformulates the latest user question using historical messages if needed.

``` title="Prompt example"
[INST]
Given a chat history and the latest user question 
which might reference context in the chat history,
formulate a standalone question which can be understood
without the chat history. Do NOT answer the question,
just reformulate it if needed and otherwise return it as is.
Conversation history:
{history}
Question: {question}
Standalone question:
[/INST]
```

#### Self Query
A self-querying utilizes a query-constructing LLM chain to generate structured queries from natural language inputs. It applies these structured queries to its VectorStore, enabling it to perform semantic similarity comparisons and extract metadata filters from the user query to execute those filters on stored documents.

``` title="Prompt example"
[INST]
Your goal is to structure the user's query to match the request schema provided below.

<< Structured Request Schema >>
When responding use a markdown code snippet with a JSON object formatted in the following schema:

```json
{
    "query": string \ text string to compare to document contents
    "filter": string \ logical condition statement for filtering documents
}

Produce only a JSON object and avoid explaining or NOTE.
The query string should contain only text that is expected to match the contents of documents.
Any conditions in the filter should not be mentioned in the query as well.
A logical condition statement is composed of one or more comparison and logical operation statements.

A comparison statement takes the form: `comp(attr, val)`:
- `comp` ({allowed_comparators}): comparator
- `attr` (string):  name of attribute to apply the comparison to
- `val` (string): is the comparison value

A logical operation statement takes the form `op(statement1, statement2, ...)`:
- `op` ({allowed_operators}): logical operator
- `statement1`, `statement2`, ... (comparison statements or logical operation statements): one or more statements to apply the operation to

Make sure that you only use the comparators and logical operators listed above and no others.
Make sure that filters only refer to attributes that exist in the data source.
Make sure that filters only use the attributed names with its function names if there are functions applied on them.
Make sure that filters only use format `YYYY-MM-DD` when handling date data typed values.
Make sure that filters take into account the descriptions of attributes and only make comparisons that are feasible given the type of data being stored.
Make sure that filters are only used as needed. If there are no filters that should be applied return "NO_FILTER" for the filter value.

<< Data Source >>
```json
{{{{
    "content": "{content}",
    "attributes": {attributes}
}}}}

<< Example {i}. >>
User Query:
{user_query}

Structured Request:
```json
{structured_request}

<< Example {i}. >>
User Query:
{{query}}

Structured Request:
[/INST]
```

#### Question Answer
LLMs can reason about wide-ranging topics, but their knowledge is limited to the public data up to a specific point in time that they were trained on. If you want to build AI applications that can reason about private data or data introduced after a model's cutoff date, you need to augment the knowledge of the model with the specific information it needs. 

```title="Prompt example"
[INST]
You are an assistant for question-answering tasks. Use the following pieces of retrieved
context to answer the question. If you don't know the answer, just say that you don't know.
Use three sentences maximum and keep the answer concise.
Context: {context}
Question: {question}
Answer:
[/INST]
```

#### Chat history summarization
Chat history summarization condenses previous interactions into a brief summary, capturing key points and context. This ensures the system can provide accurate and relevant responses in ongoing conversations, enhancing the overall user experience by maintaining coherence and avoiding information overload.

``` title="Prompt example"
[INST]
Progressively summarize the lines of conversation provided, adding onto the previous summary returning a new summary.

EXAMPLE
Current summary:
The human asks what the AI thinks of artificial intelligence. The AI thinks artificial intelligence is a force for good.

New lines of conversation:
Human: Why do you think artificial intelligence is a force for good?
AI: Because artificial intelligence will help humans reach their full potential.

New summary:
The human asks what the AI thinks of artificial intelligence. The AI thinks artificial intelligence is a force for good because it will help humans reach their full potential.
END OF EXAMPLE

Current summary:
{summary}

New lines of conversation:
{new_lines}

New summary:
[/INST]
```

#### Retry Output Parser
The retry output parser corrects errors in initial responses by analyzing the flaws and generating a revised output. It uses a debugging prompt template that includes the original prompt, flawed completion, and error details, ensuring the final response meets specified constraints and is accurate and consistent.

``` title="Prompt example"
<s>
Prompt:
{prompt}
Completion:
{completion}
</s>
[INST]
Above, the Completion did not satisfy the constraints given in the Prompt.
Details: {error}
Please try again:
[/INST]
```

## Custom prompt
Creating a custom prompt for a specific domain involves defining the system's behavior, instructions, metadata, and examples tailored to that domain. Here's a step-by-step guide to creating and using a custom prompt for a *tourism assistant*.

!!! info "Path tree showing the prompts package"
    ```
    .
    ├── domains
    │   └── tourism.py
    ├── qa_prompt.py
    ├── self_query_prompt.py
    ├── summarizer_prompt.py
    └── standalone_prompt.py
    ```


### Steps to Create a Custom Prompt

1.  **Define the System Prompt**

    The system prompt sets the context and role of the assistant. For a tourism assistant, you might define it as follows:
    
    ```
    TOURISM_ASSISTANT_SYSTEM_PROMPT = """
        You are a helpful tour assistant.
        You will use only the provided context to answer tourist questions.
        Read the given context before answering questions.
        If you cannot answer a tourist question based on the provided context, inform the tourist.
    """
    ```
2.  **Define the System Instructions**

    The instructions guide the assistant on how to use the context to answer questions. It includes placeholders for the context and the tourist's question:

    ```
    TOURISM_ASSISTANT_SYSTEM_INSTRUCTION = """
        Context:
        {context}
        Tourist question: {question}
        Answer:
    """
    ```

3.  **Define Metadata Fields**

    Metadata fields provide additional attributes that help filter and query documents more effectively. Each attribute is defined with a name, description, and type.

    !!! note
        TOURISM_METADATA_FIELD_INFO list provide a description of metadata related to uploaded documents.  
        Read more here [Manage data](../get-started/quickstart.md#manage-data).

    ```py
    TOURISM_METADATA_FIELD_INFO = [
        AttributeInfo(
            name="document_name",
            description="The name of the document",
            type="string",
        ),
        AttributeInfo(
            name="source",
            description="The source of the document",
            type="string",
        ),
        AttributeInfo(
            name="language",
            description="The language of the document. One of ['it', 'en']",
            type="string",
        ),
        AttributeInfo(
            name="name",
            description="The name of the point of interest",
            type="string",
        ),
        AttributeInfo(
            name="city",
            description="The city where the point of interest is located",
            type="string",
        ),
        AttributeInfo(
            name="address",
            description="The address of the point of interest",
            type="string",
        ),
        AttributeInfo(
            name="type",
            description="The type of the point of interest",
            type="string",
        )
    ]
    ```

4.  **Describe Document Content**

    Provide a description of the type of content stored in the documents:

    ```
    TOURISM_DOCUMENT_CONTENT_DESCRIPTION = "Information related to points of interest in a city"
    ```

5.  **Provide Examples**

    Examples help the [Self Query](#self-query) understand how to interpret and respond to different types of queries. Each example consists of a question, a query, and a filter:

    !!! tip
        Provide as many examples as possible and try generating with an artificial intelligence such as ChatGPT.

    ```py
    TOURISM_EXAMPLES = [
        ("What is the best gelato flavor in Florence?",
            {
                "query": "best gelato flavor",
                "filter": "NO_FILTER"
            }
        ),
        ("What are some popular tourist attractions in Rome?",
            {
                "query": "tourist attractions",
                "filter": "eq(\"city\", \"Rome\")"
            }
        ),
        ("Where can I find authentic Italian pizza in Naples?",
            {
                "query": "authentic Italian pizza",
                "filter": "and(eq(\"city\", \"Naples\"), eq(\"type\", \"restaurant\"))"
            }
        ),
        ("Show me museums in Paris or Berlin with at least 4 stars.",
            {
                "query": "museums",
                "filter": "and(or(eq(\"city\", \"Paris\"), eq(\"city\", \"Berlin\")), gte(\"stars\", \"4\"))"
            }
        ),
        ("I'm looking for a romantic gondola ride followed by a traditional seafood dinner in Venice.",
            {
                "query": "romantic gondola ride and seafood dinner",
                "filter": "and(eq(\"city\", \"Venice\"), or(and(eq(\"type\", \"gondola ride\")), and(eq(\"type\", \"restaurant\"), eq(\"cuisine\", \"seafood\"), eq(\"ambiance\", \"romantic\"))))"
            }
        )
    ]
    ```

### Implement custom domain prompt
To use the custom domain prompt for a tourism assistant, you need to integrate the tourism-specific prompts and metadata into your system. This involves two main files `qa_prompt.py` and `self_query_prompt.py`.

```py title="qa_prompt.py"
from prompts.domains.tourism import (
    TOURISM_ASSISTANT_SYSTEM_PROMPT,
    TOURISM_ASSISTANT_SYSTEM_INSTRUCTION
)

BOS, EOS = '<s>', '</s>'
B_INST, E_INST = "[INST]", "[/INST]"
B_SYS, E_SYS = "<<SYS>>\n", "\n<</SYS>>\n\n"

LLAMA_QA_PROMPT_TEMPLATE = PromptTemplate.from_template(
    template=B_INST + B_SYS + TOURISM_ASSISTANT_SYSTEM_PROMPT + E_SYS + TOURISM_ASSISTANT_SYSTEM_INSTRUCTION + E_INST
)


MIXTRAL_QA_PROMPT_TEMPLATE = PromptTemplate.from_template(
    template=B_INST + TOURISM_ASSISTANT_SYSTEM_PROMPT + TOURISM_ASSISTANT_SYSTEM_INSTRUCTION + E_INST
)
```

```py title="self_query_prompt.py"
from prompts.domains.tourism import (
    TOURISM_EXAMPLES,
    TOURISM_METADATA_FIELD_INFO,
    TOURISM_DOCUMENT_CONTENT_DESCRIPTION
)

EXAMPLES = TOURISM_EXAMPLES

METADATA_FIELD_INFO = TOURISM_METADATA_FIELD_INFO

DOCUMENT_CONTENT_DESCRIPTION = TOURISM_DOCUMENT_CONTENT_DESCRIPTION
```



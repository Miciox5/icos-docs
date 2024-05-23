This guide provide an overview of prompts and usage examples. The guide also includes tips, applications, limitations.

## Prompt Engineering Guide for Mixtral 8x7B

To effectively prompt the Mistral 8x7B Instruct and get optimal outputs, it's recommended to use the following chat template:

```
<s>[INST] Instruction [/INST] Model answer</s>[INST] Follow-up instruction [/INST]
```

Note that `<s>` and `</s>` are special tokens for beginning of string *(BOS)* and end of string *(EOS)* while *[INST]* and *[/INST]* are regular strings.


### Task Prompting
The application has multiple prompts divided according to the task to be performed by the model.
Currently, the tasks that the model executes during chain execution are as follows:

- [Standalone Question](#standalone-question)
- [Self Query](#self-query)
- [Question Answer](#question-answer)


#### Standalone Question
Let's start with a query-answer tourism prompt example and instruct the model to achieve a task based on an instruction.

Prompt:
```
[INST]
You are an helpful tour assistant.
You will use only the provided context to answer tourist questions.
Read the given context before answering questions.
If you can not answer a tourist question based on the provided context, inform the tourist.
Context: {context}
Tourist question: {question}
Answer:
[/INST]
```

#### Self Query


#### Question Answer






# Trustwise Safety Plugin Starter Tutorial 🦉
In this starter example, we will show how to log and monitor LLM evaluation metrics using Trustwise Safety Plugin.

## What is Trustwise Safety Plugin? 🦉

Trustwise Safety Plugin is a plugin designed to help you evaluate your Large Language Models (LLMs) RAG pipelines. RAG pipelines can be built using various tools like LlamaIndex, LangChain etc. In this example, we will show how to use this plugin with LlamaIndex.

## Setup
```
pip install trustwise
```

## Prerequisites
To use Safety Plugin, you should already be familiar with the basics of the LlamaIndex including loading documents, parsing them into nodes, constructing an index, and querying the index to get responses.

## Usage pattern

## Setup your OpenAI API key

```python
import os
os.environ['OPENAI_API_KEY']='OPENAI_API_KEY'
```

### Configure the Callback Handler for Trustwise 🦉

Generate Trustwise API key by going to this [link](http://api.trustwise.ai:8080/github-login)

```python
from trustwise.callback import TWCallbackHandler

# Initialize Trustwise CallbackHandler
tw_callback = TWCallbackHandler()

# Enter Trustwise API Key
tw_api_key = 'TRUSTWISE_API_KEY'

# Set the API key using the set_api_key method
tw_callback.set_api_key(tw_api_key)
```

```python

# Necessary LlamaIndex imports
from llama_index.callbacks import CallbackManager, LlamaDebugHandler
from llama_index.llms import OpenAI
from llama_index import VectorStoreIndex, ServiceContext, SimpleDirectoryReader

# Initialize LlamaDebugHandler object
llama_debug = LlamaDebugHandler(print_trace_on_end=True)

# Include Trustwise CallbackHandler in the LlamaIndex callback manager
tw_callback_manager = CallbackManager([llama_debug, tw_callback])

# Initialize your LLM
# You can use any LLM of your choice
gpt4_llm = OpenAI(model="gpt-4", temperature=0)

# Default service context from LlamaIndex
service_context = ServiceContext.from_defaults(
    callback_manager=tw_callback_manager, llm=gpt4_llm
)
```

### Parse documents into an Index
We have provided a `data` directory with sample PDF documents, feel free to change the code below to include your own documents and directories.
```python
# Load your documents in a directory called `data`
documents = SimpleDirectoryReader('data').load_data() 

vec_index = VectorStoreIndex.from_documents(documents, service_context=service_context)
```

### Sample Query and Responses
```python
query_engine = vec_index.as_query_engine()

# Change the query below
query = "What is VTE?"

response = query_engine.query(query)
print(response)

# Output: VTE, or Venous Thromboembolism, is a condition that can occur in individuals ...
```

### Observe and evaluate using Trustwise 🦉
```python
from trustwise.functions import Observability

observe = Observability()

observe.set_api_key(tw_api_key)

results = observe.evaluate(query, response)
results
```
<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Faithfulness</th>
      <th>Answer Relevancy</th>
      <th>Harmfulness</th>
      <th>Context Precision</th>
      <th>Trustwise Safety Score </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <th>100</th>
      <th>94</th>
      <th>0</th>
      <th>58</th>
      <th>24</th>
    </tr>
  </tbody>
</table>
</div>

## What do these metrics mean?

1. **Faithfulness**: This measures the factual consistency of the generated answer against the given context. It is calculated from answer and retrieved context. The answer is scaled to (0,100) range. Higher the better.

2. **Answer relevancy**: This focuses on assessing how pertinent the generated answer is to the given prompt. A lower score is assigned to answers that are incomplete or contain redundant information. This metric is computed using the question and the answer, with values ranging between 0 and 100, where higher scores indicate better relevancy.

3. **Harmfulness**: Harmfulness is defined as whether the answer causes or has the potential to cause harm to individuals, groups, or society at large. It is important to note that harmfulness score should not contribute to a aggregated final score due to their binary nature.

4. **Context precision**: This metric gauges the precision of the retrieved context, calculated based on both the question and contexts. The values fall within the range of (0, 100), with higher values indicating better precision.

For more details on these metrics, please refer to [Ragas documentation](https://docs.ragas.io/en/latest/concepts/metrics/index.html).

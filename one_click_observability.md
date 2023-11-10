### Trustwise Safety Plugin 🦉

Trustwise Safety Plugin is a plugin designed to help you evaluate your Large Language Models (LLMs) RAG pipelines. RAG pipelines can be built using various tools like LlamaIndex, LangChain etc. In this example, we will show how to use this plugin with LlamaIndex.

#### Usage Pattern

```python

###### Setup Trustwise callback handler ########
from trustwise.callback import TWCallbackHandler
from llama_index.callbacks import CallbackManager, LlamaDebugHandler

# Initialize Trustwise CallbackHandler
tw_callback = TWCallbackHandler()

# Enter Trustwise API Key
tw_api_key = 'TRUSTWISE_API_KEY'

# Set the API key using the set_api_key method
tw_callback.set_api_key(tw_api_key)

# Initialize LlamaDebugHandler object
llama_debug = LlamaDebugHandler(print_trace_on_end=True)

# Include Trustwise CallbackHandler in the LlamaIndex callback manager
tw_callback_manager = CallbackManager([llama_debug, tw_callback])

# Rest of the llamaindex code like indexing, llm response generation comes here

###### Evaluate LLM responses #######

from trustwise.functions import Observability

observe = Observability()

observe.set_api_key(tw_api_key)

results = observe.evaluate(query, response)

```

#### Guides

```{toctree}
---
maxdepth: 1
---
/examples/callbacks/trustwise_hop.md
Colab <>
```
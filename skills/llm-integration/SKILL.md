---
mode: auto
name: llm-integration
description: LLM API integration patterns for OpenAI, Anthropic, and local models
tags: [llm, ai, openai, anthropic, langchain, api]
version: 1.0.0
---

# LLM Integration

**Author**: <AUTHOR_NAME>  
**Date**: 2026-04-06

Expert knowledge for integrating Large Language Models (LLMs) into applications, including OpenAI GPT-4, Anthropic Claude, and local models.

## Technology Stack

- **LLM APIs**: OpenAI, Anthropic Claude, Cohere
- **Local Models**: Llama 2, Mistral, Phi-2
- **Frameworks**: LangChain, LlamaIndex, Semantic Kernel
- **Vector DBs**: Pinecone, Weaviate, ChromaDB
- **Python**: 3.11+

## OpenAI Integration

### Basic Chat Completion
```python
# @author <AUTHOR_NAME>
from openai import OpenAI

client = OpenAI(api_key=os.environ["OPENAI_API_KEY"])

def chat_completion(prompt: str, model="gpt-4") -> str:
    """Generate chat completion."""
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": "You are a storage interface expert."},
            {"role": "user", "content": prompt}
        ],
        temperature=0.7,
        max_tokens=1000
    )
    
    return response.choices[0].message.content
```

### Streaming Responses
```python
# @author <AUTHOR_NAME>
def stream_completion(prompt: str):
    """Stream LLM response."""
    stream = client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}],
        stream=True
    )
    
    for chunk in stream:
        if chunk.choices[0].delta.content:
            yield chunk.choices[0].delta.content
```

### Function Calling
```python
# @author <AUTHOR_NAME>
functions = [{
    "name": "get_volume_status",
    "description": "Get Storage Interface volume status",
    "parameters": {
        "type": "object",
        "properties": {
            "volume_id": {"type": "string"}
        },
        "required": ["volume_id"]
    }
}]

def chat_with_functions(prompt: str) -> dict:
    """Use function calling."""
    response = client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}],
        functions=functions,
        function_call="auto"
    )
    
    return response.choices[0].message.function_call
```

## Anthropic Claude Integration

### Claude Messages API
```python
# @author <AUTHOR_NAME>
from anthropic import Anthropic

client = Anthropic(api_key=os.environ["ANTHROPIC_API_KEY"])

def claude_completion(prompt: str) -> str:
    """Generate completion with Claude."""
    message = client.messages.create(
        model="claude-3-sonnet-20240229",
        max_tokens=1024,
        temperature=0.7,
        messages=[{
            "role": "user",
            "content": prompt
        }]
    )
    
    return message.content[0].text
```

## LangChain Integration

### Chain Construction
```python
# @author <AUTHOR_NAME>
from langchain.chat_models import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.schema.output_parser import StrOutputParser

def create_code_gen_chain():
    """Create code generation chain."""
    llm = ChatOpenAI(model="gpt-4", temperature=0.2)
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", "Generate Go code for storage interfaces. Follow project standards."),
        ("user", "{request}")
    ])
    
    chain = prompt | llm | StrOutputParser()
    return chain

# Usage
chain = create_code_gen_chain()
code = chain.invoke({"request": "Create volume validation function"})
```

### RAG (Retrieval Augmented Generation)
```python
# @author <AUTHOR_NAME>
from langchain.vectorstores import Chroma
from langchain.embeddings import OpenAIEmbeddings
from langchain.chains import RetrievalQA

def create_rag_chain(docs_path: str):
    """Create RAG chain for codebase Q&A."""
    embeddings = OpenAIEmbeddings()
    vectorstore = Chroma(
        persist_directory=docs_path,
        embedding_function=embeddings
    )
    
    llm = ChatOpenAI(model="gpt-4")
    qa_chain = RetrievalQA.from_chain_type(
        llm=llm,
        chain_type="stuff",
        retriever=vectorstore.as_retriever(search_kwargs={"k": 5})
    )
    
    return qa_chain
```

## Local Model Integration

### Llama.cpp with Python
```python
# @author <AUTHOR_NAME>
from llama_cpp import Llama

def load_local_model(model_path: str):
    """Load local Llama model."""
    llm = Llama(
        model_path=model_path,
        n_ctx=2048,
        n_threads=8
    )
    return llm

def generate_local(llm, prompt: str) -> str:
    """Generate with local model."""
    output = llm(
        prompt,
        max_tokens=500,
        temperature=0.7,
        stop=["###"]
    )
    return output['choices'][0]['text']
```

### Ollama Integration
```python
# @author <AUTHOR_NAME>
import requests

def ollama_generate(prompt: str, model="llama2") -> str:
    """Generate using Ollama."""
    response = requests.post(
        "http://localhost:11434/api/generate",
        json={
            "model": model,
            "prompt": prompt,
            "stream": False
        }
    )
    return response.json()["response"]
```

## Cost Optimization

### Token Counting
```python
# @author <AUTHOR_NAME>
import tiktoken

def count_tokens(text: str, model="gpt-4") -> int:
    """Count tokens for cost estimation."""
    encoding = tiktoken.encoding_for_model(model)
    return len(encoding.encode(text))

def estimate_cost(prompt: str, completion: str) -> float:
    """Estimate API cost."""
    input_tokens = count_tokens(prompt)
    output_tokens = count_tokens(completion)
    
    # GPT-4 pricing (example)
    input_cost = (input_tokens / 1000) * 0.03
    output_cost = (output_tokens / 1000) * 0.06
    
    return input_cost + output_cost
```

### Caching Responses
```python
# @author <AUTHOR_NAME>
from functools import lru_cache
import hashlib

@lru_cache(maxsize=1000)
def cached_completion(prompt_hash: str, model: str) -> str:
    """Cache LLM responses."""
    return chat_completion(prompt_hash, model)

def get_cached_response(prompt: str, model="gpt-4") -> str:
    """Get cached or new response."""
    prompt_hash = hashlib.md5(prompt.encode()).hexdigest()
    return cached_completion(prompt_hash, model)
```

## Best Practices

### Error Handling
```python
# @author <AUTHOR_NAME>
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=4, max=10)
)
def robust_completion(prompt: str) -> str:
    """LLM call with retries."""
    try:
        return chat_completion(prompt)
    except Exception as e:
        logger.error(f"LLM error: {e}")
        raise
```

## Reference Files

- **AI Code Assistant**: `.ai-config/agents/ai-code-assistant.md`
- **Prompt Engineering**: `.ai-config/skills/prompt-engineering/SKILL.md`
- **LLM Best Practices**: `.ai-config/rules/llm-best-practices.md`

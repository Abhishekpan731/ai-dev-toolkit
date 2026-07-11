---
mode: auto
name: prompt-engineering
description: Effective prompt design and optimization for LLMs
tags: [llm, prompts, optimization, ai, gpt-4, claude]
version: 1.0.0
---

# Prompt Engineering

**Author**: <AUTHOR_NAME>  
**Date**: 2026-04-06

Expert knowledge for designing effective prompts, optimizing LLM outputs, and building reliable AI-powered applications.

## Technology Stack

- **LLMs**: GPT-4, Claude 3, Llama 2
- **Tools**: LangChain, Guidance, DSPy
- **Evaluation**: ROUGE, BLEU, semantic similarity
- **Python**: 3.11+

## Prompt Design Patterns

### Zero-Shot Prompting
```python
# @author <AUTHOR_NAME>
def zero_shot_prompt(task: str) -> str:
    """Basic zero-shot prompt."""
    return f"""Task: {task}

Generate Go code following these rules:
- Functions <50 lines
- Add @author <AUTHOR_NAME>
- Use error wrapping with %w
- Follow storage interface patterns

Output:"""
```

### Few-Shot Prompting
```python
# @author <AUTHOR_NAME>
def few_shot_prompt(task: str, examples: list) -> str:
    """Few-shot learning with examples."""
    examples_text = "\n\n".join([
        f"Example {i+1}:\nInput: {ex['input']}\nOutput: {ex['output']}"
        for i, ex in enumerate(examples)
    ])
    
    return f"""{examples_text}

Now generate for:
Input: {task}
Output:"""
```

### Chain-of-Thought (CoT)
```python
# @author <AUTHOR_NAME>
def chain_of_thought_prompt(problem: str) -> str:
    """Encourage step-by-step reasoning."""
    return f"""Problem: {problem}

Let's solve this step by step:
1. First, identify the requirements
2. Then, design the approach
3. Next, implement the solution
4. Finally, verify correctness

Step 1:"""
```

## Role-Based Prompting

### System Prompts
```python
# @author <AUTHOR_NAME>
SYSTEM_PROMPTS = {
    "code_generator": """You are an expert Go developer for storage interfaces.
    
Rules:
- All functions must be <50 lines
- Add @author <AUTHOR_NAME> to new code
- Use fmt.Errorf with %w for error wrapping
- Follow CSI specification v1.8.0
- Include comprehensive error handling""",
    
    "code_reviewer": """You are a senior code reviewer.
    
Focus on:
- Function length (must be <50 lines)
- Error handling patterns
- Security vulnerabilities
- Performance issues
- Test coverage""",
    
    "documentation_writer": """You are a technical documentation expert.
    
Generate:
- Clear, concise explanations
- Code examples with syntax highlighting
- Cross-references to related sections
- Practical usage examples"""
}
```

## Structured Output

### JSON Mode
```python
# @author <AUTHOR_NAME>
from openai import OpenAI

client = OpenAI()

def structured_output(prompt: str) -> dict:
    """Get structured JSON output."""
    response = client.chat.completions.create(
        model="gpt-4-1106-preview",
        response_format={"type": "json_object"},
        messages=[{
            "role": "system",
            "content": "Return valid JSON only"
        }, {
            "role": "user",
            "content": f"""{prompt}

Return JSON with keys: function_name, parameters, return_type, code"""
        }]
    )
    
    return json.loads(response.choices[0].message.content)
```

### Pydantic Models
```python
# @author <AUTHOR_NAME>
from pydantic import BaseModel
from langchain.output_parsers import PydanticOutputParser

class CodeFunction(BaseModel):
    name: str
    description: str
    parameters: list[dict]
    return_type: str
    code: str

def parse_function_output(llm_output: str) -> CodeFunction:
    """Parse LLM output into structured model."""
    parser = PydanticOutputParser(pydantic_object=CodeFunction)
    return parser.parse(llm_output)
```

## Prompt Optimization

### A/B Testing Prompts
```python
# @author <AUTHOR_NAME>
from typing import Callable

def test_prompts(prompts: list[str], test_cases: list, 
                 evaluator: Callable) -> dict:
    """Compare prompt performance."""
    results = {}
    
    for i, prompt in enumerate(prompts):
        scores = []
        for case in test_cases:
            output = generate(prompt.format(**case))
            score = evaluator(output, case['expected'])
            scores.append(score)
        
        results[f"prompt_{i}"] = {
            "avg_score": sum(scores) / len(scores),
            "prompt": prompt
        }
    
    return sorted(results.items(), key=lambda x: x[1]['avg_score'], reverse=True)
```

### Automatic Prompt Optimization
```python
# @author <AUTHOR_NAME>
from dspy import ChainOfThought, evaluate

def optimize_prompt(task: str, examples: list):
    """Optimize prompt using DSPy."""
    # Define metric
    def accuracy_metric(example, pred):
        return example.answer == pred.answer
    
    # Compile optimizer
    optimizer = BootstrapFewShot(metric=accuracy_metric)
    
    # Optimize
    optimized = optimizer.compile(
        ChainOfThought("question -> answer"),
        trainset=examples
    )
    
    return optimized
```

## Prompt Templates

### Code Generation Template
```python
# @author <AUTHOR_NAME>
CODE_GEN_TEMPLATE = """Generate a Go function for storage interface:

Function Purpose: {purpose}
Input Parameters: {parameters}
Return Type: {return_type}

Context from codebase:
{context}

Requirements:
- Function length: <50 lines
- Add: @author <AUTHOR_NAME>
- Error handling: Use fmt.Errorf with %w
- Style: Follow existing codebase patterns

Generate complete function with:
1. Function signature
2. Input validation
3. Core logic
4. Error handling
5. Return statement
"""

def generate_function(purpose, parameters, return_type, context):
    """Generate function using template."""
    prompt = CODE_GEN_TEMPLATE.format(
        purpose=purpose,
        parameters=parameters,
        return_type=return_type,
        context=context
    )
    return llm.invoke(prompt)
```

## Safety and Constraints

### Content Filtering
```python
# @author <AUTHOR_NAME>
def safe_prompt(user_input: str) -> str:
    """Add safety constraints to prompt."""
    return f"""User Request: {user_input}

SAFETY CONSTRAINTS:
- No code that could harm systems
- No credential handling
- No external API calls without review
- Follow security best practices

Generate safe code that:"""
```

### Input Sanitization
```python
# @author <AUTHOR_NAME>
import re

def sanitize_prompt(prompt: str) -> str:
    """Remove prompt injection attempts."""
    # Remove common injection patterns
    dangerous_patterns = [
        r"ignore previous instructions",
        r"disregard all",
        r"system:\s*you are",
    ]
    
    for pattern in dangerous_patterns:
        prompt = re.sub(pattern, "", prompt, flags=re.IGNORECASE)
    
    return prompt
```

## Evaluation Metrics

### Semantic Similarity
```python
# @author <AUTHOR_NAME>
from sentence_transformers import SentenceTransformer, util

model = SentenceTransformer('all-MiniLM-L6-v2')

def evaluate_similarity(generated: str, expected: str) -> float:
    """Measure semantic similarity."""
    emb1 = model.encode(generated, convert_to_tensor=True)
    emb2 = model.encode(expected, convert_to_tensor=True)
    
    similarity = util.cos_sim(emb1, emb2)
    return float(similarity)
```

## Best Practices

### Prompt Checklist
```python
# @author <AUTHOR_NAME>
PROMPT_CHECKLIST = {
    "clarity": "Is the task clearly defined?",
    "context": "Is relevant context provided?",
    "examples": "Are examples included (if needed)?",
    "constraints": "Are output constraints specified?",
    "format": "Is output format specified?",
    "safety": "Are safety constraints included?"
}
```

## Reference Files

- **LLM Integration**: `.ai-config/skills/llm-integration/SKILL.md`
- **AI Code Assistant**: `.ai-config/agents/ai-code-assistant.md`
- **Optimize Prompts Command**: `.ai-config/commands/optimize-prompts.md`
- **LLM Best Practices**: `.ai-config/rules/llm-best-practices.md`

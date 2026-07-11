---
name: generate-code-ai
description: AI-powered code generation using LLMs
shortcut: /gen-code
---

# Generate Code with AI Command

**Author**: <AUTHOR_NAME>  
**Date**: 2026-04-06

Generate production-ready code using LLMs with context from the codebase.

## Usage

```bash
# Generate function
/gen-code --function="CreateVolume validation" --lang=go

# Generate with context
/gen-code --task="Implement volume expansion" --context=true

# Generate tests
/gen-code --tests-for="pkg/driver/controller.go::CreateVolume"

# Generate from description
/gen-code --describe="Add metrics collection to node service"
```

## Command Execution

### Complete Generation Script
```python
#!/usr/bin/env python3
# @author <AUTHOR_NAME>

import argparse
from langchain.chat_models import ChatOpenAI
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import Chroma

def generate_code(args):
    """Generate code using AI."""
    print(f"=== AI Code Generation ===")
    print(f"Task: {args.task}")
    
    # Step 1: Retrieve relevant context
    if args.context:
        print("\nStep 1: Retrieving context from codebase...")
        context = retrieve_context(args.task)
        print(f"Found {len(context)} relevant files")
    else:
        context = ""
    
    # Step 2: Generate code
    print("\nStep 2: Generating code with LLM...")
    llm = ChatOpenAI(model="gpt-4", temperature=0.2)
    
    prompt = f"""Generate Go code for storage interface:

Task: {args.task}

Relevant context from codebase:
{context}

Requirements:
- Functions <50 lines
- Add @author <AUTHOR_NAME>
- Error handling with fmt.Errorf and %w
- Follow storage interface patterns
- Include comprehensive error checking

Generate:"""
    
    generated_code = llm.invoke(prompt).content
    
    # Step 3: Validate
    print("\nStep 3: Validating generated code...")
    issues = validate_code(generated_code)
    
    if issues:
        print(f"⚠️  Found {len(issues)} issues:")
        for issue in issues:
            print(f"  - {issue}")
    else:
        print("✅ Code validation passed")
    
    # Step 4: Save or display
    if args.output:
        with open(args.output, 'w') as f:
            f.write(generated_code)
        print(f"\n✅ Code saved to: {args.output}")
    else:
        print("\n=== Generated Code ===")
        print(generated_code)
    
    print("\n=== Generation Complete ===")

def retrieve_context(task: str) -> str:
    """Retrieve relevant code context."""
    embeddings = OpenAIEmbeddings()
    vectorstore = Chroma(
        persist_directory=".ai-config/embeddings",
        embedding_function=embeddings
    )
    
    docs = vectorstore.similarity_search(task, k=3)
    return "\n\n".join([doc.page_content for doc in docs])

def validate_code(code: str) -> list:
    """Validate generated code."""
    issues = []
    
    # Check function length
    functions = extract_functions(code)
    for func in functions:
        if count_lines(func) > 50:
            issues.append(f"Function {func['name']} exceeds 50 lines")
    
    # Check for @author tag
    if "@author <AUTHOR_NAME>" not in code:
        issues.append("Missing @author <AUTHOR_NAME> tag")
    
    # Check error handling
    if "fmt.Errorf" not in code and "return" in code:
        issues.append("Missing proper error handling")
    
    return issues

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--task", required=True)
    parser.add_argument("--context", action="store_true")
    parser.add_argument("--output", type=str)
    parser.add_argument("--lang", default="go")
    
    args = parser.parse_args()
    generate_code(args)
```

## Generation Modes

### 1. Function Generation
```bash
# Generate complete function
/gen-code --function="ValidateVolumeCapabilities" \
  --params="volumeId string, capabilities []VolumeCapability" \
  --return="error"
```

### 2. Test Generation
```bash
# Generate table-driven tests
/gen-code --tests-for="pkg/driver/node.go::NodePublishVolume" \
  --coverage=100
```

### 3. Refactoring
```bash
# Suggest refactorings
/gen-code --refactor="pkg/driver/controller.go::CreateVolume" \
  --target-lines=50
```

## Expected Output

```
=== AI Code Generation ===
Task: Create volume validation function

Step 1: Retrieving context from codebase...
Found 3 relevant files:
  - pkg/driver/validation.go
  - pkg/driver/controller.go
  - pkg/types/volume.go

Step 2: Generating code with LLM...
✅ Code generated (45 lines)

Step 3: Validating generated code...
✅ Code validation passed
  ✅ All functions <50 lines
  ✅ @author tag present
  ✅ Proper error handling

=== Generated Code ===
```go
// ValidateVolumeRequest validates CreateVolume request.
// @author <AUTHOR_NAME>
func ValidateVolumeRequest(req *csi.CreateVolumeRequest) error {
    if req.GetName() == "" {
        return fmt.Errorf("volume name required: %w", ErrInvalidArgument)
    }
    
    if req.GetVolumeCapabilities() == nil {
        return fmt.Errorf("volume capabilities required: %w", ErrInvalidArgument)
    }
    
    for _, cap := range req.GetVolumeCapabilities() {
        if err := validateCapability(cap); err != nil {
            return fmt.Errorf("invalid capability: %w", err)
        }
    }
    
    return nil
}
```

✅ Code saved to: pkg/driver/validation_generated.go

=== Generation Complete ===
```

## Best Practices

### 1. Always Use Context
```bash
# ✅ GOOD: Retrieve context
/gen-code --task="Add caching" --context=true

# ❌ BAD: No context
/gen-code --task="Add caching"
```

### 2. Validate Generated Code
```bash
# Always review before using
/gen-code --task="..." --output=temp.go
# Review temp.go
# Then move to actual file
```

### 3. Iterative Refinement
```bash
# Generate
/gen-code --task="Create function X"

# Refine
/gen-code --task="Optimize function X for performance" --context=true
```

## Reference Files

- **AI Code Assistant**: `.ai-config/agents/ai-code-assistant.md`
- **LLM Integration**: `.ai-config/skills/llm-integration/SKILL.md`
- **Prompt Engineering**: `.ai-config/skills/prompt-engineering/SKILL.md`
- **AI Code Standards**: `.ai-config/rules/ai-code-standards.md`

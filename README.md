# ShopAssist AI 2.0

ShopAssist AI 2.0 is a function-calling based laptop recommendation assistant that collects a user's requirements, matches them against a product catalog, validates results, and returns structured top recommendations in a consistent format.

## Overview

This project upgrades a rule-and-prompt driven chatbot into a more reliable assistant by using OpenAI Function Calling for structured tool invocation. It captures user preferences across key laptop attributes such as GPU intensity, display quality, portability, multitasking, processing speed, and budget, then calls a matching function to score and rank products from a catalog.

A moderation layer is also added before model processing and optionally after tool execution to improve safety and compliance.

## Objectives

- Replace brittle prompt parsing with structured function calling
- Improve consistency of recommendation outputs
- Add moderation for user inputs and tool outputs
- Keep the conversation natural by asking at most two features per turn
- Build a cleaner, more testable architecture

## Key Features

- **Function Calling Integration** for schema-driven tool execution
- **Catalog-based Recommendations** using feature matching and budget filtering
- **Safety Layer** with input moderation and optional output moderation
- **Structured Recommendation Format** for stable top-3 responses
- **Validation Logic** to filter out low-confidence recommendations

## Project Workflow

1. Initialize a controlled conversation with a system prompt
2. Collect user requirements step by step
3. Run pre-input moderation on the latest user message
4. Let the model invoke the comparison function automatically
5. Read the laptop catalog and score matching products
6. Validate the recommendations
7. Optionally moderate tool output
8. Return the final top-3 recommendations in a fixed format

## System Architecture

### Core Components

#### 1. Conversation Manager
Defines the assistant behavior, dialog rules, and questioning strategy.

#### 2. Function Schema
Declares `compare_laptops_with_user` with structured JSON arguments so the model can send valid tool inputs.

#### 3. Moderation Layer
- **Pre-input moderation** checks user prompts before model inference
- **Post-tool moderation** optionally checks generated tool output before sending it back to the model

#### 4. Recommendation Engine
Reads `updated_laptop.csv`, filters products by budget, maps qualitative preferences to ordinal values, scores each candidate, and returns ranked results.

#### 5. Validation Layer
Filters out weak matches using a confidence threshold.

#### 6. Dialogue Orchestrator
Runs the user interaction loop, handles tool calls, validates results, and formats the final output.

## Function Calling Design

### Tool Name
`compare_laptops_with_user`

### Expected Inputs
- `gpu intensity`
- `display quality`
- `portability`
- `multitasking`
- `processing speed`
- `budget`

The first five fields are expected as `low`, `medium`, or `high`, and budget is expected as an integer in INR.

### Behavior
The function returns top laptop recommendations that meet or exceed the requested requirements and pass the validation threshold.

## Matching Logic

The recommendation algorithm follows a simple rule-based scoring approach:

- Parse the laptop catalog from `updated_laptop.csv`
- Convert `Price` into numeric format
- Filter laptops within the user's budget
- Convert qualitative feature levels into numeric values:
  - `low = 0`
  - `medium = 1`
  - `high = 2`
- Score each laptop by checking whether its feature level meets or exceeds the user's requested level
- Rank laptops by score
- Return top matches after validation

## Output Format

The final assistant response returns the top three laptops in a consistent format, typically ordered by price in descending order:

1. Laptop Name : Key specs, Price in ₹
2. Laptop Name : Key specs, Price in ₹
3. Laptop Name : Key specs, Price in ₹

## Moderation Strategy

### Pre-input Moderation
Every user message is checked before sending it to the chat model.

### Post-tool Moderation
Tool output can optionally be checked before being injected back into the model context.

### Policy Behavior
The system currently uses a fail-open moderation fallback on network or moderation errors, though this can be configured differently for stricter deployments.

## Testing Approach

### Functional Tests
- Valid recommendation flow for laptop buying use cases
- Budget edge cases where no laptops fall within range
- Multi-turn requirement gathering with controlled questioning

### Moderation Tests
- Disallowed input refusal before tool execution
- Unsafe tool output blocked before final model response

### Quality Goals
- More stable output formatting
- Reduced parser fragility
- Improved reliability and maintainability

## Challenges Addressed

- Handling feature dictionaries stored as strings in CSV
- Enforcing exact function schema keys
- Managing moderation latency and false positives
- Keeping the conversation short but informative

## Lessons Learned

- Function calling significantly improves reliability over pure prompt extraction
- Small, focused tools are easier to debug and maintain
- Safety checks are easiest to add at the system boundaries
- Controlled dialog design improves user experience and recommendation quality

## Limitations

- The catalog is CSV-based rather than database-backed
- Ranking logic is rule-based rather than learned
- Interface is CLI-oriented rather than web-based
- Observability and telemetry are limited

## Future Improvements

- Replace CSV with database or API-backed product inventory
- Add real-time price and availability updates
- Introduce weighted ranking or ML-based recommendation scoring
- Build a web UI with streaming output
- Add telemetry for tool usage, latency, and moderation decisions

## Project Structure

A typical implementation includes:

```text
ShopAssist_AI_2_0/
├── updated_laptop.csv
├── main.py / notebook.ipynb
├── recommendation utilities
├── moderation utilities
└── README.md
```

## Requirements

- Python 3.9+
- `openai`
- `pandas`

Install dependencies with:

```bash
pip install openai pandas
```

## Configuration

Set the required environment variable:

```bash
export OPENAI_API_KEY="your_api_key"
```

Optional settings:

- `ENABLE_MODERATION=1`
- `SHOPASSIST_MOD_MODEL=omni-moderation-latest`

## How to Run

Run the script or notebook and provide laptop requirements interactively.

Example flow:

```text
User: for AI and ML
Assistant: asks follow-up questions on performance, portability, and budget
User: with high cpu, gpu, portability
User: 100000
User: high
Assistant: returns top-3 laptop recommendations
```

## Use Case

This project is suitable for:

- Laptop recommendation assistants
- Function-calling chatbot demos
- Safe tool-using LLM workflows
- Structured conversational commerce applications

## Author Notes

ShopAssist AI 2.0 demonstrates how function calling, light validation, and moderation can work together to create a more production-ready recommendation assistant with reduced glue code and improved output consistency.

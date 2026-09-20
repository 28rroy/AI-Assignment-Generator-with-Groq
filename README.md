
# Owlyard — AI Assignment Generator with Groq

Developed as part of my AI Full Stack Developer role at Owlyard, focusing on prompt engineering and validation for AI-generated educational assignments. Shared with permission from Owlyard.

A Python notebook that generates multiple-choice educational assignments using an LLM, checks the response structure, and requests a second model review of the answers and explanations.

## Overview

The notebook accepts a grade level, subject, topics, and question count. It requests an assignment in JSON format, applies Python validation checks, and asks the same model to review the proposed answers. Responses that fail these checks are regenerated, with up to five generation attempts by default.

The included configuration requests five middle-school math questions on linear equations and fractions.

## Features

- **Configurable prompts:** Specify the educational level, subject, topics, and number of questions.
- **Structured output:** Request questions, four answer options, answer indices, and explanations in a consistent JSON format.
- **Python validation:** Check question counts, option counts, answer-index bounds, explanation length, and point values.
- **Model-based review:** Make a separate request to check whether the marked answers and explanations agree.
- **Bounded retries:** Regenerate responses that fail parsing, structure checks, or model review.

## How it works

```text
Assignment settings
        |
        v
Build prompt and request generation
        |
        v
Parse JSON and validate structure
        |
        v
Request model review of answers and explanations
        |
        v
Return formatted JSON

Failed checks -> regenerate, up to the attempt limit
```

| Function | Purpose |
| --- | --- |
| `build_prompt()` | Builds the assignment prompt from the input settings and output rules. |
| `validate_output(data)` | Applies structural checks to the parsed response. |
| `verify_with_model(data)` | Requests a separate model review of answer correctness and explanation consistency. |
| `generate_assignment(max_retries=5)` | Coordinates generation, validation, review, and retries. |

## Technology

- Python and Jupyter Notebook
- Groq's OpenAI-compatible API
- OpenAI Python client, configured to call Groq
- `openai/gpt-oss-120b`, as configured in the notebook
- Python's built-in `json` module

## Getting started

### 1. Install dependencies

In your Python environment:

```bash
python -m pip install openai notebook
```

### 2. Configure credentials

Before running the notebook, replace its `YOUR_GROQ_API_KEY` placeholder with an environment-variable lookup:

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["GROQ_API_KEY"],
    base_url="https://api.groq.com/openai/v1",
)
```

Set `GROQ_API_KEY` in the environment used to launch Jupyter. Keep the key out of notebook source, saved outputs, and version control. This environment-variable change is a setup step; the current notebook does not yet implement it.

You need a Groq account and access to the configured model. If that model is unavailable to your account, update `MODEL` to an available compatible model.

### 3. Open the notebook

```bash
jupyter notebook Practice-Groq.ipynb
```

Edit `userRequestData` and run the cells in order. The notebook prints attempt status and, when all checks pass, the resulting JSON. Running it sends requests to Groq and uses your account's API quota.

## Configuration

The notebook uses this exact Python dictionary:

```python
userRequestData = {
    "level": "middle_school",
    "subject": "math",
    "topics": "solving linear equation, fraction",
    "numModules": "1",
    "moduleTypes": { "1": "mcq_4" },
    "moduleQuestionCounts": { "1": 5 },
    "multipleCorrect": "no",
    "explanations": "yes",
}
```

| Field | Format and meaning |
| --- | --- |
| `level` | String specifying the educational level. |
| `subject` | String specifying the assignment subject. |
| `topics` | A comma-separated string of topics, not a list. |
| `numModules` | String `"1"` indicating one module. |
| `moduleTypes` | Dictionary mapping the string module ID `"1"` to `"mcq_4"` (four-option multiple choice). |
| `moduleQuestionCounts` | Dictionary mapping the string module ID `"1"` to the integer question count `5`. |
| `multipleCorrect` | String `"no"` indicating one correct answer per question. |
| `explanations` | String `"yes"` requesting explanations. |

Keep the quoted values as strings; `5` is an integer, and `"yes"`/`"no"` are not Python booleans.

The current code reads `level`, `subject`, `topics`, and `moduleQuestionCounts["1"]`. The other fields describe the intended request, but the prompt currently fixes the format to four options, one correct answer, explanations, and zero point values. Changing those fields alone does not enable additional modules or question formats.

`max_retries` is not a field in `userRequestData`. It is a separate parameter of `generate_assignment(max_retries=5)` that limits generation to five attempts by default.

## Output format

Illustrative example for a one-question request, not a recorded model output:

```json
{
  "questions": {
    "1": {
      "question": "Solve for x: 2x + 3 = 11.",
      "options": ["2", "3", "4", "5"],
      "questionType": "single",
      "points": 0
    }
  },
  "correctAnswers": [[2]],
  "explanations": [
    "Subtract 3 from both sides to get 2x = 8, then divide by 2 to get x = 4."
  ]
}
```

Answer indices are zero-based, so `2` identifies the third option. Answers and explanations correspond to the question iteration order.

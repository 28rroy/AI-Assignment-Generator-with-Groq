# AI Assignment Generator with Groq

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

Before running the notebook, replace its hard-coded `api_key` value with an environment-variable lookup:

```python
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

The current implementation uses these settings:

| Setting | Example | Purpose |
| --- | --- | --- |
| `level` | `"middle_school"` | Intended educational level |
| `subject` | `"math"` | Assignment subject |
| `topics` | `"solving linear equation, fraction"` | Topics included in the prompt |
| `moduleQuestionCounts["1"]` | `5` | Number of questions to generate |

Other fields appear in the input dictionary, but the current prompt is fixed to one module of four-option, single-answer questions with explanations and zero point values. Changing those additional fields alone does not change the question format.

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

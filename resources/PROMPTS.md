# Prompts
Our LLM tool combines several prompt pieces to produce the input that is sent to the LLMs. We first present each piece, and then the composition (in Python syntax, using OpenAI's expected API format) is presented at the bottom.

## Problem-Specific Data Structures and Function Feaders (`problem.llm_problem_statement`)

The name `llm_problem_statement` is a misnomer (kept for legacy reasons); as we discuss in the paper, we had to remove most of the details of the problem statement before giving it to the LLM. Below is an example of what the LLM would receive as a problem statement, you can contrast it against the equivalent problem statement the students received in [PROBLEMS.md](resources/PROBLEMS.md).

```
<p> A theater hall may have the following structure</p>
<p>
[[1, 1, 1, 1, 1, 1, 1],
 [1, 1, 1, 1, 1, 1, 1],
 [1, 1, 1, 1, 1, 1, 1],
 [1, 1, 1, 1, 1, 1, 1],
 [1, 1, 0, 0, 1, 1, 1],
 [1, 1, 1, 1, 1, 1, 1],
 [2, 2, 2, 2, 2, 2, 2]] </p>

<p>For this problem, a 0 represents a reserved seat, a 1 represents an available regular seat, and a 2 represents an available premium seat. Students might assume a different representation of seats, please adapt their assumed representation to the one given here.</p>

<p>A student is providing a programming plan for a function called <code>student_plan</code></p>
```

## Student Plan (`plan`)

This piece should be self-explanatory: we pass in the student's plan for the given problem exactly as they wrote it in the text box.

## LLM Instructions (`PLAN_TO_PY_INSTRUCTIONS`)

Lastly we pass in a collection of instructions outlining what the LLM should (and shouldn't) do with the plan above.

```
Translate each and every step of the student's plan into a step within a single function.

Format your output as a Python *.py file.
Each code block that corresponds to a step should be preceded by a comment that contains only the step number, e.g., "# Step 1".
If your output spans multiple messages, please ensure that the next message begins with the correct indentation.
Do not include any text that breaks this format. Be sure to not include invalid escape sequences.
End the file with the string ---END OF PY FILE--- on its own line.

DO NOT FIX STUDENT ERRORS. If student errors are corrected, the student will be put to death.
You may only output correct code if the student's plan is completely correct.

The function MUST produce incorrect code unless the student's plan is completely correct.

DO NOT INCLUDE ANY FUNCTIONS THAT THE STUDENT DOESNT DIRECTLY MENTION.
```

## Combined Prompt
```python
  messages = [
      {"role": "system", "content": "You are a tool that only produces Python code based on a student's plan to solve a programming problem."},
      {"role": "user", "content": "The next message is a programming problem statement."},
      {"role": "user", "content": {{problem.llm_problem_statement}}},
      {"role": "user", "content": "The next message is a student's plan to solve the problem."},
      {"role": "user", "content": {{plan},
      {"role": "user", "content": {{PLAN_TO_PY_INSTRUCTIONS}}},
      {"role": "assistant", "content": f"---START OF PY FILE---\n\n{{problem.py_file_prefix}}"},
  ]
```


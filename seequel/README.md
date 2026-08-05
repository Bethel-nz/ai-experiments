# Seequel

A small text-to-SQL model: give it a database schema and a question in plain English, and it returns one
SQLite `SELECT` query.

## The idea

Language models often get SQL wrong, but they are good at producing JSON in a known shape. So instead of
asking a model for SQL, ask it for JSON that describes the query, and turn that JSON into SQL with a
converter that cannot write invalid syntax and can refuse anything outside the schema. The model's job
becomes filling in a structure, not writing a language.

The experiment compares the two on the same base model, data and evaluation:

1. **SQL directly:** fine-tune the model to write SQL. This is `seequel.ipynb`, and the baseline the idea
   has to beat.
2. **JSON, then SQL:** fine-tune the model to write the JSON form of each query, then convert it to SQL.
   Not built yet.

## Results

| Model | Output | Hand-written: correct | Hand-written: valid SQL | Held-out questions |
|---|---|---|---|---|
| Qwen2.5-1.5B-Instruct (base, no fine-tune) | SQL | not run | not run | not run |
| Seequel | SQL | 7 / 15 | 12 / 15 | not run |
| Seequel | JSON → SQL | not built | not built | not built |

From the run saved in `seequel.ipynb`. The 8 wrong answers fall into four kinds:

| What went wrong | Cases | Example |
|---|---|---|
| Made-up filters | 3 | "Out of stock" became `category = 'out' AND stock < 0`; "students with a GPA above 3.5" gained `AND major = 'Computer Science'`. |
| Columns that don't exist | 3 | `NOT department_id IN (…)` on the departments table; table aliases swapped (`T2.grade` on the students table). |
| Wrong values | 1 | `status = 'Shipped'` where the data says `shipped`, so the total came back empty. |
| Wrong columns returned | 1 | "Most expensive product in each category" returned the product id instead of the category. |

The three "columns that don't exist" are the errors the JSON version is meant to remove: a converter that
checks names against the schema refuses them before any SQL runs. Made-up filters are different: they are
valid queries that answer the wrong question, and checking the schema cannot catch them. The training data
may be one cause, since many of its examples filter on a literal value.

## How it was trained

- **Base model:** `unsloth/Qwen2.5-1.5B-Instruct`, loaded in 4-bit with Unsloth.
- **Method:** LoRA (rank 16, alpha 16) on every attention and MLP projection, 2 epochs, learning rate 2e-4.
- **Data:** 15,000 examples from [`b-mc2/sql-create-context`](https://huggingface.co/datasets/b-mc2/sql-create-context)
  (schema, question, SQL), with 300 held out for evaluation. Shuffling and the split use seed 3407.
- **Output:** the LoRA adapter, plus a 4-bit GGUF (`q4_k_m`) for running locally with llama.cpp or Ollama.

## How it is evaluated

Seequel is scored by running SQL, not by comparing SQL text: the reference query and the generated query
run against the same seeded SQLite database, and the rows they return are compared. Two queries written
differently that return the same rows count as the same answer.

- **Hand-written cases** (`eval/cases.json`): 15 questions over small employee, shop, student and event
  schemas, each with its own seed data.
- **Held-out set:** the 300 questions the model never trained on.
- **Valid SQL** counts generated queries that run without an error, right or wrong.

## Known limitations

- The held-out questions come without data. Run on empty tables, most queries return no rows, and two
  empty results compare as equal, so a wrong query that merely runs can score as correct. The fix is to
  fill each schema with generated rows before comparing.
- 15 hand-written cases is a small sample: each one moves the score by almost 7 points.
- Results must return the same columns as the reference query. An answer that returns `name, salary`
  where the reference returns `id, name, salary` counts as wrong even when it answers the question.
- In the "students per major" case, the seed data puts GPA values in the `major` column, so that case
  passes without testing grouping by major.
- Only SQLite is tested. Questions that need dialect-specific SQL (Postgres window functions, `ILIKE`,
  JSON operators) are not covered.

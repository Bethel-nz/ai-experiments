# ai-experiments

Notebooks from building with language models. Each folder has its own README covering what the project
does, how it works, and what it doesn't do well yet.

| Project | What it is |
|---|---|
| [seequel](./seequel) | Tests whether a small model writes better SQL by producing JSON that a converter turns into SQL. The direct-SQL baseline is built and scored; the JSON version is next. |
| [cadence](./cadence) | Matches resumes to jobs from three signals: embedding similarity, skill coverage and years of experience. |
| [seo-bot](./seo-bot) | Crawls a website, scores its SEO basics, and asks Gemini for specific fixes. |

These are experiments, not maintained libraries. Notebooks run on Google Colab; Seequel needs a GPU.

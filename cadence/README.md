# Cadence

Ranks resumes against a job description, or jobs against a resume, and returns a score, a recommended
action and the skills each side has in common or is missing.

## How it scores

Each match combines three signals into one score out of 100:

| Signal | Weight | How it is measured |
|---|---|---|
| Skill coverage | 45% | Of the skills the job asks for, the share the candidate has, weighted by how confidently each skill was found in the job description. |
| Semantic similarity | 40% | Cosine similarity of `all-mpnet-base-v2` sentence embeddings of the two documents. |
| Experience | 15% | Candidate years over required years, capped at 0.8 when the candidate has fewer. |

The score maps to an action: priority review (80+), review (60+), consider (45+), otherwise reject.

Skills are extracted three ways, most trusted first: SkillNER (when installed), regular expressions for
acronyms and names like `Node.js` or `C++`, and spaCy noun phrases.

## Known limitations

- Years of experience is the largest number next to "years" anywhere in the text, which can pick up
  unrelated numbers.
- Number words are replaced inside other words before parsing ("often" becomes "of10").
- The weights and thresholds are set by hand and have not been checked against hiring decisions.

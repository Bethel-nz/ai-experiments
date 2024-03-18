# SEO bot (Untitled AI)

Give it a website URL. It crawls the site page by page, pulls out each page's images, headings, paragraphs
and meta tags, gives the page a simple SEO score, and asks Gemini for specific fixes, with examples. The
tips are written to `seo_analysis.txt`.

## Running it

1. Open the notebook in [Google Colab](https://colab.research.google.com/).
2. Get a key from [Google AI Studio](https://aistudio.google.com/app/apikey) and add it as a Colab secret
   named `GOOGLE_API_KEY` (the key icon in the left sidebar).
3. Run every cell and enter the URL when asked.

## How it scores a page

Out of 40 points: 15 if the most common word makes up more than 2% of the text, 10 if any image has alt
text, 10 if the title, description and author meta tags are all present, and 5 for a Flesch reading-ease
score above 60.

## Known limitations

Written against `gemini-pro` and the `google-generativeai` SDK, both since replaced.

- The page summary sent to Gemini is capped at one token, so it carries almost nothing.
- The crawler has no page limit and follows links to other sites with two-part domains, so it can wander
  off the site it was given.
- Each page is downloaded twice.

# cvpr2026-papers

List of CVPR 2026 papers with AI-extracted keywords.

## CSV schema

`cvpr26-papers.csv` contains one row per paper:

For easier browsing on GitHub, split copies of the CSV are also available in `view/` as `cvpr26-papers-part-01.csv` through `cvpr26-papers-part-05.csv`.

| column | description |
|---|---|
| `title` | Paper title from the CVPR 2026 Open Access listing. |
| `pdf` | CVF Open Access PDF URL. |
| `arxiv` | arXiv abstract URL when a matching arXiv record was found; blank otherwise. |
| `keywords` | Up to 10 semicolon-separated topical keywords extracted by an LLM. |
| `conference_day` | CVPR 2026 conference-day bucket used when scraping the Open Access listing. |

## Keyword extraction

The `keywords` column is generated from each paper's abstract, not from manual tagging.

The pipeline used two abstract sources:

1. **Papers with arXiv links**
   - Fetch the arXiv abstract page.
   - Extract the text inside the arXiv abstract block.
   - Send the title and abstract to an LLM prompt that asks for up to 10 distinct, specific computer-vision keywords or short noun phrases.

2. **Papers without arXiv links**
   - Download the CVF PDF.
   - Run `pdftotext` on the first two pages.
   - Heuristically slice the abstract text from `Abstract` to the `Introduction` heading.
   - Send the title and recovered abstract to the same keyword-extraction prompt.

The model prompt required a single-line output using `; ` as the separator, with no numbering or extra prose. The extractor preferred specific technical phrases such as `3D Gaussian Splatting`, `novel view synthesis`, `vision-language models`, or `semantic segmentation` over generic terms.

Extraction was run as a resumable batch job. Rows were saved incrementally, and rows with missing abstracts or failed model calls could be retried without regenerating the whole CSV. The primary model used for keyword extraction was `claude-haiku-4-5`; a fallback model was configured for rate-limit or provider failures during the PDF-based fill pass.

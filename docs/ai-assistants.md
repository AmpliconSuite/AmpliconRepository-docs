# Use AmpliconRepository with an AI assistant

AmpliconRepository is designed to be readable by AI assistants, not just by
people. If you use Claude, ChatGPT, Gemini, or an agent you have built
yourself, it can query the repository's amplification calls directly and answer
questions from the primary data rather than from whatever it remembers.

## The one thing to paste

```
https://ampliconrepository.org/llms.txt
```

That file is a short, plain-text briefing written for machine readers. It names
the endpoints, gives worked query examples, and — the part that matters most —
states the limits of the data, so an assistant that reads it does not overstate
what a filtered count means.

A prompt that works with essentially any assistant:

> Read https://ampliconrepository.org/llms.txt and then use the
> AmpliconRepository API to answer: **which cancer types have the most ecDNA
> amplifications containing MYC?**

Assistants that can fetch URLs will follow it from there. Those that cannot
fetch will still accept the file's contents pasted in.

## What it can now do

Nearly every question resolves to a single call against
`GET /api/v1/features/`, which searches every amplicon in the repository at
once — no need to find a project first.

| Question | Call |
| --- | --- |
| Where is MYC on ecDNA? | `/api/v1/features/?gene_any=MYC&classification=ecDNA` |
| Are MYC and PVT1 on the *same* amplicon? | `/api/v1/features/?gene_all=MYC,PVT1&same_amp=true` |
| How many EGFR amplifications are there? | `/api/v1/features/?gene_any=EGFR&count_only=true` |
| What can I filter on? | `/api/v1/features/facets/` |
| Which **samples**, not which amplicons? | `/api/v1/features/samples/?gene_any=MYC` |
| Is the U2OS cell line in here? | `/api/v1/features/samples/?sample_name_contains=U2OS` |

Every row names its project, sample, feature, classification, genes, oncogenes,
coordinates, and reference build, and carries `project_url`, `sample_url` and
`sample_page_url` so the assistant can cite what it used. See the
[API Reference](api.md) for the full parameter list, or fetch
[`/api/v1/openapi.json`](https://ampliconrepository.org/api/v1/openapi.json)
for the machine-readable specification.

## What to check in the answer

The repository's amplicon calls — classification, genes, coordinates,
reference build — are computed for every sample and are complete.

**But not every row is an amplicon.** A sample the pipeline analysed and found
clean is kept as a result, with `classification: "None"` and no genes, and
those rows are a large minority of the corpus. An assistant that answers "there
are N amplicons" from a raw row count will overstate it. Ask for
`classification=None` excluded, or ask what fraction of samples came back clean
— which is a real and interesting question the same data answers.

The **sample metadata is not**. `cancer_type`, `sample_type`, and
`tissue_of_origin` are free text supplied by whoever submitted each project.
They are not controlled vocabularies, they are not unified across projects, and
many samples carry no value at all. This produces one specific failure, and it
is worth knowing how to spot it:

!!! warning "A filtered count is a floor, not an answer"

    "There are 3 MYC ecDNA amplifications in breast cancer" is almost
    certainly wrong. What is true is that 3 rows **say** breast — samples whose
    submitter never recorded a cancer type cannot be reached by any filter on
    cancer type, and they may be the majority.

    A good answer says so. `/api/v1/features/facets/` returns `total_rows`
    alongside the per-value counts precisely so an assistant can compare the
    two and report the shortfall.

!!! warning "Rows are amplicons; the denominator is usually samples"

    Most samples carry several rows and some carry over a hundred, so a
    percentage computed from row counts is not a percentage of samples. Every
    search response carries `sample_count` next to `count`; a good answer uses
    it for both halves of any fraction of samples.

    The related trap is on the way in: an assistant that deduplicates
    `sample_name` itself will merge distinct samples, because thousands of
    names here occur in more than one project. Identity is project **and**
    name.

!!! warning "A cell line's name here may not be the name you know"

    The same line can be recorded as `U2OS_BONE`, or with a clone suffix like
    `G-292_clone_A141B1`. `sample_name_contains=` finds those, but a substring
    match is not an identity: `HOS` matches `HOS-MNNG`, which is a different
    line, and hundreds of names here are a prefix of another.

    Ask for `/api/v1/features/samples/?sample_name_contains=...` and for the
    candidate list it returned, so you can see which sample the answer is
    actually about.

Two related habits worth asking for:

- **Ask which project, and check its coverage.** Each project object carries
  `metadata_coverage`; a `0.0` for `cancer_type` means that project recorded
  none, so filtering it by cancer type could only ever return nothing. Its
  `description` also states the cohort and the paper, which is context worth
  quoting.
- **Check the spelling used.** Case is folded for you, but granularity is not:
  `Breast` and `Breast Adenocarcinoma` are separate values covering overlapping
  samples. An assistant should read `/features/facets/` and query the spellings
  it finds there rather than guessing.
- **Treat `NA` as a value, not a gap.** It means the submitter recorded "not
  applicable". It is filterable, and it is common.

If an answer cites a project and version, you can verify it: every row carries
a URL, and projects are versioned so that published results stay reachable.

## If you are building an agent

- **No authentication is required for public data.** A personal token from your
  Profile page additionally reaches private projects you are a member of, and
  raises rate limits.
- **No browser User-Agent is required** on `/api/v1/` paths, `/robots.txt`, or
  `/llms.txt`. Send whatever your HTTP library sends.
- **Errors are uniform**: `{"error": "...", "code": "..."}`. Branch on `code`.
- **An unrecognised parameter is a `400`**, and the error names the parameters
  that do exist. It is never silently ignored, so a `200` means your filter was
  applied.
- **Respect `429`.** The response carries `Retry-After`. The limits exist
  because uncontrolled crawling has taken this site down before.
- **Please query the API rather than crawling the HTML pages.** One call
  answers what would otherwise take hundreds of page renders, and the JSON is
  easier to parse than the rendered tables.

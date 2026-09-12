# AmpliconRepository API

AmpliconRepository provides a REST API for finding projects, inspecting project metadata, listing samples, and downloading full project archives from the command line.

Public projects can be listed and downloaded without logging in. Private projects require a personal API token from your AmpliconRepository profile.

Base URL:

```
https://ampliconrepository.org/api/v1
```

!!! tip "You may not need to read this page"

    If you are working with an AI assistant, give it this link and ask your
    question in plain English:

    ```
    https://ampliconrepository.org/llms.txt
    ```

    That file is written for an AI agent rather than a person. It describes
    every endpoint, the traps that produce confidently wrong answers, and the
    conventions this API follows, and it points at the machine-readable
    specification. Assistants that can fetch a URL can go from *"how many
    glioblastoma samples here carry ecDNA?"* to the right query without your
    help.

    Check the answer the way you would check a colleague's: ask which endpoint
    and filters it used, and whether the denominator counts samples or
    amplicons. [Use AmpliconRepository with an AI assistant](ai-assistants.md)
    covers what to ask for.

## Endpoint quick reference

| Method | Endpoint | Auth | Purpose |
| --- | --- | --- | --- |
| `GET` | `/projects/` | optional | List projects (public always; private/hidden with a token) |
| `GET` | `/projects/?name=<substr>` | optional | Same, filtered by case-insensitive name substring |
| `GET` | `/projects/<id>/` | optional | Project metadata |
| `GET` | `/projects/<id>/samples/` | optional | Sample-level metadata for a project |
| `GET` | `/projects/<id>/samples/<name>/` | optional | One sample's rows |
| `GET` | `/projects/<id>/download/` | optional | Download the project `.tar.gz` (302-redirects to storage) |
| `POST` | `/projects/download/` | optional | Resolve several project IDs to download URLs in one call |
| `GET` | `/features/` | optional | Search every amplicon in the repository at once |
| `GET` | `/features/samples/` | optional | The same search, one entry per sample instead of per amplicon |
| `GET` | `/features/facets/` | optional | The values `/features/` can be filtered on, with counts |
| `GET`/`POST`/`DELETE` | `/token/` | required | Inspect, regenerate, or revoke your API token |
| `GET` | `/openapi.json` | none | Machine-readable specification of every endpoint |

All responses are JSON except `download/`, which returns a `.tar.gz` archive. Auth is "optional" in the sense that public projects need no token; private projects require one.

If you are searching for amplifications rather than fetching a known project, start at [Search every amplicon](#search-every-amplicon) — one call there replaces a loop over `/projects/` and `/samples/`.

## Setup

Copy these into your shell once; every example below reuses them.

```bash
BASE="https://ampliconrepository.org/api/v1"

# Only needed for private projects — paste a token from your Profile page.
TOKEN="paste-your-token-here"
```

!!! note "No browser User-Agent is needed"

    Every `/api/v1/` path answers a plain `curl`, `requests`, or any other
    client, as of September 2026. Earlier versions of this page told you to set
    a browser `User-Agent`; that is no longer required, and the `-A "$UA"` in
    older scripts is harmless but does nothing. The HTML pages are still gated,
    so a `403` from a page URL is expected and a `403` from an API path means
    something else — see [Common errors](#common-errors).

## Search every amplicon

`GET /features/` searches the amplicons of every project you can see, in one
call. You do not need to find a project first, and this is usually the endpoint
you want.

```bash
# Every ecDNA amplicon containing MYC, across all public projects
curl -s "$BASE/features/?gene_any=MYC&classification=ecDNA"

# MYC and PVT1 on the same amplification, not merely in the same sample
curl -s "$BASE/features/?gene_all=MYC,PVT1&same_amp=true"

# Just the count, with a per-reference-build breakdown
curl -s "$BASE/features/?gene_any=EGFR&count_only=true"
```

The response is `{"count": N, "sample_count": M, "results": [...],
"next_cursor": "...", "reference_builds": {...}}`. Page by passing the returned
`next_cursor` back as `?cursor=`. Each row carries:

`project_id`, `project_name`, `project_url`, `sample_name`, `sample_url`,
`sample_page_url`, `feature_id`, `classification`, `genes`, `oncogenes`,
`locations`, `reference_build`, `cancer_type`, `sample_type`,
`tissue_of_origin`.

`sample_url` fetches that one sample's rows; `sample_page_url` is the
human-readable page, which is what to cite or hand to a colleague.

Use `?fields=` to ask for a subset — `genes` and `locations` are large, and
omitting them makes a bulk pull much smaller.

!!! warning "`count` counts amplicons; `sample_count` counts samples"

    Most samples carry several rows, and some carry over a hundred, so the two
    numbers are not interchangeable and the gap is big enough to change a
    conclusion. Any "what fraction of samples" question wants `sample_count`
    on both sides of the fraction.

    Do not try to recover it by deduplicating `sample_name` in the results —
    a sample is identified by **project and name together**, and thousands of
    names in this repository occur in more than one project (`COLO320DM` is in
    four). Deduplicating on the name alone silently merges different samples.

!!! warning "Not every row is an amplicon"

    A sample that was analysed and found to carry no focal amplification is a
    result, and it is in the corpus as a row with `classification: "None"` and
    no genes. It is a large minority of the rows.

    So a row count is not an amplicon count. Filter on `classification=None` to
    count the samples that came back clean, and exclude them before reporting
    "how many amplicons" or computing "what fraction of samples carry ecDNA".

### Filters

| Parameter | Notes |
| --- | --- |
| `gene_any`, `gene_all` | Comma-delimited gene symbols. `gene_all` requires all of them |
| `same_amp` | With `gene_all`, require the genes on one amplicon rather than one sample |
| `oncogenes_only` | Match against the oncogene list rather than all genes |
| `classification` | `ecDNA`, `BFB`, `Linear`, … — see `/features/facets/` |
| `cancer_type`, `sample_type`, `tissue_of_origin` | Free-text sample metadata |
| `project_id`, `project_name`, `sample_name` | Restrict to a project or sample. `sample_name` is exact, with case folded for you |
| `sample_name_contains` | Case-insensitive substring of the sample name — see [Finding a sample by name](#finding-a-sample-by-name) |
| `reference_build` | `hg38`, `hg19`, `mm10` |
| `limit`, `cursor`, `fields`, `count_only` | Paging and response shape |

Gene lists are comma-delimited, because a comma cannot occur in a gene symbol.
**Metadata values are not** — real ones contain commas — so pass several by
repeating the parameter:

```bash
curl -s "$BASE/features/?tissue_of_origin=Lung&tissue_of_origin=lung"
```

An unrecognised parameter is a `400` naming the ones that exist, so a `200`
always means your filter was applied.

## One entry per sample

`GET /features/samples/` takes every filter `/features/` takes and answers at
sample granularity instead of amplicon granularity:

```bash
# Which samples carry MYC on ecDNA, rather than which amplicons do
curl -s "$BASE/features/samples/?gene_any=MYC&classification=ecDNA"
```

```json
{
  "count": 412,
  "results": [
    {
      "project_id": "6a5e...",
      "project_name": "CCLE",
      "sample_name": "U2OS_BONE",
      "reference_build": "hg38",
      "row_count": 4,
      "amplicon_count": 4,
      "classifications": ["BFB", "ecDNA"],
      "project_url": "...", "sample_url": "...", "sample_page_url": "..."
    }
  ],
  "next_cursor": null
}
```

`count` is the number of matching samples, and it always equals `sample_count`
from the same query against `/features/`. `amplicon_count` of `0` means the
sample was analysed and nothing focal was found — a result, not a gap.

## Finding a sample by name

`sample_name=` is an **exact** match, with case folded for you. There is no
need to lowercase, strip punctuation, or fetch a whole project to search it
client-side.

For a name you only half know — the same cell line may be recorded here as
`U2OS_BONE`, or with a clone suffix like `G-292_clone_A141B1` —
`sample_name_contains=` is a plain case-insensitive substring. It is literal
text: no wildcards, no operators.

```bash
curl -s "$BASE/features/samples/?sample_name_contains=U2OS"
```

!!! danger "A name that contains another name is a different sample"

    `HOS` and `HOS-MNNG` are different cell lines, and hundreds of sample
    names in this repository are a prefix of another one (`COLO320` is a
    prefix of five). Substring and prefix matching will happily merge them.

    So resolve names through `/features/samples/`, look at the candidates it
    returns, and decide which one you meant — then query that exact
    `sample_name`. Summing the matches is almost never right.

## What you can filter on

`GET /features/facets/` returns every value present in the metadata fields,
with a count for each, plus `total_rows`:

```bash
curl -s "$BASE/features/facets/" | jq '.facets.classification'
curl -s "$BASE/features/facets/" | jq '.total_rows'
```

The shape, with counts standing in for whatever the repository holds when you
call it:

```json
{
  "total_rows": 37795,
  "facets": {
    "classification":    [{"value": "ecDNA", "count": 4193}, ...],
    "cancer_type":       [...],
    "sample_type":       [...],
    "tissue_of_origin":  [...],
    "reference_build":   [...]
  }
}
```

Every value listed here is filterable under the name it is listed under, and
returns exactly the count shown.

!!! warning "`total_rows` is there for a reason: a filtered count is a floor"

    `cancer_type`, `sample_type`, and `tissue_of_origin` are free text supplied
    by whoever submitted each project. They are not controlled vocabularies,
    they are not unified across projects, and **many samples carry no value at
    all**.

    A sample whose submitter never recorded a cancer type appears in no
    `cancer_type` facet entry and can never be reached by a filter on it. So
    compare the facet counts for a field against `total_rows`: the shortfall is
    what no filter on that field can see. Report it alongside any number you
    derive.

    Per project, the project object answers the same question directly:
    `metadata_coverage` gives the fraction of that project's rows carrying
    each of the three fields. A `0.0` means the project recorded none of it,
    so a filter on that field returns an empty result that is not an answer.

    Two more consequences. Case is folded for you, so `lung` and `Lung` are one
    facet entry with one count — but granularity is not: `Breast` and `Breast
    Adenocarcinoma` are separate values over overlapping samples, so query the
    spellings the facets endpoint actually lists. And `NA` is a recorded value
    meaning "not applicable", not a gap.

## Machine-readable specification

```bash
curl -s "$BASE/openapi.json"
```

The complete OpenAPI 3 description of every endpoint, parameter, and response
field. If you are generating a client, or pointing an AI assistant at this API,
fetch this rather than scraping the page you are reading — see
[Use AmpliconRepository with an AI assistant](ai-assistants.md).

## List public projects

Anyone can list public AmpliconRepository projects. No login or token is required.

```bash
curl -s "$BASE/projects/"
```

The response is a JSON array. Each project includes `project_name`, `id`, `sample_count`, `visibility`, `description`, `reference_genome`, `metadata_coverage`, and more. Use the `id` value when downloading a project.

`description` is worth reading before you query: it is written by whoever
submitted the project and states the cohort, the paper and the selection
criteria — context that appears nowhere in the feature rows.

If `jq` is installed, print a simple project-name list:

```bash
curl -s "$BASE/projects/" \
  | jq -r '.[].project_name'
```

Print project names with their download IDs and sample counts:

```bash
curl -s "$BASE/projects/" \
  | jq -r '.[] | [.project_name, .id, .sample_count] | @tsv'
```

Filter public projects by a case-insensitive project-name substring:

```bash
curl -s "$BASE/projects/?name=CCLE" \
  | jq -r '.[] | [.project_name, .id, .sample_count, .visibility] | @tsv'
```

## Download one project

Each project has a stable project ID. Use that ID in the download endpoint:

```bash
PROJECT_ID="64a1b2c3d4e5f6a7b8c9d0e1"

curl -sS -L -o "${PROJECT_ID}.tar.gz" "$BASE/projects/${PROJECT_ID}/download/"
```

- `-L` is **required**: production downloads 302-redirect to a short-lived storage URL.
- `-o <filename>` names the output file. Do **not** use `curl -O` here — the endpoint URL ends in `/download/` and carries no filename for curl to derive, so `-O` fails with `curl: (23) Failed writing received data to disk`.

The downloaded file is a `.tar.gz` archive containing the project output packaged by AmpliconSuiteAggregator. Inside, `results/aggregated_results.csv` holds the combined per-feature table; see [Project Archive Structure](project-structure.md) for the rest.

Archives are large, and for the big projects they are several gigabytes. If you are answering a question about genes, classifications or metadata, `/features/` already answers it without the download.

## Private projects

Private projects require an API token.

To create a token:

1. Log in at [ampliconrepository.org](https://ampliconrepository.org).
2. Open your Profile page.
3. Under **Developer API Token**, choose **Generate / Regenerate Token**.
4. Copy the token when it is shown (it is displayed only once).

Set `TOKEN` (see [Setup](#setup)) and add the `Authorization` header:

```bash
PROJECT_ID="64a1b2c3d4e5f6a7b8c9d0e1"

curl -sS -L -H "Authorization: Token ${TOKEN}" \
  -o "${PROJECT_ID}.tar.gz" "$BASE/projects/${PROJECT_ID}/download/"
```

The same token also lets you list private projects where you are a project member (they appear alongside public ones):

```bash
curl -s -H "Authorization: Token ${TOKEN}" "$BASE/projects/" \
  | jq -r '.[] | [.project_name, .id, .visibility] | @tsv'
```

## Download several projects

The batch endpoint resolves a list of project IDs to individual download URLs in one call. IDs that are missing, inaccessible, or have no archive are returned in `skipped`.

```bash
curl -s -X POST "$BASE/projects/download/" \
  -H "Content-Type: application/json" \
  -d '{"ids": ["id1", "id2", "id3"]}'
```

For public projects, omit the `-H "Authorization: Token ..."` header. With `jq`, download every resolved archive:

```bash
curl -s -H "Authorization: Token ${TOKEN}" -X POST "$BASE/projects/download/" \
  -H "Content-Type: application/json" \
  -d '{"ids": ["id1", "id2", "id3"]}' \
  | jq -r '.downloads[] | [.id, .download_url] | @tsv' \
  | while IFS=$'\t' read -r id url; do
      curl -sS -L -H "Authorization: Token ${TOKEN}" -o "${id}.tar.gz" "$url"
    done
```

## Inspect project metadata and samples

Project metadata:

```bash
curl -s "$BASE/projects/${PROJECT_ID}/"
```

Sample metadata (one JSON object per sample, with classification, copy-number, and oncogene fields):

```bash
curl -s "$BASE/projects/${PROJECT_ID}/samples/"
```

Add the token header for private projects:

```bash
curl -s -H "Authorization: Token ${TOKEN}" "$BASE/projects/${PROJECT_ID}/samples/"
```

## Common errors

| Status | Meaning |
| --- | --- |
| `400` | Bad request, such as a batch request where `ids` is not a JSON array. |
| `401` | A token is missing, invalid, or not authorized for the private project. |
| `403` | You are authenticated but not a member of the private project. Retrying with the same token will not succeed. |
| `404` | The project or downloadable archive was not found. |
| `429` | Rate limited. The response carries a `Retry-After` header and a `retry_after` field, both in seconds. Wait that long. |
| `503` | Temporarily unavailable — either the archive from storage, or (`code: index_unavailable`) the amplicon search index while it rebuilds. Retry shortly; an empty result would be wrong. |

Errors carry a uniform body:

```json
{"error": "human-readable explanation", "code": "machine_readable_code"}
```

Branch on `code`, not on the prose in `error`. The prose may be reworded; the
code will not.

`400` includes one case worth knowing: an unrecognised query parameter is
**rejected**, not ignored, and the error names the parameters that do exist. A
`200` therefore means your filter was applied.

</content>
</invoke>

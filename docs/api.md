# AmpliconRepository API

AmpliconRepository provides a REST API for finding projects, inspecting project metadata, listing samples, and downloading full project archives from the command line.

Public projects can be listed and downloaded without logging in. Private projects require a personal API token from your AmpliconRepository profile.

Base URL:

```
https://ampliconrepository.org/api/v1
```

## Endpoint quick reference

| Method | Endpoint | Auth | Purpose |
| --- | --- | --- | --- |
| `GET` | `/projects/` | optional | List projects (public always; private/hidden with a token) |
| `GET` | `/projects/?name=<substr>` | optional | Same, filtered by case-insensitive name substring |
| `GET` | `/projects/<id>/` | optional | Project metadata |
| `GET` | `/projects/<id>/samples/` | optional | Sample-level metadata for a project |
| `GET` | `/projects/<id>/download/` | optional | Download the project `.tar.gz` (302-redirects to storage) |
| `POST` | `/projects/download/` | optional | Resolve several project IDs to download URLs in one call |

All responses are JSON except `download/`, which returns a `.tar.gz` archive. Auth is "optional" in the sense that public projects need no token; private projects require one.

## Setup

Copy these into your shell once; every example below reuses them.

```bash
BASE="https://ampliconrepository.org/api/v1"

# The API gateway requires a standard browser User-Agent header. Set it once.
# (Any real browser UA works from any OS — Linux or macOS. This requirement
#  will be relaxed in a future update.)
UA="Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0 Safari/537.36"

# Only needed for private projects — paste a token from your Profile page.
TOKEN="paste-your-token-here"
```

## List public projects

Anyone can list public AmpliconRepository projects. No login or token is required.

```bash
curl -s -A "$UA" "$BASE/projects/"
```

The response is a JSON array. Each project includes `project_name`, `id`, `sample_count`, `visibility`, `description`, `reference_genome`, and more. Use the `id` value when downloading a project.

If `jq` is installed, print a simple project-name list:

```bash
curl -s -A "$UA" "$BASE/projects/" \
  | jq -r '.[].project_name'
```

Print project names with their download IDs and sample counts:

```bash
curl -s -A "$UA" "$BASE/projects/" \
  | jq -r '.[] | [.project_name, .id, .sample_count] | @tsv'
```

Filter public projects by a case-insensitive project-name substring:

```bash
curl -s -A "$UA" "$BASE/projects/?name=CCLE" \
  | jq -r '.[] | [.project_name, .id, .sample_count, .visibility] | @tsv'
```

## Download one project

Each project has a stable project ID. Use that ID in the download endpoint:

```bash
PROJECT_ID="64a1b2c3d4e5f6a7b8c9d0e1"

curl -sS -L -A "$UA" -o "${PROJECT_ID}.tar.gz" "$BASE/projects/${PROJECT_ID}/download/"
```

- `-L` is **required**: production downloads 302-redirect to a short-lived storage URL.
- `-o <filename>` names the output file. Do **not** use `curl -O` here — the endpoint URL ends in `/download/` and carries no filename for curl to derive, so `-O` fails with `curl: (23) Failed writing received data to disk`.

The downloaded file is a `.tar.gz` archive containing the project output packaged by AmpliconSuiteAggregator. Inside, `results/aggregated_results.csv` holds the combined per-feature table.

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

curl -sS -L -A "$UA" -H "Authorization: Token ${TOKEN}" \
  -o "${PROJECT_ID}.tar.gz" "$BASE/projects/${PROJECT_ID}/download/"
```

The same token also lets you list private projects where you are a project member (they appear alongside public ones):

```bash
curl -s -A "$UA" -H "Authorization: Token ${TOKEN}" "$BASE/projects/" \
  | jq -r '.[] | [.project_name, .id, .visibility] | @tsv'
```

## Download several projects

The batch endpoint resolves a list of project IDs to individual download URLs in one call. IDs that are missing, inaccessible, or have no archive are returned in `skipped`.

```bash
curl -s -A "$UA" -X POST "$BASE/projects/download/" \
  -H "Content-Type: application/json" \
  -d '{"ids": ["id1", "id2", "id3"]}'
```

For public projects, omit the `-H "Authorization: Token ..."` header. With `jq`, download every resolved archive:

```bash
curl -s -A "$UA" -H "Authorization: Token ${TOKEN}" -X POST "$BASE/projects/download/" \
  -H "Content-Type: application/json" \
  -d '{"ids": ["id1", "id2", "id3"]}' \
  | jq -r '.downloads[] | [.id, .download_url] | @tsv' \
  | while IFS=$'\t' read -r id url; do
      curl -sS -L -A "$UA" -H "Authorization: Token ${TOKEN}" -o "${id}.tar.gz" "$url"
    done
```

## Inspect project metadata and samples

Project metadata:

```bash
curl -s -A "$UA" "$BASE/projects/${PROJECT_ID}/"
```

Sample metadata (one JSON object per sample, with classification, copy-number, and oncogene fields):

```bash
curl -s -A "$UA" "$BASE/projects/${PROJECT_ID}/samples/"
```

Add the token header for private projects:

```bash
curl -s -A "$UA" -H "Authorization: Token ${TOKEN}" "$BASE/projects/${PROJECT_ID}/samples/"
```

## Common errors

| Status | Meaning |
| --- | --- |
| `400` | Bad request, such as a batch request where `ids` is not a JSON array. |
| `401` | A token is missing, invalid, or not authorized for the private project. |
| `403` | The gateway rejected the request — usually a missing/non-browser `User-Agent`. Set `-A "$UA"` (see [Setup](#setup)). |
| `404` | The project or downloadable archive was not found. |
| `503` | The archive is temporarily unavailable from storage. Try again later. |
</content>
</invoke>

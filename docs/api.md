# AmpliconRepository API

AmpliconRepository provides a REST API for finding projects, inspecting project metadata, listing samples, and downloading full project archives from the command line.

Public projects can be listed and downloaded without logging in. Private projects require a personal API token from your AmpliconRepository profile.

Base URL:

```bash
https://ampliconrepository.org/api/v1
```

## List public projects

Anyone can list public AmpliconRepository projects from the command line. No login or token is required.

```bash
curl "https://ampliconrepository.org/api/v1/projects/"
```

The response is JSON. Each project includes `project_name`, `id`, `sample_count`, and `visibility`. Use the `id` value when downloading a project.

If `jq` is installed, print a simple project-name list:

```bash
curl -s "https://ampliconrepository.org/api/v1/projects/" \
  | jq -r '.[].project_name'
```

Print project names with their download IDs and sample counts:

```bash
curl -s "https://ampliconrepository.org/api/v1/projects/" \
  | jq -r '.[] | [.project_name, .id, .sample_count] | @tsv'
```

Filter public projects by a case-insensitive project-name substring:

```bash
curl "https://ampliconrepository.org/api/v1/projects/?name=CCLE"
```

With `jq`:

```bash
curl -s "https://ampliconrepository.org/api/v1/projects/?name=CCLE" \
  | jq -r '.[] | [.project_name, .id, .sample_count, .visibility] | @tsv'
```

## Download one project

Each project has a stable project ID. Use that ID in the download endpoint:

```bash
PROJECT_ID="64a1b2c3d4e5f6a7b8c9d0e1"

curl -L -O "https://ampliconrepository.org/api/v1/projects/${PROJECT_ID}/download/"
```

The `-L` option is important because production downloads may redirect to a short-lived storage URL. The `-O` option tells `curl` to save the remote archive to a file.

The downloaded file is a `.tar.gz` archive containing the project output packaged by AmpliconSuiteAggregator.

## Private projects

Private projects require an API token.

To create a token:

1. Log in at [ampliconrepository.org](https://ampliconrepository.org).
2. Open your Profile page.
3. Under **Developer API Token**, choose **Generate / Regenerate Token**.
4. Copy the token when it is shown.

Use the token in the `Authorization` header:

```bash
TOKEN="paste-your-token-here"
PROJECT_ID="64a1b2c3d4e5f6a7b8c9d0e1"

curl -L -O "https://ampliconrepository.org/api/v1/projects/${PROJECT_ID}/download/" \
  -H "Authorization: Token ${TOKEN}"
```

The same token also lets you list private projects where you are a project member:

```bash
curl "https://ampliconrepository.org/api/v1/projects/" \
  -H "Authorization: Token ${TOKEN}"
```

## Download several projects

The batch endpoint resolves project IDs to download URLs. Projects that are missing, inaccessible, or do not have an archive are returned in `skipped`.

```bash
TOKEN="paste-your-token-here"

curl -s -X POST "https://ampliconrepository.org/api/v1/projects/download/" \
  -H "Authorization: Token ${TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"ids": ["id1", "id2", "id3"]}'
```

With `jq`, download all resolved archives:

```bash
TOKEN="paste-your-token-here"

curl -s -X POST "https://ampliconrepository.org/api/v1/projects/download/" \
  -H "Authorization: Token ${TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"ids": ["id1", "id2", "id3"]}' \
  | jq -r '.downloads[].download_url' \
  | while read -r url; do
      curl -L -O "$url" -H "Authorization: Token ${TOKEN}"
    done
```

For public projects, omit the `Authorization` headers.

## Inspect project metadata and samples

Project metadata:

```bash
curl "https://ampliconrepository.org/api/v1/projects/${PROJECT_ID}/"
```

Sample metadata:

```bash
curl "https://ampliconrepository.org/api/v1/projects/${PROJECT_ID}/samples/"
```

Add the token header for private projects:

```bash
curl "https://ampliconrepository.org/api/v1/projects/${PROJECT_ID}/samples/" \
  -H "Authorization: Token ${TOKEN}"
```

## Common errors

| Status | Meaning |
| --- | --- |
| `400` | Bad request, such as a batch request where `ids` is not a JSON array. |
| `401` | A token is missing, invalid, or not authorized for the private project. |
| `404` | The project or downloadable archive was not found. |
| `503` | The archive is temporarily unavailable from storage. Try again later. |

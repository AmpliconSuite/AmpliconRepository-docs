# Adding Results to AmpliconRepository

Sharing and exploring focal amplification data on [AmpliconRepository.org](https://ampliconrepository.org) is designed to be straightforward.

**TL;DR for Cohort Users:** If you have run [AmpliconSuite-pipeline](https://github.com/AmpliconSuite/AmpliconSuite-pipeline) on a cohort of samples, simply package the resulting output directory into a `.tar.gz` or `.zip` file and upload it. The backend will automatically handle the discovery and aggregation of all samples in your cohort.

---

### 1. Generate focal amplification predictions

Data on AmpliconRepository typically comes from the **AmpliconSuite** ecosystem. The recommended workflow is to use [AmpliconSuite-pipeline](https://github.com/AmpliconSuite/AmpliconSuite-pipeline), which integrates:
* **Data Preparation** (CNV calling via CNVkit or other tools)
* **AmpliconArchitect (AA)** (Focal amplification reconstruction)
* **AmpliconClassifier (AC)** (Classification of amplicon types like ecDNA or BFB)

Once your pipeline runs are complete, you will have directories containing `_AA_results`, `_classification`, and `_cnvkit_output`.

**Long reads?** [CoRAL](https://github.com/AmpliconSuite/CoRAL) reconstructions are also accepted. Run
AmpliconClassifier (version 2.0.0 or newer) on your CoRAL output to produce the `_classification`
directory, then package as described in [CoRAL results](#coral-long-read-results) below.

---

### 2. Package your results

To upload data, you must package your results into one or more `.tar.gz` or `.zip` archives. When you upload these files, **the AmpliconRepository backend automatically discovers and aggregates your data.**

The backend is designed to be flexible: it walks your directory tree and identifies samples based on folder suffixes and the contents of your classification tables.

#### Key Components
For the backend to correctly link your data, ensure the following components are present in your archive:

1.  **Authoritative Sample List:** The backend uses **`*_result_table.tsv`** files (produced by AmpliconClassifier) as the source of truth. It will include every sample listed in these tables that it can find matching data for.
2.  **Reconstructions:** Directories ending in **`_AA_results`** containing the AmpliconArchitect
    reconstructions. A **CoRAL** per-sample directory works too — it is recognized by the
    **`*_summary.txt`** / **`*_amplicon_summary.txt`** file it contains, so it does not need to be
    renamed to `_AA_results`.
3.  **Classification Data:** Directories or files containing the AmpliconClassifier classification outputs.
4.  **CNV Calls:** Directories ending in **`_cnvkit_output`** (or `_cnvkit_outputs`) containing the `.bed` files, or generally speaking any directory(s) containing the whole genome CNV call files named like [sample_name]_CNV_CALLS.bed

#### Flexible Organizational Styles
You can organize your files in the way that best fits your workflow:

*   **Per-Sample Organization:** Each sample has its own set of directories (e.g., `sample1_AA_results/`, `sample1_classification/`, etc.).
*   **Project-Wide Organization:** You can have a single, consolidated classification directory for the entire project, while keeping AA and CNVkit results in separate folders.
*   **Nested Folders:** The backend will recursively search through your archive, so you can group samples into subfolders (e.g., by cohort or batch).

#### Example Structure (Mixed Style)
The following is just one example of a valid hierarchy:

```text
my_study/
├── cohort_A/
│   ├── sample1_AA_results/
│   ├── sample1_cnvkit_output/
│   ├── sample2_AA_results/
│   └── sample2_cnvkit_output/
└── project_classification/      <-- Consolidated classifications
    ├── sample1_result_table.tsv
    ├── sample2_result_table.tsv
    └── ...
```

#### How to create your archive
Package your results from the command line, running from *outside* your data directory.
**Important:** do **not** include original `.bam`, `.fastq`, or `.cram` files.

```bash
tar -czf my_project.tar.gz \
    --exclude='*.bam*' \
    --exclude='*.cram*' \
    --exclude='*.fastq*' \
    --exclude='*.fq*' \
    my_study/
```

Or, if you prefer `.zip`:

```bash
zip -rq my_project.zip my_study/ \
    -x '*.bam*' '*.cram*' '*.fastq*' '*.fq*'
```

#### CoRAL (long-read) results

CoRAL output is packaged the same way, with two things to watch for.

**Directory layout.** CoRAL writes everything for a sample into one flat directory. Move the CNVkit segment file into a
`cnvkit_output/` subdirectory so the backend can find it — copy-number files sitting loose next to
the reconstructions are **not** picked up:

```text
SAMPLE/
├── SAMPLE_amplicon1_graph.txt
├── SAMPLE_amplicon1_cycles.txt
├── SAMPLE_amplicon1_graph.png       (and .pdf)
├── SAMPLE_amplicon1_cycles.png      (and .pdf)
├── SAMPLE_amplicon_summary.txt      <-- marks this as a results directory
├── SAMPLE_CNV_SEEDS.bed
├── SAMPLE_reconstruct.log
└── cnvkit_output/
    └── SAMPLE.cns                   <-- CNV calls
```

!!! note
    The backend converts the plain `.cns` into the `_CNV_CALLS.bed` it displays. The
    `.call.cns` and `.bintest.cns` files are not used, and `SAMPLE_cnvkit_output/` works
    just as well as a bare `cnvkit_output/`.

**Leave out the intermediates.** A finished CoRAL run keeps several large working files that AmpliconRepository never reads. Excluding
them is worth the effort — on one test sample the archive dropped from **158 MB to under 500 KB**:

| Pattern | What it is | Typical size |
|---|---|---|
| `*_chimeric_alignments.pickle` | cached chimeric alignments | 100s of MB |
| `*.cnn` | CNVkit target / antitarget coverage | 10s of MB |
| `*.cnr` | CNVkit per-bin copy ratios | 10s of MB |
| `*-tmp.bed` | CNVkit reference scratch files | 10s of MB |
| `models/` | Pyomo/Gurobi solver models and logs | varies |

```bash
tar -czf my_project.tar.gz \
    --exclude='*.pickle' \
    --exclude='*.cnn' \
    --exclude='*.cnr' \
    --exclude='*-tmp.bed' \
    --exclude='models' \
    --exclude='*.bam*' \
    --exclude='*.cram*' \
    --exclude='*.fastq*' \
    --exclude='*.fq*' \
    my_study/
```

Or with `zip`:

```bash
zip -rq my_project.zip my_study/ \
    -x '*.pickle' '*.cnn' '*.cnr' '*-tmp.bed' '*/models/*' \
       '*.bam*' '*.cram*' '*.fastq*' '*.fq*'
```

Keep the `_reconstruct.log` — it is compressed on the server and carries the CoRAL version. And
remember that AmpliconClassifier still has to be run on the CoRAL output: the resulting
`*_result_table.tsv` is what the backend uses as its list of samples.

---

### 3. Create or Update a Project

Once your archives are ready, head to [AmpliconRepository.org](https://ampliconrepository.org).

#### Creating a New Project
1. **Log in** and select **"New Project"** from your account menu.
2. **Project Name & Alias:** Choose a descriptive name. You can also set a **Project Alias** (e.g., `my-study-2024`) to create a clean, shareable URL: `ampliconrepository.org/project/my-study-2024/`.
3. **Upload Archives:** You can select **multiple** `.tar.gz` files at once. The server will aggregate them into a single project automatically.
4. **Submit:** After clicking submit, you will be redirected to the project page. 

**Note:** Large projects are processed in the **background**. You can leave the page or close your browser; the project will update automatically once processing is complete.

#### Updating or Adding Samples to a Project
You can update your project or add new data at any time by selecting **"Edit Project"** from the project page.

When adding data, you can choose from three modes:
*   **Append Samples (Add Data):** Use this to add new `.tar.gz` files to your existing project. The new samples will be aggregated alongside your current data. This is the standard way to grow a project as more samples are processed.
*   **Replace Project:** Use this to wipe all current samples and replace them entirely with the newly uploaded files.
*   **Re-aggregate:** Re-process the existing data on the server. This is useful if you want to apply new metadata (like sample renaming) to data that is already uploaded without re-uploading the archives.

**Steps to add samples:**
1. Navigate to your project page and click the **Edit (pencil icon)** button.
2. Under the **"Upload"** section, select your new `.tar.gz` or `.zip` archives.
3. Ensure the **"Append Samples"** mode is selected (this is usually the default).
4. Click **Submit**. The backend will process the new samples and add them to the project view once complete.


---

### 4. Add Metadata & Rename Samples

You can upload a metadata file (CSV, TSV, or XLSX) to annotate your samples.

**Metadata Requirements:**
* **First Column:** Must be `sample_name` (must match the names in your uploaded data).
* **Second Column:** Must be `cancer_type`.
* **Additional Columns:** Any other annotations (Tissue, Stage, Treatment, etc.).

#### Deep Renaming with Aliases
If your metadata includes a column named **`sample_name_alias`**, the repository can perform a "Deep Rename." This physically renames the samples in the database and across all internal files (tables, logs, etc.) to match your aliases, making the project much easier for others to navigate.

---

## Need help?

If you encounter issues during upload or have questions about data formatting, please reach out on our [GitHub issues page](https://github.com/AmpliconSuite/AmpliconRepository/issues). We are happy to help!

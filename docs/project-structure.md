# Project Archive Structure

When you download an entire project from AmpliconRepository, you receive a compressed archive (`.tar.gz`) with the standardized structure described below. The site's backend creates this structure automatically by discovering and aggregating results from the loosely organized files you upload.

Your uploaded archive does **not** need to follow this download structure. See [Adding Results to AmpliconRepository](getting-started.md) for the supported input layouts and packaging instructions.

## Root Directory: `results/`

The top-level directory contains the following key files:

*   **`aggregated_results.csv`**: A flat, tabular summary of all samples and features in the project. This is the best starting point for analyzing the dataset in Excel or R.
*   **`aggregated_results.html`**: A searchable, interactive HTML table of your results.
*   **`run.json`**: A machine-readable JSON file containing all metadata and relative file paths for the project.

## Sample Data: `samples/`

The `samples/` directory contains a subdirectory for every sample in the project. Inside each sample folder (e.g., `samples/sample1/`), you will find:

*   **`sample1_AA_results/`**: An uncompressed directory containing the raw output from AmpliconArchitect (AA), including:
    *   **`*_summary.txt`**: A text summary of the amplicons found in the sample.
    *   **`*_cycles.txt` & `*_graph.txt`**: The bioinformatic reconstructions of each amplicon.
    *   **`*.pdf` & `*.png`**: Visualizations of the amplicon structures.
*   **`sample1_cnvkit_output.tar.gz`**: A compressed archive of the CNVkit results.
*   **`sample1_CNV_CALLS.bed`**: An uncompressed BED file containing the whole-genome copy number calls used to identify candidate focal amplification regions; AA independently re-estimates copy number within those regions.
*   **Metadata & Logs**:
    *   `sample1_run_metadata.json` / `sample1_sample_metadata.json`
    *   `sample1.log`: The pipeline execution log for this sample.
    *   `sample1_timing_log.txt`: Performance metrics for the run.

## Consolidated Analysis: `consolidated_classification/`

This directory aggregates results from **AmpliconClassifier (AC)** across all samples in the project, using the project name as a prefix.
More about these files is available from the [AC GitHub Readme](https://github.com/AmpliconSuite/AmpliconClassifier/blob/main/README.md#3-outputs).

*   **`*_result_table.tsv`**: The authoritative list of all focal amplifications identified across the project.
*   **`*_amplicon_classification_profiles.tsv`**: Detailed profiles for each identified amplicon.
*   **`*_gene_list.tsv`**: A comprehensive list of genes associated with each identified feature.
*   **`*_ecDNA_counts.tsv`**: Summary counts of ecDNA identified in the project.
*   **`*_ecDNA_context_calls.tsv`**: Data regarding the genomic context of identified ecDNA.
*   **`*_feature_basic_properties.tsv` & `*_feature_entropy.tsv`**: Metrics regarding the complexity and properties of identified features.
*   **Subdirectories**:
    *   **`*_annotated_cycles_files/`**: Contains cycle files annotated with gene information.
    *   **`*_classification_bed_files/`**: Contains BED files for each identified feature.
    *   **`*_SV_summaries/`**: Summaries of structural variants associated with the features.

## Other Files: `other_files/`

If you included supplementary files in an `AUX_DIR` during upload (e.g., additional metadata or ID mappings), those files will be consolidated here.

## Sample Download Structure

The **Download sample** action on a sample page creates a `.zip` file containing that sample's available results. Unlike a project download, the sample files are placed directly at the root of the archive:

```text
sample1.zip
├── sample1_result_data.tsv
├── sample1_sample_metadata.json
├── sample1_CNV_CALLS.bed
├── <reconstruction-directory>.tar.gz
├── sample1_amplicon1_graph.txt
├── sample1_amplicon1_cycles.txt
├── sample1_classification_bed_files/
│   └── <feature_id>.bed
└── sample1_sashimi_plots/
    ├── sample1_amplicon1.png
    ├── sample1_amplicon1.pdf
    ├── sample1_amplicon1_cycles.png
    └── sample1_amplicon1_cycles.pdf
```

The archive contents depend on which results are available for the sample:

*   **`*_result_data.tsv`**: One row per classified feature. File-reference columns contain relative paths to the corresponding files in the ZIP.
*   **`*_sample_metadata.json`**: Sample-level metadata.
*   **`*_CNV_CALLS.bed`**: Whole-genome copy number calls used to identify candidate focal amplification regions. AA independently re-estimates copy number within those regions and does not use these calls internally.
*   **`*_classification_bed_files/`**: One BED file per classified feature.
*   **`*_sashimi_plots/`**: Available graph and cycle visualizations in PNG and PDF formats.
*   **`*_graph.txt` and `*_cycles.txt`**: Available reconstruction graph and cycle files.
*   **`<reconstruction-directory>.tar.gz`**: The complete AA or CoRAL reconstruction-results directory. Newer projects preserve the original directory name; older projects may use `aa_directory.tar.gz`.

Batch sample downloads contain the same per-sample files, nested as `<project_name>/<sample_name>/` within a `batch_samples_<timestamp>.zip` archive.

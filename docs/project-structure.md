# Project Structure

When you download a project or a selection of samples from AmpliconRepository, you receive a compressed archive (`.tar.gz`). This archive is organized by the **AmpliconSuiteAggregator** into a standardized structure designed for both human readability and programmatic analysis.

---

### Root Directory: `results/`

The top-level directory contains the following key files:

*   **`aggregated_results.csv`**: A flat, tabular summary of all samples and features in the project. This is the best starting point for analyzing the dataset in Excel or R.
*   **`aggregated_results.html`**: A searchable, interactive HTML table of your results.
*   **`run.json`**: A machine-readable JSON file containing all metadata and relative file paths for the project.

---

### Sample Data: `samples/`

The `samples/` directory contains a subdirectory for every sample in the project. Inside each sample folder (e.g., `samples/sample1/`), you will find:

*   **`sample1_AA_results/`**: An uncompressed directory containing the raw output from AmpliconArchitect (AA), including:
    *   **`*_summary.txt`**: A text summary of the amplicons found in the sample.
    *   **`*_cycles.txt` & `*_graph.txt`**: The bioinformatic reconstructions of each amplicon.
    *   **`*.pdf` & `*.png`**: Visualizations of the amplicon structures.
*   **`sample1_cnvkit_output.tar.gz`**: A compressed archive of the CNVkit results.
*   **`sample1_CNV_CALLS.bed`**: An uncompressed BED file containing the copy number calls used by AA.
*   **Metadata & Logs**:
    *   `sample1_run_metadata.json` / `sample1_sample_metadata.json`
    *   `sample1.log`: The pipeline execution log for this sample.
    *   `sample1_timing_log.txt`: Performance metrics for the run.

---

### Consolidated Analysis: `consolidated_classification/`

This directory aggregates results from **AmpliconClassifier (AC)** across all samples in the project, using the project name as a prefix.

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

---

### Other Files: `other_files/`

If you included supplementary data in an `AUX_DIR` during upload (e.g., FISH images, pathology reports), those files will be consolidated here.

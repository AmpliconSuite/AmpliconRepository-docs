# Analysis Tools

AmpliconRepository.org provides powerful tools for searching, filtering, and analyzing focal amplification data across projects and samples.

---

### Unified Search

The search functionality allows you to find specific genes, projects, or classifications across the entire repository. You can access the search via the **Search** button in the navigation bar.

#### Advanced Search Features
*   **Wildcards:** Use `*` to match partial names. For example, `CCLE*` will find all projects starting with CCLE, and `MYC*` will find all genes in the MYC family.
*   **Logic Operators:** 
    *   Use `|` for OR logic (e.g., `MYC|EGFR` finds amplifications containing either gene).
    *   Use `&` for AND logic (e.g., `MYC&EGFR` finds amplifications containing both genes).
*   **Metadata Search:** Collapse the metadata section to search by:
    *   **Sample Type:** (e.g., "cell line", "primary tumor")
    *   **Cancer Type or Tissue:** (e.g., "Glioblastoma", "Lung")

#### Filtering Results
In the search results page, you can further refine the list of samples using the **Project Filters** sidebar. This allows you to toggle specific projects on or off to focus your analysis on specific cohorts.

---

### Co-amplification Graph Analysis

The **Co-amplification Graph** tool (available in the navigation bar) allows you to visualize significant gene-gene associations across multiple projects.

#### How to use the tool:
1.  **Select Projects:** Select one or more projects from the list. 
    *   **Note:** All selected projects must use the same reference genome group (hg19/GRCh37 OR hg38/GRCh38).
2.  **Generate Graph:** Click "Generate Coamplification Graph."
3.  **Adjust Thresholds:** Use the sliders to filter the graph:
    *   **Significance Threshold:** Filter edges based on the chi-squared significance score (q-value).
    *   **Co-amplification Frequency:** Filter edges based on how often the genes appear together in the same amplicons.
4.  **Visualize:** Interact with the graph to see how genes cluster together.

#### Data Export
You can download the graph data as a **CSV file** containing the edges and their associated scores for further offline analysis.

---

### Batch Downloads

You can download large numbers of samples at once directly from the search results page.

1.  Perform a search to find your samples of interest.
2.  Use the **Select All** or individual checkboxes to select samples.
3.  Click **Download Selected Samples**.

**Large Downloads:**
For batches larger than 100 samples, the system will process your request in the background. Once the zip archive is ready, you will receive an **email with a secure download link** (S3 presigned URL).

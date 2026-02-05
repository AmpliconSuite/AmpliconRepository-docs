# Getting Started

### 1. Generate focal amplification predictions with AmpliconSuite-pipeline

AmpliconSuite-pipeline wraps data preparation, AmpliconArchitect, and AmpliconClassifier into a single workflow with our recommended best practices and filtering methods.

* For usage options and documentation, see the [AmpliconSuite-pipeline GitHub page](https://github.com/AmpliconSuite/AmpliconSuite-pipeline)
* After running AmpliconSuite-pipeline, you'll have a collection of output files for one or more samples

### 2. Package your results for upload

Once you have one or more samples processed with AmpliconSuite-pipeline, package the output files into a single `.tar.gz` archive.

**Your archive should include:**
* AmpliconArchitect (AA) output directories
* AmpliconClassifier (AC) output directories  
* CNVkit output directories (or whole-genome copy number bed files)
* *(Optional)* A directory of auxiliary files (e.g., FISH images) with an `AUX_DIR` marker file in the top level

**Important:** Do not include original `.bam` or `.fastq` files in your archive.

**To create your archive:**
```bash
tar --exclude='*.gz' --exclude='*.bam*' -czf project_name.tar.gz your_consolidated_directory/
```

This command will:
* Package your directory into a compressed `.tar.gz` file
* Automatically exclude `.bam` files and pre-compressed `.gz` files
* Preserve the directory structure needed by AmpliconRepository

**Example directory structure before packaging:**
```
my_project/
├── sample1_AA/
├── sample1_classification/
├── sample1_CNVkit/
├── sample2_AA/
├── sample2_classification/
└── sample2_CNVkit/
```

### 3. Upload to AmpliconRepository

1. **Log in** to [AmpliconRepository.org](https://ampliconrepository.org)
2. Click your **username** in the upper-right corner and select **"New Project"**
3. **Fill out the project form:**
   * **Project Name:** A descriptive title for your project
   * **Description:** Explain the data source and what it represents. Include citation information if relevant
   * **Upload:** Select your `.tar.gz` file
4. **Submit** your project

**Note:** Large files (>500MB) may take several minutes to upload. For files larger than 1GB, please [contact the site administrators](https://github.com/AmpliconSuite/AmpliconRepository/issues) for assistance.

### 4. Add metadata (optional)

You can optionally upload a metadata file (CSV, TSV, or XLSX) containing sample annotations:
* First column must be `sample_name`
* Second column must be `cancer_type`
* Additional columns can include any sample-specific metadata

See the upload page for a template and detailed formatting requirements.

---

## Need help?

Have questions, want to request a dataset, or report an issue? Please reach out on our [GitHub issues page](https://github.com/AmpliconSuite/AmpliconRepository/issues). We're very responsive!

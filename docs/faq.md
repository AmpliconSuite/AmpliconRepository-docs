# FAQ

We provide a user guide with [FAQs here](https://github.com/AmpliconSuite/AmpliconSuite-pipeline/blob/master/documentation/GUIDE.md).

### Repository Features

**Why does my project name say "(Processing...)"?**
Large project uploads are processed in the background using the AmpliconSuiteAggregator. This includes validating files, extracting metadata, and building the search index. The project will remain in the "Processing" state until this is complete. You do not need to keep your browser window open.

**I selected many samples for batch download but didn't get a file immediately. What happened?**
For batch downloads exceeding 100 samples, the server prepares the zip archive in the background to ensure stability. Once the archive is ready, you will receive an email (to the address associated with your account) containing a secure link to download the data.

**Can I rename my samples after I've already uploaded them?**
Yes. You can perform a "Deep Rename" by uploading a metadata file that includes a `sample_name_alias` column. Go to the "Edit Project" page, upload your metadata, and check the "Remap sample names" option. This will update the sample names across all database records and internal files.

### Technical Questions
**What's the difference between an AA amplicon and an ecDNA, or other focal amplification?**

   - AmpliconArchitect uses the term "amplicon" to represent a bioinformatic concept of a collection of genome intervals linked by structural variants or having close proximity in the linear genome (thus being amplified together). Inside the regions comprising an "amplicon", there may be various focal amplifications. Each focal amplification identified by AmpliconClassifier is termed a "feature". Within an amplicon there may be multiple distinct focal amplifications. E.g., an ecDNA and a BFB or two different ecDNA, etc. AmpliconClassifier would then ID these as `[sampleX]_amplicon1_ecDNA_1` (first ecDNA found in amplicon1 of sampleX).

**How do I interpret the AA cycles file? Are the entries each an ecDNA?**

   - The contents of the AA cycles file represent bioinformatic decompositions of the AA amplicon. In most cases each entry in that file is a bioinformatic substructure of the true focal amplification structure. Obtaining a completely resolved structure for a focal amplification from short read data is typically not possible unless there is a trivial structure to the focal amplification, there are no SVs missed by the short reads, and heterogeneity is minimal. Consequently, users should think carefully before visualizing these cycles using CycleViz.


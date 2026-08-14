# What is AmpliconRepository? {.overview-title}

AmpliconRepository is a community resource for sharing and exploring focal amplifications detected in cancer whole-genome sequencing (WGS) datasets.

Repository results are generated with tools in the [AmpliconSuite](https://github.com/AmpliconSuite) toolkit: [AmpliconArchitect](https://github.com/AmpliconSuite/AmpliconSuite-pipeline) reconstructs focal amplifications from short-read WGS data, [CoRAL](https://github.com/AmpliconSuite/CoRAL) reconstructs them from long-read WGS data, and [AmpliconClassifier](https://github.com/AmpliconSuite/AmpliconClassifier) classifies the reconstructed structures. AmpliconRepository brings these reconstructions, classifications, copy-number profiles, and genomic annotations together in a centralized platform for searching, visualization, and download.

**Key Features:**

*   **Unified Search:** Find genes, projects, or specific amplicon types (ecDNA, BFB, etc.) with advanced wildcard and logic support.
*   **Co-amplification Analysis:** Visualize gene-gene associations and clusters across multiple projects using our interactive graph tool.
*   **Bulk Data Access:** Download high-volume sample data in batches, with background processing and email delivery for large requests.
*   **Background Processing:** Upload large datasets and manage project versions with automatic background aggregation.

## Navigation

*   [**Adding Results to AmpliconRepository**](getting-started.md): Instructions on packaging and uploading your data.
*   [**API**](api.md): List public projects, inspect metadata, and download project archives from the command line.
*   [**Analysis Tools**](analysis-tools.md): Detailed guide on using the search, filtering, and graph analysis tools.
*   [**Project Archive Structure**](project-structure.md): Description of the standardized archive created automatically from uploaded results.
*   [**FAQ**](faq.md): Common questions and troubleshooting.
*   [**Team & Contributors**](contributors.md): The people who have developed and guided AmpliconRepository.

## Contact & Support

For technical questions, bug reports, or feature requests, please use the [GitHub issues page](https://github.com/AmpliconSuite/AmpliconRepository/issues). For direct inquiries, contact:

- Jens Luebeck: jluebeck@ucsd.edu
- Vineet Bafna: vbafna@ucsd.edu

See [**Team & Contributors**](contributors.md) for the people who have built and guided AmpliconRepository.

View the [**AmpliconSuite GitHub Organization**](https://github.com/AmpliconSuite) to learn more about the available tools in AmpliconSuite.

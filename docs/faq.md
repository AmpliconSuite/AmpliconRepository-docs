# FAQ

We provide a user guide with [FAQs here](https://github.com/AmpliconSuite/AmpliconSuite-pipeline/blob/master/documentation/GUIDE.md).

A couple of the more common questions are below:

- What's the difference between an AA amplicon and an ecDNA, or other focal amplification?

   - AmpliconArchitect uses the term "amplicon" to represent a bioinformatic concept of a collection of genome intervals linked by structural variants or having close proximity in the linear genome (thus being amplified together). Inside the regions comprising an "amplicon", there may be various focal amplifications. Each focal amplification identified by AmpliconClassifier is termed a "feature". Within an amplicon there may be multiple distinct focal amplifications. E.g., an ecDNA and a BFB or two different ecDNA, etc. AmpliconClassifier would then ID these as `[sampleX]_amplicon1_ecDNA_1` (first ecDNA found in amplicon1 of sampleX).

- How do I interpret the AA cycles file? Are the entries each an ecDNA?

   - The contents of the AA cycles file represent bioinformatic decompositions of the AA amplicon. In most cases each entry in that file is a bioinformatic substructure of the true focal amplification structure. Obtaining a completely resolved structure for a focal amplification from short read data is typically not possible unless there is a trivial structure to the focal amplification, there are no SVs missed by the short reads, and heterogeneity is minimal. Consequently, users should think carefully before visualizing these cycles using CycleViz.

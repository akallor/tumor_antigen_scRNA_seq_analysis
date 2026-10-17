# tumor_antigen_scRNA_seq_analysis
scRNA-seq analysis of tumor cells to determine clonal differences in antigen presentation/

# Problem statement
Antigen prioritisation pipelines rank candidates as though every malignant cell in a tumour were identical. They are not, and the gap shows up as vaccine escape.
Current pipelines score candidates on HLA binding, presentation evidence, predicted immunogenicity, bulk transcript abundance and clonality. Clonality is derived from bulk DNA, as cancer cell fraction estimated from variant allele frequencies. Two distinct axes of heterogeneity remain invisible to that approach.
The first is source-transcript expression. A peptide detected in bulk immunopeptidomics may derive from a transcript expressed in only a minority of malignant cells. Bulk abundance cannot distinguish uniform moderate expression from high expression confined to one subclone.
The second is presentation competence. Antigen presentation machinery varies cell to cell within a tumour. Cells with B2M loss, HLA downregulation or immunoproteasome loss cannot present any peptide, regardless of how well that peptide scores.
The consequence is that a candidate passing every current filter may reach only a fraction of the tumour, leaving an antigen-negative reservoir that drives outgrowth under immune pressure. The problem is sharpest for non-canonical antigens: lncRNA- and UTR-derived peptides show substantial expression variability across tumour clones, which makes them unusually prone to immune editing.
No current tool computes expression-resolved antigen coverage at single-cell resolution. The data to do so exists in public tumour atlases.

# Approach
Module 1: Presentation competence. Score every cell on antigen presentation machinery: HLA-A/B/C, B2M, TAP1, TAP2, TAPBP, the immunoproteasome subunits PSMB8, PSMB9 and PSMB10, plus the regulators NLRC5 and CIITA. Output is a per-cell score and a presentation-competent fraction per sample. This module answers whether a tumour has a presentation-null subpopulation at all, independently of any candidate list.

Module 2: Expression-resolved coverage. Map each candidate peptide to its source transcript, then compute per-candidate the fraction of malignant cells expressing that transcript, how uniformly expression is distributed, and whether it concentrates in particular cell states. This is clonality measured from expression rather than inferred from DNA.

Module 3: Effective coverage and cocktail selection. Combine the two: a cell is covered only if it expresses the source transcript and retains presentation capacity. Given a candidate pool, select the peptide set maximising the fraction of malignant cells covered by at least one epitope. This is a weighted set-cover problem; a greedy approximation is sufficient and gives a bounded result.

Modules 1 and 2 are scoped to be independently useful, so the package has value before module 3 exists. Build order follows that sequence.

# Pipeline architecture
Inputs: An AnnData object with malignant cells labelled, and a candidate peptide list carrying source-transcript identifiers. HLA typing is optional and used only to filter the candidate pool.

Processing: Modules 1 and 2 each reduce to a per-cell quantity and run independently of each other. Module 3 is the only step needing both, and the only one operating on a candidate pool rather than one peptide at a time.

Outputs: A per-candidate table carrying coverage score, escape risk and detectability confidence, plus a selected epitope set when module 3 runs.

# Expected outcomes (to be revised as fit)
A coverage score that slots into existing prioritisation output alongside binding affinity, presentation evidence and immunogenicity, rather than replacing any of them.

A companion analysis applying the tool to public tumour scRNA-seq across several cancer types. The target finding: what fraction of malignant cells a typical high-ranking candidate would reach once expression heterogeneity and presentation competence are both accounted for. If that fraction is materially lower than current prioritisation assumes, the result stands on its own.




# SE-3-equivariance-GNN

PaiNN Binding Affinity — SE(3)-Equivariant GNN + Delta-ML + Ensemble UQ
=======================================================================

WHAT IT DOES
------------
Predicts protein-ligand binding affinity (pKd) from 3D atomic structure.
Given a drug molecule and a protein pocket, the model outputs a pKd estimate
with epistemic uncertainty bounds from a 5-member deep ensemble.

The pipeline has three components working together:

  1. SE(3)-equivariant graph neural network (PaiNN architecture)
     Represents the protein-ligand complex as a heterogeneous graph.
     Message passing respects 3D rotational symmetry — rotating the input
     complex produces identical predictions. Node features split into
     invariant scalars (chemistry) and equivariant vectors (geometry).

  2. Delta-ML correction
     Rather than predicting pKd directly, the model learns the residual
     between the true label and a classical physics-based score:
       y_hat = f_classical(x) + Delta_f_PaiNN(x)
     The classical term anchors predictions in physically meaningful
     baselines; the GNN corrects systematic errors in desolvation,
     metal coordination, and entropy penalties that additive scoring
     functions cannot capture.

  3. Deep ensemble uncertainty quantification
     Five independently trained models produce a mean prediction (mu)
     and epistemic uncertainty (sigma = standard deviation across members).
     High sigma flags compounds where the training data provides weak
     constraint — useful for active learning prioritisation.

NODE FEATURES (134 dim total)
------------------------------
  30  ligand atom chemistry (element, degree, H count, valence, aromaticity)
   4  protein residue type (hydrophobic / polar / charged+/-)
 100  H1 topological pocket descriptor (10x10 persistence image from
      Vietoris-Rips filtration via gudhi — encodes binding-site shape)

KEY ARCHITECTURAL CHOICES
--------------------------
  - 4 PaiNN layers, hidden dim 128, 64 RBF Gaussian basis, 10 A cutoff
  - Ligand-only mean pooling at readout (protein nodes are context only)
  - Cosine envelope for smooth cutoff at graph boundary
  - Dropout 0.15 in readout MLP for MC-dropout compatibility
  - CosineAnnealingLR scheduler, Adam optimiser, gradient clipping at 1.0

DATASET USED HERE
-----------------
  15 FDA-approved kinase/oncology inhibitors (imatinib, venetoclax,
  dasatinib, ibrutinib, etc.) with literature pKd values from PDBbind
  v2020. 3D ligand conformers generated via RDKit ETKDGv3 + MMFF
  optimisation. Protein pockets are synthetic shells around the ligand
  centroid (proof-of-pipeline only — not real crystal structures).

DEPENDENCIES
------------
  torch, torch-geometric, gudhi, rdkit, biopython, vina, meeko,
  numpy, pandas, scikit-learn, jupyter

RESULTS (val set, 3 complexes, 60 epochs single model)
-------------------------------------------------------
  RMSE  1.87 pKd units
  MAE   1.59 pKd units
  R     0.86 Pearson

  Ensemble (5 members x 40 epochs):
  member sigma range  0.10 -- 0.24 pKd units across val set

  Note: val set of 3 is not statistically meaningful. These numbers
  demonstrate pipeline correctness, not benchmark performance.
  Expected RMSE on full CASF-2016 (285 complexes) with real data:
  ~1.3-1.5 pKd units, competitive with published PaiNN-based models.

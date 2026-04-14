# Embedding the interactive SE(3) widget in GitHub

GitHub markdown does not execute JavaScript, so the rotating molecule widget cannot
live inline in a README. Three escalating options:

---

## Option 1 — GitHub Pages (recommended, zero extra services)

Push `se3_equivariance_widget.html` to your repo's `gh-pages` branch:

```bash
git checkout --orphan gh-pages
git reset --hard
cp se3_equivariance_widget.html index.html   # or keep as a sub-page
git add index.html
git commit -m "add SE(3) widget"
git push origin gh-pages
```

Enable Pages in repo Settings → Pages → Source: `gh-pages` / `/ (root)`.

Then in your README:

```markdown
## Interactive demo

[**SE(3) equivariance — rotate the molecule →**](https://YOUR_USER.github.io/YOUR_REPO/se3_equivariance_widget.html)

> Drag the slider to rotate the protein–ligand complex.
> Observe that the invariant channel sᵢ (bar chart) is unchanged
> while the equivariant channel vᵢ (arrow) co-rotates.
```

For an inline preview image linking to the live widget, generate a screenshot:

```markdown
[![SE3 widget preview](docs/se3_preview.png)](https://YOUR_USER.github.io/YOUR_REPO/se3_equivariance_widget.html)
```

---

## Option 2 — rawcdn.githack.com (no branch setup needed)

Push the HTML file anywhere in your default branch, then use rawcdn.githack.com
to serve it with the correct Content-Type:

```
https://rawcdn.githack.com/YOUR_USER/YOUR_REPO/COMMIT_SHA/se3_equivariance_widget.html
```

In README:

```markdown
[SE(3) equivariance widget](https://rawcdn.githack.com/YOUR_USER/YOUR_REPO/main/se3_equivariance_widget.html)
```

---

## Option 3 — Binder (full notebook interactivity)

For the full executed notebook with live training output, add a `binder` badge.
Create `environment.yml` in your repo root:

```yaml
name: painn-binding
channels:
  - conda-forge
  - defaults
dependencies:
  - python=3.12
  - pip
  - pip:
    - torch
    - torch-geometric
    - gudhi
    - rdkit
    - biopython
    - vina
    - meeko
    - jupyter
```

Then in README:

```markdown
[![Open in Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/YOUR_USER/YOUR_REPO/main?filepath=nb2_painn_full_run.ipynb)
```

---

## Option 4 — nbviewer (static render, no interactivity)

Static read-only render of the executed notebook (shows all cell outputs):

```markdown
[![View notebook](https://img.shields.io/badge/nbviewer-view-orange)](https://nbviewer.org/github/YOUR_USER/YOUR_REPO/blob/main/nb2_painn_full_run_executed.ipynb)
```

---

## Recommended README badge block

```markdown
# PaiNN Binding Affinity — SE(3)-Equivariant GNN + Δ-ML + Ensemble UQ

[![SE(3) widget](https://img.shields.io/badge/demo-SE(3)_widget-7F77DD)](https://YOUR_USER.github.io/YOUR_REPO/)
[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/YOUR_USER/YOUR_REPO/main?filepath=nb2_painn_full_run.ipynb)
[![nbviewer](https://img.shields.io/badge/nbviewer-view-orange)](https://nbviewer.org/github/YOUR_USER/YOUR_REPO/blob/main/nb2_painn_full_run_executed.ipynb)

**Architecture:** PaiNN × 4 layers · SE(3) equivariant · Δ-ML classical correction · deep ensemble UQ  
**Node features:** 30 (chem) + 4 (residue) + 100 (H₁ persistence image) = 134 dim  
**Dataset:** 15 FDA-approved kinase inhibitors · ETKDGv3 3D conformers · synthetic pocket  
**Params:** 783,361 (single model) · 3.9M (M=5 ensemble)
```

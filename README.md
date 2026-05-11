# Automating BMS Metadata Normalization

Weak supervision and transformer token classification for decomposing
Building Management System (BMS) tags into four semantic roles
(`BUILDING`, `SYSTEM`, `COMPONENT`, `POINT_TYPE`).

This repository accompanies the master's thesis *Automating Metadata
Normalization in Building Management Systems: Weak Supervision and
Transformer Token Classification for Semantic Tag Decomposition*
(KTH Royal Institute of Technology, 2026).


## What this does

BMS tags like `01_01_414506_01_LB04_DAM_KOMFEL_AL` are inconsistent
across buildings, vendors, and installation eras, which prevents
analytics across a portfolio. This pipeline:

1. Splits each tag on `_` and `-` into delimiter-separated tokens.
2. Applies ten domain-specific Snorkel labelling functions to assign
   each token a probabilistic label across four semantic classes.
3. Fine-tunes a multilingual transformer on the resulting weak
   labels, with the Swedish description of the parent tag supplied
   as a second input.
4. Produces a four-level JSON decomposition of each tag, with
   low-confidence tokens marked `unsure`.

Headline result on a 631-token, single-annotator gold set:
**~84 % weighted F1**, with the three top multilingual models
(BERT multilingual, DistilBERT multilingual, XLM-RoBERTa large)
converging within a 0.6-point band. Performance is bounded by the
abbreviation dictionary rather than model capacity.

---



## Quick start

1. Place a JSON file of the form `{"tag_string": "Swedish description", ...}`
   in `data/AllTags.json`.
2. Run `Pre_processor.ipynb` end to end. Main outputs:
   - `labeled_tags.json` — every token with predicted class and
     confidence.
   - `low_confidence_tokens.json` — the unique tokens that fall
     below the 0.30 LabelModel confidence threshold.
   - `val_set.json` — the high-confidence subset used as training
     and validation data for the transformer.
3. Run the transformer training script in `training/` with the
   weak labels as targets.

The four output classes use integer labels:

```python
ABSTAIN    = -1
BUILDING   = 0
SYSTEM     = 1
COMPONENT  = 2
POINT_TYPE = 3
```

---

## A note on data

The BMS tag data used in the thesis came from the portfolio of
[Skandia Fastigheter](https://www.skandiafastigheter.se/) and is
not redistributable. The pipeline runs on any JSON of the form
`{tag: description}` where descriptions are in Swedish (or any
language a multilingual transformer encoder handles well).

The four-class taxonomy and the labelling functions are calibrated
for Swedish BMS naming conventions. Applying the pipeline to BMS
data from other countries, vendors, or naming traditions will
likely require extending the abbreviation dictionary in
`abbrevations2.json` and adjusting the structural labelling
functions.

---




## Contact

Oskar Pålhagen — palhagen@kth.se

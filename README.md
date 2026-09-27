# Heterogeneous Data Analysis & String Similarity

This repository contains a Jupyter notebook exploring techniques for
computing similarity between strings and sets - a core building block of
heterogeneous data analysis and record linkage, where the same real-world
entity may appear differently formatted across sources. It covers atomic
string similarity measures, set-based similarity coefficients, and a full
MinHash + Locality Sensitive Hashing (LSH) pipeline for scalable document
similarity search.

## Overview

The notebook is split into two main parts:

### 1. Atomic String Similarity

Implements and compares several classic algorithms for measuring similarity
between individual strings:

- **Minimum Edit Distance (Levenshtein)** — the minimum number of
  single-character insertions, deletions, and substitutions needed to
  transform one string into another, applied to `"INTENTION"` vs
  `"EXECUTION"`. A modified version (`mod_ldist`) adds a configurable
  substitution cost parameter.
- **Jaro Similarity** — a similarity measure based on matching characters
  and transpositions, well-suited to short strings like names.
- **Jaro-Winkler Similarity** — extends Jaro similarity with a bonus for
  strings that share a common prefix.
- **Soundex** — a phonetic encoding algorithm that assigns similar codes to
  words that sound alike, applied to `"EDUCATE"` and `"EVALUATE"`.

These are tested on a shared benchmark set of near-duplicate name pairs
(e.g. `('MARTHA', 'MARHTA')`, `('DIXON', 'DICKSONX')`,
`('JELLYFISH', 'SMELLYFISH')`), a standard set used to illustrate how each
algorithm handles typos, transpositions, and phonetic variation differently.

### 2. Similarity for Sets

Tokenizes short documents into word sets and computes pairwise similarity
using four set-based coefficients:

- **Jaccard Similarity** — intersection over union
- **Sørensen Coefficient** — twice the intersection over the sum of set sizes
- **Tversky Index** — a generalization of Jaccard/Sørensen with asymmetric
  weighting (α, β) for set differences
- **Overlap Coefficient** — intersection over the smaller set's size

Applied to four short "costume gathering" sentences that vary subtly in
wording, to illustrate how each coefficient responds differently to partial
overlap.

### 3. Locality Sensitive Hashing (LSH)

Implements a full pipeline for scalable near-duplicate document detection,
using a sentence-pair dataset pulled from the
[SICK dataset](https://github.com/brmson/dataset-sts):

- **k-Shingling** — converts each sentence into a set of overlapping
  k-character substrings ("shingles")
- **One-hot encoding** — builds a vocabulary across all shingles and
  represents each document as a sparse one-hot vector
- **MinHashing** — compresses each sparse vector into a compact, fixed-length
  signature using random hash permutations, approximating Jaccard similarity
  between the original shingle sets
- **LSH banding** — splits each signature into bands and hashes them into
  buckets, so that only documents sharing a bucket are flagged as
  "candidate pairs" for further comparison — avoiding an all-pairs
  comparison across the full dataset

## Key findings

- Levenshtein distance with the default substitution cost of 2 gives a
  distance of 8 between "INTENTION" and "EXECUTION"; using a substitution
  cost of 1 instead gives 5, since substitutions become relatively cheaper
  compared to insertions/deletions.
- Jaro-Winkler consistently scores prefix-sharing pairs (e.g.
  `('MARTHA', 'MARHTA')`) higher than plain Jaro similarity, since it adds
  a bonus for shared prefixes — while pairs with no shared prefix (e.g.
  `('DATASCIENCE', 'SCIENCEDATA')`) score identically under both methods.
- MinHash signature similarity closely approximates the true Jaccard
  similarity computed directly on the shingle sets, confirming the
  signatures preserve similarity information despite being far more compact.
- The LSH banding step successfully narrows down candidate similar sentence
  pairs from the full sentence collection without needing to compare every
  possible pair — demonstrating the core efficiency gain LSH provides at scale.

## Requirements

This project uses Python with the following packages:

```bash
pip install pandas numpy requests
```

- `pandas` / `numpy` — data handling and array operations
- `requests` — fetching the SICK sentence dataset directly from GitHub

All similarity algorithms (Levenshtein, Jaro, Jaro-Winkler, Soundex,
Jaccard, Sørensen, Tversky, Overlap, MinHash, LSH) are implemented from
scratch in the notebook — no specialized similarity library is required.

## Repository structure

```
.
├── String_similarity_heterogeneous_data.ipynb   # Jupyter notebook source
├── String_similarity_heterogeneous_data.pdf      # Rendered PDF export of the notebook
└── README.md
```

## Usage

Open `String_similarity_heterogeneous_data.ipynb` in Jupyter or Google Colab
and run all cells top to bottom. The notebook was authored and exported from
Google Colab, using `nbconvert` and `xelatex` to produce the accompanying PDF.

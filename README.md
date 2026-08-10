
# Functional Group Profiling of Drug Molecules Using RDKit, Pandas, and Seaborn

A cheminformatics analysis pipeline that scans a dataset of pharmaceutical compounds and automatically detects which key functional groups are present in each molecule — using SMARTS-based substructure matching directly on SMILES strings, with no manual structural inspection required.

## 🧪 The Science

Functional groups are the reactive "building blocks" of a molecule — the specific atom arrangements that drive a compound's chemical behavior, reactivity, and pharmacological profile. Identifying which functional groups a drug candidate carries is a foundational step in medicinal chemistry, informing everything from solubility and metabolic stability to potential toxicity and drug-drug interactions.

This project screens each molecule against **8 pharmacologically relevant functional groups**, defined using SMARTS notation for precise substructure matching:

| Functional Group | SMARTS Pattern |
|---|---|
| Primary Amine | `[NH2]` |
| Hydroxyl | `[OH]` |
| Carboxylic Acid | `C(=O)[OH]` |
| Ester | `C(=O)O[#6]` |
| Amide | `C(=O)[NH]` |
| Ketone | `[#6]C(=O)[#6]` |
| Aromatic Ring | `c1ccccc1` |
| Halogen | `[F,Cl,Br,I]` |

Unlike plain SMILES comparison, SMARTS allows each pattern to describe a precise chemical environment (e.g., distinguishing a free hydroxyl from an oxygen buried inside an ester), making the detection chemically meaningful rather than a superficial text match.

## 📊 Pipeline Architecture & Logic Steps

### 1️⃣ Data Import via Google Drive
* **Mounting:** Connected the notebook directly to Google Drive using `drive.mount()`, then loaded `Drugs.csv` from a specified Drive path into a pandas DataFrame (`df`).

### 2️⃣ Column Selection & Preparation
* **Reduction:** Reduced the dataset to the two columns required for structural analysis — `Compound ID` and `smiles` — copied into an independent working DataFrame (`readydf`) using `.copy()` to avoid modifying the original data.

### 3️⃣ SMILES Validation & Molecule Conversion
* **Defensive Parsing:** Built a custom `check_safety_import_mol()` function that validates each SMILES entry's type and content (rejecting non-strings and empty values) before attempting conversion.
* **Safe Conversion:** Wrapped `Chem.MolFromSmiles()` in a `try/except` block, converting invalid entries to `None` rather than allowing a single malformed SMILES to break the pipeline.

### 4️⃣ SMARTS-Based Functional Group Detection
* **Pattern Dictionary:** Defined all 8 functional groups as a name → SMARTS-pattern dictionary.
* **Detection Function:** Built `find_all_functional_groups()`, which loops through the dictionary for a given molecule, parses each SMARTS string with `Chem.MolFromSmarts()`, skips any pattern that fails to parse, and checks for a substructure match using `HasSubstructMatch()`.

### 5️⃣ Applying Detection Across the Dataset
* **Vectorized Application:** Used `.apply()` combined with a `lambda` wrapper to run the detection function across the entire `Mol` column, generating a new `Functional_Groups` column — a list of matched functional groups per compound.

### 6️⃣ Frequency Analysis
* **Flattening:** Used `.explode()` to unpack each per-compound list into individual rows.
* **Counting:** Applied `.value_counts()` to calculate how many compounds contain each functional group, producing a clean two-column summary (`group_counts`) ranked by frequency.

### 7️⃣ Data Visualization (Seaborn + Matplotlib)
* **Frequency Bar Chart:** A solid-color `sns.barplot()` visualizing the count of compounds containing each functional group, styled with a bold title and labeled axes, and saved at 300 dpi.

### 8️⃣ Exporting Results
* **Report Generation:** Saved the complete `readydf` — including Compound ID, SMILES, the RDKit Mol object, and the detected Functional Groups list — to `Final_Analysis_Report.csv`.
* **Local Download:** Triggered a direct download of the report to the local machine using Google Colab's `files.download()`.

## 📈 Screening Results & Insights

* **Total compounds screened:** 1,128
* **Most frequent functional group:** Aromatic Ring — present in 565 compounds (**50.09%**)
* **Least frequent functional group:** Carboxylic Acid — detected in 0 compounds (0.00%)

| Functional Group | Molecule Count | % of Dataset |
|---|---|---|
| Aromatic Ring | 565 | 50.09% |
| Halogen | 332 | 29.43% |
| Hydroxyl | 242 | 21.45% |
| Amide | 154 | 13.65% |
| Ester | 135 | 11.97% |
| Ketone | 86 | 7.62% |
| Primary Amine | 76 | 6.74% |
| Carboxylic Acid | 0 | 0.00% |

*Takeaway: Aromatic rings dominate this dataset, appearing in roughly half of all screened compounds — consistent with their well-known prevalence across FDA-approved drugs, where rigid planar ring systems are frequently used to achieve stable binding with biological targets. The complete absence of free carboxylic acid groups is also notable, suggesting this library favors more metabolically stable bioisosteres (such as esters and amides) over the free acid form.*

## 🚀 Repository Contents

* `Drugs.csv` — The raw input dataset containing Compound IDs and SMILES strings prior to analysis.
* `Final_Analysis_Report.csv` — The final processed output featuring each compound's detected functional groups.
* `*.png` — Bar chart visualizing the count of each functional group across the dataset.
* `*.ipynb` — Interactive Google Colab notebook containing the full pipeline, from Drive import through visualization.
* `*.py` — Clean, standalone Python script version of the analysis logic.

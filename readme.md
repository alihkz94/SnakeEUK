# SnakeEUK <img src="snakeeuk.png" alt="SnakeEUK" height="40" align="right">

[![GitHub release](https://img.shields.io/github/v/release/alihkz94/SnakeEUK)](https://github.com/alihkz94/SnakeEUK/releases)
[![Snakemake](https://img.shields.io/badge/snakemake-≥6.0-brightgreen.svg)](https://snakemake.readthedocs.io)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

[![Runs with Conda](https://img.shields.io/badge/runs%20with-conda-brightgreen.svg)](https://conda.io/)
[![Runs with Docker](https://img.shields.io/badge/runs%20with-docker-blue.svg)](https://www.docker.com/)

**Database formatting pipeline for eukaryotic taxonomy FASTA files**

A pipeline for processing taxonomic FASTA files and converting them into formats suitable for multiple downstream tools including General taxonomy, DADA2, Mothur, QIIME2, and SINTAX.EUK

![SnakeEUK logo](snakeeuk.png)

Database formatting pipeline for taxonomy FASTA files.

A pipeline for processing taxonomic FASTA files and converting them into formats suitable for multiple downstream tools including General taxonomy, DADA2, Mothur, QIIME2, and SINTAX.

## Overview

This pipeline is designed to process and clean FASTA files containing taxonomic information. It performs robust encoding conversion (from latin‑1 to ASCII) on the fly, filters and cleans taxonomy headers, and generates outputs tailored for various bioinformatics tools. The workflow is using [Snakemake](https://snakemake.readthedocs.io/) workflow to ensure reproducibility and parallel processing. Each tool-specific script is modular and can be adjusted to meet specific requirements.

## Repository Structure

```plaintext
.
├── Snakefile                 # Snakemake workflow file
├── config.yaml               # Pipeline configuration (versioning, etc.)
├── scripts
│   ├── utils.py              # Utility module for robust file handling (encoding conversion)
│   ├── general.py            # Generates General taxonomy formatted FASTA files
│   ├── dada2.py              # Converts FASTA headers for DADA2 (removes accession numbers)
│   ├── mothur.py             # Generates Mothur-compatible FASTA and TAX files
│   ├── qiime.py              # Generates QIIME2-compatible FASTA and TSV files with cleaned taxonomy fields
│   └── sintax.py             # Converts FASTA files to SINTAX format
└── README.md                 # This README file
```

## Installation & Dependencies

### Requirements

- **Python 3.6+**
- [Biopython](https://biopython.org/)  
  Install via: `pip install biopython`
- [Snakemake](https://snakemake.readthedocs.io/)  
  Install via: `pip install snakemake`
- [Seqkit](https://bioinf.shenwei.me/seqkit/)  
  Install via Conda: `conda install -c bioconda seqkit` or follow instructions on the [Seqkit website](https://bioinf.shenwei.me/seqkit/)

### Installation

Clone the repository and install Python dependencies:

```bash
git clone https://github.com/alihkz94/metagenomics_metabarcoding.git
cd metagenomics_metabarcoding/Snake_EUK
pip install biopython snakemake
```

Ensure that `seqkit` is installed and available in your system PATH.

## Usage

## Input File Format and Requirements

### Expected Input Structure

This pipeline is designed to process taxonomic FASTA files with a specific header format. Each sequence header must contain:

1. **Accession ID**: A unique sequence identifier (e.g., database accession number)
2. **Taxonomic hierarchy**: Semicolon-separated taxonomic levels from Kingdom to Species
3. **Optional metadata**: Additional information after the main taxonomy

**Example header format:**
```
>EUK1157284;Straminipila;Chrysophyta;Chrysophyceae;Chromulinales;Chromulinaceae;Spumella;.;UPD1.8i
>KF849527;Fungi;Glomeromycota;Glomeromycetes;Diversisporales;Acaulosporaceae;Acaulospora;.;Põlme
```

### Taxonomic Hierarchy Structure

The pipeline expects **up to 7 main taxonomic levels** in the following order:
1. **Kingdom** (e.g., Straminipila, Fungi)
2. **Phylum** (e.g., Chrysophyta, Glomeromycota)
3. **Class** (e.g., Chrysophyceae, Glomeromycetes)
4. **Order** (e.g., Chromulinales, Diversisporales)
5. **Family** (e.g., Chromulinaceae, Acaulosporaceae)
6. **Genus** (e.g., Spumella, Acaulospora)
7. **Species** (e.g., species epithet or identifier)

### Placeholder Handling

- **Dot placeholders (`.`)**: Used to indicate unclassified or missing taxonomic levels
- **Empty fields**: Gaps in taxonomic assignment
- **"Unused" designations**: Alternative placeholder text

The pipeline intelligently handles these placeholders by:
- Removing trailing dots and empty fields
- Converting placeholders to "unclassified" labels where appropriate
- Truncating taxonomy at the first meaningful placeholder (DADA2 format)

### Character Encoding and Cleaning

The pipeline includes robust encoding conversion capabilities:

- **Input encoding**: Reads files using `latin-1` encoding to handle special characters
- **Output encoding**: Converts all content to ASCII, replacing problematic characters
- **Quote removal**: Strips quotation marks from headers
- **Special character handling**: Replaces non-ASCII characters with ASCII equivalents

### File Naming Convention

Input files should be named according to the target region or dataset:
- `ITS.fasta` - Internal Transcribed Spacer sequences
- `LSU.fasta` - Large Subunit rRNA sequences  
- `SSU.fasta` - Small Subunit rRNA sequences
- `longread.fasta` - Long-read sequencing datasets

### Input Files

Place your input FASTA files (e.g., `ITS.fasta`, `LSU.fasta`, `SSU.fasta`, `longread.fasta`) in the repository root (or designated input folder).

### How to Prepare Your Database

If you have a custom taxonomic database, format it as follows:

1. **Header structure**: `>AccessionID;Kingdom;Phylum;Class;Order;Family;Genus;Species`
2. **Separator**: Use semicolons (`;`) between taxonomic levels
3. **Missing data**: Use dots (`.`) or leave empty for unclassified levels
4. **Encoding**: Save file with UTF-8 or latin-1 encoding (pipeline handles conversion)
5. **Sequence format**: Standard FASTA format with sequences on separate lines

**Example of properly formatted input:**
```fasta
>ABC123;Fungi;Ascomycota;Eurotiomycetes;Eurotiales;Aspergillaceae;Aspergillus;niger
ATCGATCGATCG...
>DEF456;Plantae;Streptophyta;.;.;.;.;.
GCTAGCTAGCTA...
```

### Understanding Your Database and Construction Guidelines

#### Assessing Your Current Database

Before using this pipeline, examine your existing database to understand its structure:

1. **Check taxonomic depth**: Count the number of semicolon-separated fields
   ```bash
   head -5 your_database.fasta | grep ">" | sed 's/;/\n/g' | wc -l
   ```

2. **Identify placeholder patterns**: Look for common placeholder formats
   ```bash
   grep -o ";[^;]*;" your_database.fasta | sort | uniq -c | head -20
   ```

3. **Check for encoding issues**: Look for non-ASCII characters
   ```bash
   file your_database.fasta  # Should show ASCII or UTF-8
   ```

#### Database Construction Best Practices

**For Maximum Compatibility:**

1. **Standard Format**: Use the format `>AccessionID;Kingdom;Phylum;Class;Order;Family;Genus;Species`
2. **Consistent Separators**: Always use semicolons (`;`) between taxonomic levels
3. **Handle Missing Data**: Use dots (`.`) for unknown/unclassified levels
4. **Avoid Special Characters**: Stick to alphanumeric characters and common punctuation
5. **Quality Control**: Ensure sequences are properly formatted and contain valid nucleotide codes

**Example Database Construction:**

```fasta
# Well-formatted entries
>AB123456;Fungi;Ascomycota;Eurotiomycetes;Eurotiales;Aspergillaceae;Aspergillus;niger
ATCGATCGATCGATCG...

>CD789012;Plantae;Streptophyta;Magnoliopsida;Rosales;Rosaceae;Rosa;.
GCTAGCTAGCTAGCTA...

# Entries with missing higher-level taxonomy (acceptable)
>EF345678;Bacteria;.;.;.;.;Escherichia;coli
TACGTACGTACGTACG...

# Avoid these problematic formats:
# >BadExample1,Fungi,Ascomycota  # Wrong separator
# >BadExample2;Fungi;Ascomycota;  # Trailing separator
# >BadExample3 Fungi Ascomycota   # No separators
```

#### Taxonomic Considerations

**Taxonomic Rank Expectations:**
- The pipeline is optimized for **standard Linnaean hierarchy**
- Supports **7 main taxonomic levels**: Kingdom → Phylum → Class → Order → Family → Genus → Species
- Can handle **fewer levels** but will pad with "unclassified" in General format
- **Additional levels** beyond 7 will be preserved in some formats, truncated in others

**Kingdom-Level Guidelines:**
- Use standard kingdom names (e.g., Fungi, Plantae, Bacteria, Protista)
- For eukaryotic microorganisms, use appropriate supergroup names (e.g., Straminipila, Alveolata)
- Maintain consistency across your database

**Species-Level Considerations:**
- Species names can be binomial (Genus species) or just species epithet
- Use underscores instead of spaces in species names if needed
- Placeholder dots (.) are acceptable for unidentified species

#### Quality Assurance Steps

1. **Validate Headers**: Ensure all headers follow the expected format
2. **Check Taxonomy Completeness**: Verify taxonomic assignments are reasonable
3. **Test Pipeline**: Run on a small subset first to verify output format
4. **Sequence Quality**: Ensure sequences contain only valid nucleotide codes (A, T, G, C, N)
5. **Duplicate Detection**: Check for and remove duplicate sequences if needed

By following these guidelines, you can construct a high-quality database that will work seamlessly with this pipeline and produce reliable outputs for downstream analysis tools.

### Configuration

Edit `config.yaml` to set the desired version string, for example:

```yaml
version: "1.9.4"
```

### Running the Pipeline

Run the entire pipeline with:

```bash
snakemake --cores <number_of_cores> --rerun-incomplete --keep-going
```

To run only for instance the DADA2 conversion over your original FASTA files, execute:

```bash
snakemake dada2/DADA2_EUK_ITS_v1.9.4.fasta dada2/DADA2_EUK_LSU_v1.9.4.fasta dada2/DADA2_EUK_SSU_v1.9.4.fasta dada2/DADA2_EUK_longread_v1.9.4.fasta --rerun-incomplete --keep-going --cores 8
```

> **Tip:** If you suspect output files are outdated or incorrect, you can force re-run of jobs using `--forceall` or `--forcerun <target>`.

### Output Files

- **General:** `general/General_EUK_{base}_v{version}.fasta`
- **DADA2:** `dada2/DADA2_EUK_{base}_v{version}.fasta`  
  *Note:* The DADA2 script removes the accession number and truncates the header at the first placeholder.
- **Mothur:** `mothur/mothur_EUK_{base}_v{version}.fasta` and `mothur/mothur_EUK_{base}_v{version}.tax`
- **QIIME2:** `qiime2/QIIME2_EUK_{base}_v{version}.fasta` and `qiime2/QIIME2_EUK_{base}_v{version}.tsv`  
  *Note:* Taxonomy in the TSV files is cleaned to remove trailing dot placeholders.
- **SINTAX:** `sintax/SINTAX_EUK_{base}_v{version}.fasta`

## Pipeline Processing Details

### Input Transformation by Tool

The pipeline processes the same input file differently for each downstream tool, optimizing the format for specific requirements:

#### 1. General Format (`general/`)
- **Purpose**: Standardized format with complete taxonomic prefixes
- **Processing**: 
  - Adds rank prefixes: `k__`, `p__`, `c__`, `o__`, `f__`, `g__`, `s__`
  - Converts placeholders to "unclassified" labels
  - Ensures all 7 taxonomic ranks are present
- **Example transformation**:
  ```
  Input:  >EUK1157284;Straminipila;Chrysophyta;Chrysophyceae;Chromulinales;Chromulinaceae;Spumella;.
  Output: >EUK1157284;k__Straminipila;p__Chrysophyta;c__Chrysophyceae;o__Chromulinales;f__Chromulinaceae;g__Spumella;s__unclassified
  ```

#### 2. DADA2 Format (`dada2/`)
- **Purpose**: Simplified taxonomy without accession numbers
- **Processing**:
  - **Removes accession ID** (first field)
  - **Truncates at first placeholder** - stops processing when encountering `.` or empty field
  - Maintains only valid taxonomic levels
- **Example transformation**:
  ```
  Input:  >EUK1157284;Straminipila;Chrysophyta;Chrysophyceae;Chromulinales;Chromulinaceae;Spumella;.;UPD1.8i
  Output: >Straminipila;Chrysophyta;Chrysophyceae;Chromulinales;Chromulinaceae;Spumella;
  ```

#### 3. Mothur Format (`mothur/`)
- **Purpose**: Separate FASTA and taxonomy files
- **Processing**:
  - **FASTA file**: Contains only accession IDs as headers
  - **TAX file**: Tab-separated file with accession ID and cleaned taxonomy
  - Removes trailing dot placeholders
- **Example transformation**:
  ```
  FASTA: >EUK1157284
  TAX:   EUK1157284	Straminipila;Chrysophyta;Chrysophyceae;Chromulinales;Chromulinaceae;Spumella
  ```

#### 4. QIIME2 Format (`qiime2/`)
- **Purpose**: QIIME2-compatible FASTA and TSV files
- **Processing**:
  - **FASTA file**: Contains only accession IDs as headers
  - **TSV file**: Two-column format with "Feature ID" and "Taxon" headers
  - Removes trailing dot placeholders and empty fields
- **Example transformation**:
  ```
  FASTA: >EUK1157284
  TSV:   Feature ID	Taxon
         EUK1157284	Straminipila;Chrysophyta;Chrysophyceae;Chromulinales;Chromulinaceae;Spumella
  ```

#### 5. SINTAX Format (`sintax/`)
- **Purpose**: SINTAX/USEARCH compatible format with rank prefixes
- **Processing**:
  - Uses General format as input (requires rank prefixes)
  - Converts rank prefixes: `k__` → `d:`, `p__` → `p:`, `c__` → `c:`, etc.
  - Excludes "unclassified" entries
  - Formats as `tax=d:Kingdom,p:Phylum,c:Class...`
- **Example transformation**:
  ```
  Input:  >EUK1157284;k__Straminipila;p__Chrysophyta;c__Chrysophyceae;o__Chromulinales;f__Chromulinaceae;g__Spumella;s__unclassified
  Output: >EUK1157284;tax=d:Straminipila,p:Chrysophyta,c:Chrysophyceae,o:Chromulinales,f:Chromulinaceae,g:Spumella;
  ```

### Data Quality Improvements

The pipeline automatically handles several data quality issues:

1. **Encoding Problems**: Converts non-ASCII characters to ASCII equivalents
2. **Quote Removal**: Strips quotation marks that may interfere with parsing
3. **Whitespace Normalization**: Trims excess whitespace from taxonomic fields
4. **Sequence Formatting**: 
   - Converts sequences to uppercase (via seqkit)
   - Removes line wrapping for consistent formatting
5. **Placeholder Standardization**: Handles various placeholder formats (`.`, empty, "unused")

### Taxonomic Completeness

The pipeline intelligently handles incomplete taxonomic assignments:
- **Missing higher ranks**: Fills with "unclassified" labels in General format
- **Partial taxonomy**: Preserves available levels, handles missing ones appropriately
- **Inconsistent depth**: Normalizes taxonomy depth across different sequences

This ensures compatibility with downstream analysis tools that expect consistent taxonomic formatting.

## Pipeline Details

- **Robust Encoding Conversion:**  
  All scripts utilize a custom file-like wrapper (implemented in `utils.py`) that reads files using `latin-1` decoding and converts them on the fly to ASCII. This ensures that all non-ASCII characters are handled gracefully.

- **Taxonomy Header Cleaning:**  
  The scripts are designed to remove unwanted placeholders (`.`) and extra taxonomic levels.  
  - For DADA2, the script removes the accession number and retains taxonomy fields only until the first placeholder.
  - For QIIME2 and Mothur, the scripts clean the TSV taxonomy output by stripping trailing dot placeholders.

- **Reproducible Workflow:**  
  The entire process is managed by Snakemake, ensuring that jobs run in the correct order with proper dependencies, even when running in parallel.

## Customization & Troubleshooting

- **Modifying Header Formatting:**  
  The header processing logic is contained within each script (e.g., `dada2.py`, `qiime.py`). You can modify the functions `transform_header` or `clean_taxonomy` as needed.

- **Resource Management:**  
  If you encounter memory or process issues (e.g., SIGKILLs), try reducing the number of cores with `--cores 2` or increasing system resources.

- **Debugging:**  
  Use Snakemake's verbose and print shell command options (`--printshellcmds`) for detailed execution logs.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## Acknowledgements

This pipeline was developed to streamline taxonomic FASTA file processing for downstream bioinformatics applications. Contributions, suggestions, and bug reports are welcome.

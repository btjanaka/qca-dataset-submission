# Molecule filtering


Here I'm (David Mobley) experimenting with molecule filtering; we need to begin ensuring datasets -- especially those we're using for testing and benchmarking -- focus on drug-like compounds.

## Filtering resources

Several options have been proposed for filtering datasets:
- Pat Walters’ RD-filters: https://github.com/PatWalters/rd_filters?files=1 (and blog post http://practicalcheminformatics.blogspot.com/2018/08/filtering-chemical-libraries.html )
- OpenEye filtering also: https://docs.eyesopen.com/toolkits/cpp/molproptk/filter_theory.html#filter-theory-variations-of-filters -- BlockBuster, Drug, Lead, or Fragment
- QED for druglikeness: see https://www.rdkit.org/docs/source/rdkit.Chem.QED.html in RDKit and https://www.ncbi.nlm.nih.gov/pubmed/22270643 for paper

Here I intend to experiment with one or more of these.

## Prerequisites

- Requires Walters' `rd_filters`, https://github.com/PatWalters/rd_filters with appropriate `FILTER_RULES_DATA` environment variable set
- `rdkit`
- OpenEye toolkits (and license)


## Running

### RDfilters

Filtering with `rd_filters` should be something like `rd_filters filter --in discrepancies_in.smi --prefix discrepancies_out`; the file `discrepancies_in.smi` has been generated first by running `add_numbers.py` on the `input.smi` file from the 2019-07-05 eMolecules descrepancies set. Output is:

```
Using alerts from Inpharmatica
Wrote SMILES for molecules passing filters to discrepancies_out.smi
Wrote detailed data to discrepancies_out.csv
1849 of 2904 passed filters 63.7%
Elapsed time 2.16 seconds
```

Per Walters' blog post, a handy command after running this is `awk -F',' '{print $4}' discrepancies_out.csv | sort | uniq -c | sort -rn | head` which gives:
```
1887 OK
 519 Filter9_metal > 0
 130 Filter5_azo > 0
  64 Filter82_pyridinium > 0
  57 Filter39_imine > 0
  49 Filter94_2_halo_pyridine > 0
  46 Filter38_aldehyde > 0
  32 Filter74_thiol > 0
  18 Filter6_benzyl_halide > 0
  18 Filter2_acyl_phosphyl_sulfonyl_halide > 0
 ```
 So we're losing most of our compounds to having metals in them, a bunch to having azo or pyridinium or imines, halo pyridines, etc. Total removed 1058.

 Visualized PDFs of what was removed and what remains; indeed, a lot of while chemistry was removed, but there is plenty of interesting chemistry remaining. I suspect this will be a rather challenging test set.

### QED

Run `filter_by_QED.py` with an input file and an output prefix specified, e.g. `python filter_by_QED.py discrepancies_in.smi discrepancies_QED`. With currrent threshold (0.5) it removes 346 molecules.

### OEChem

Run `filter_by_OEChem.py`. Currently filters with the recommended "blockbusters" filter for drug-likeness (an OpenEye built-in filter). Retains 2106 of 2904 molecules. (798 removed)

### Consensus

Run `crosscheck.py`, looks at compounds across all three sets and retains only those that none of the methods flag for removal. Leaves only 592 (2312 removed).

## Manifest

- `rdfilters`-related:
    - `add_numbers.py`: One-off script which adds molecule numbers to a SMILES file to reformat it for rdfilters
    - `discrepancies_in.smi`: Reformatted eMolecules discrepancies set for input to rdfilters
    - `discrepancies_out.smi`: Remaining compounds
    - `discrepancies_out.csv`: Info provided by filtering
    - `visualize_rdfiltered_mols.py`: Generate PDF of molecules filtered out.
    - `discrepancies_removed.pdf`: Compounds removed by filtering
    - `discrepancies_remains.pdf`: What's left after filtering
- `QED`-related:
    - `filter_by_QED.py`: Utilizes RDKit QED code to filter by that score (runs 0 to 1). Does visualization of output molecules as well.
    - `discrepancies_QED_removed.smi`: Molecules removed by QED filtering with specified threshold
    - `discrepancies_retained.smi`: Molecules retained after QED filtering with specified threshold
    - `discrepancies...pdf`: PDF files visualizing the former two sets.
- `OEChem`-related:
    - `filter_by_OEChem.py`: Filters molecules using OEChem toolkits
    - `discrepancies_OE_removed.smi`: Removed molecules from oechem
    - `discrepancies_OE_retained.smi`: Retained molecules from OEChem
    - `discrepancies_OE_retained.pdf`: Retained molecules from OEChem
    - `discrepancies_OE_removed.pdf`: Removed molecules from OEChem
- Consensus:
    - `crosscheck.py`: Combines results of all three analyses; generates a set where all compounds ANY method suggests should be removed are removed, so the only ones left are those no method flags for removal.
    - `discrepancies_combined_removed.pdf`: All compounds removed by any method
    - `discrepancies_combined_retained.pdf`: All compounds retained by all methods.
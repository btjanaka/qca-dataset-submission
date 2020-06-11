# DANCE 2: eMolecules t142 random

### General Information

- Date: 6/5/2020
- Class: Forcefield Parameterization
- Purpose: Use molecules randomly selected from the eMolecules database to
  improve t142 parameterization in smirnoff99Frosst.
- Collection: TorsiondriveDataset
- Name: OpenFF DANCE 2 eMolecules t142 v1.0
- Number of Entries: 20 1-D torsions
- Submitter: Bryon Tjanaka (Mobley Lab)

### Generation procedure

1. Molecules are stored in `t142_random.smi`. Run `python 01_generate.py` to
   turn these molecules into the input JSON file `optimization_inputs.json.gz`.
   The indices of the `t142` parameter are re-calculated while doing this and
   stored in the `atom_indices` field in the JSON file.

### Notes

Molecules were generated with the script in this commit:
https://github.com/btjanaka/dance/commit/347100ffff35000ed042b18a12b56a164021042b

### Manifest

- `01_generator.py`: Python script used in the dataset generation
- `t142_random.smi`: Output from DANCE (same order as the OEB)
- `optimization_inputs.json.gz`: Molecules generated from `01_generator.py`
- `02_create_torsiondrive_dataset.py`: script for creating the
  TorsiondriveDataset and submitting
- `requirements.txt`: version of toolkits used in the dataset generation
  (generated with `conda list | tail -n +3 > requirements.txt`)

### Usage

1. Executing `python 01_generate.py` generates `optimization_inputs.json.gz`
   which stores the selected torsions.
2. Create dataset on QCFractal server.
   ```
   python 02_create_torsiondrive_dataset.py
   ```
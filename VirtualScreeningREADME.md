# VirtualScreeningREADME.md

> **Important Notice:**  
> **This document is intended for advanced users only.**  
> **Please read [README.md](README.md) first** for general instructions. This file supplements the main documentation and contains commands that, if misused, can quickly fill up your quota. Only proceed if you are confident in what you're doing.

---

# Virtual Screening with AutoDock

This document details the steps to prepare ligand libraries and run virtual screening using AutoDock. It covers instructions for downloading ligands from ZINC15, handling FDA-approved ligands, running jobs via the SLURM array system, and packaging data efficiently.

---

## ⚠️ **CAUTION** ⚠️

Before downloading or processing ligand files **from ZINC15**, please note:

1. **grp_MolecularDock Quota (40GB with a maximum of 1,000,000 files):**  
   - **Do not run the command in [Section 1](#1-preparing-a-library-of-ligands-from-zinc15)** if you risk filling the 40GB quota. An unchecked command may fill the entire quota, leaving no space for other operations.

2. **Archive Quota (20TB with a 2,000,000 file limit):**  
   - If you expect more than 2,000,000 ligand files, split the ligands into smaller segments. Exceeding this limit will cause the process to fail.

3. **Search Existing Libraries:**  
   - Before downloading large ligand libraries, search the supercomputer for existing libraries. Use winSCP for the easiest searching experience.

---

## 1. Preparing a Library of Ligands from ZINC15

1. **Download Ligands:**
   - Visit [ZINC15 Tranches](https://zinc15.docking.org/tranches/home/).
   - Click the **3D** tab.
   - Select the desired tranches.
   - Click the **Download** button.
   - Choose **AutoDock (*.pdbqt.zt)** as the download format.
   - Select **WGET** as the download method.

2. **Run AutoDock Script:**

   ```bash
   python3 autodock.py -Z <Downloaded_file.gz.wget> <Working_folder> <File_count_limit> <Update_submission_script:Bool (default=False)> <your_email>
   ```

   **What This Does:**
   - Downloads ligands using commands inside the downloaded file.
   - Unzips and compiles the ligands into a master file (default: `master_lig.pdbqt`).
   - If a file count limit is set, segments the master file into groups (e.g., 27 molecules split into groups of 9 each).
   - Passing `True` for the fourth argument updates the job submission script and prepares ligand input text files.

---

## 2. Preparing FDA Approved Ligands

1. **Download Ligands:**
   - Go to [ZINC FDA Subset](https://zinc15.docking.org/subsets/fda/).
   - Click the download icon and choose **SMI** format.
   - If the SMI download fails, opt for **TXT** format. The script will convert TXT to SMI format automatically.

2. **Run the Script:**

   ```bash
   python3 autodock.py -F <downloaded_ligand>.smi <Working_folder> <File_limit> True <your_email>
   ```

   This command compiles all molecules into a master PDBQT file, segments them based on the file limit, and updates the SLURM submission script.

---

## 3. Running Docking Jobs with SLURM

### 3.1 Running Multiple Dockings

To run multiple docking jobs efficiently, use the SLURM array system. Generate a submission script with:

```bash
python3 autodock.py -u <mode:str> <email:str> <max_array_No:int> <Working_folder> <exhaustiveness:int> <Script_name:str> <Prep_lig_txt:Bool (default=False)>
```

**Parameters:**
- `mode`: Use `norm` for normal docking or `vs` for virtual screening.
- `email`: Your email for job completion notifications.
- `max_array_No`: Number of tasks (usually the total number of ligands). If uncertain, run:

  ```bash
  python3 autodock.py -m <path_to_Working_folder>
  ```

- `Working_folder`: Directory for all ligand and receptor files.
- `exhaustiveness`: Determines search exhaustiveness.
- `Script_name`: Name for the generated SLURM submission script.
- `Prep_lig_txt`: Set to `True` when total ligand count is ≥ 5000.

**Example:**

```bash
python3 autodock.py -u vs example@byu.edu 9 Dock01 32 submit_test.sh
```

### 3.2 Example SLURM Submission Script

This generates a script similar to:

```bash
#!/bin/bash --login
#SBATCH --time=1-00:00:00
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --mem-per-cpu=100M
#SBATCH -J "AD_norm"
#SBATCH --mail-user=example@byu.edu
#SBATCH --mail-type=END
#SBATCH --array=1-9

Lig_array=($(ls Dock01/Ligands/*.pdbqt))
recep=$1
config=$2

for i in ${SLURM_ARRAY_TASK_ID}; do
    python3 autodock.py -d -r $recep -l ${Lig_array[$((SLURM_ARRAY_TASK_ID - 1))]} -c $config -e 32
done
```

### 3.3 Running Docking on a Library

To run docking on a library:

```bash
python3 autodock.py -dv -r <path_to_receptorPDBQT> -c <path_to_config_file>
```

**Output:**
- Best binding energies saved in `Working_folder/logs/Virtural_screen_result.out`.
- Top 10 ligands stored in `<Working_folder>/top_Ligands`.
- Explicit docking runs with increased exhaustiveness (256) saved in `Working_folder/output`.

---

## 4. Data Packaging

To conserve quota, package relevant files:

```bash
python3 autodock.py --package <Working_folder> <tar_out> <master> <result_file_or_folder> <output_file_or_folder>
```

**Parameters:**
- `Working_folder`: Folder where docking ran.
- `tar_out`: Name for the tar.gz archive.
- `master`: Optional master file.
- `result_file_or_folder`: Optional result file or folder path.
- `output_file_or_folder`: Optional output structure file or folder path.

---

## 5. Useful tar Commands

**List archive contents:**

```bash
tar -tzf <your_archive.tar.gz>
```

**Extract to a specific location:**

```bash
tar -xzf <your_archive.tar.gz> -C <destination_folder>
```

**Compress files/folders into a tar.gz archive:**

```bash
tar -czf <archive_name.tar.gz> <file1> <file2> ...

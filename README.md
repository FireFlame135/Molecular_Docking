---

# README for Docking Simulations

---

## Preface for BYU Affiliates

Molecular docking through `autodock.py` can be done in several ways. The easiest option for BYU students is using the on-campus supercomputer, which comes with all the necessary software pre-installed. Locally, you will only need PyMOL to visualize inputs (receptor, ligand, and search space) and outputs.

> **CAUTION:** Avoid using the `sudo` command on the supercomputer. Attempts to use `sudo` will fail, but you may be flagged and face potential disciplinary action for repeated attempts.

---

## Steps to Interface with the BYU Supercomputer

### 1. Requesting an Account

1. Visit [BYU Office of Research Computing (ORC)](https://rc.byu.edu) and request an account using the button in the top right corner.
2. Obtain permission to access the group "grp\_MolecularDock."
   - Your advisor must send a confirmation email to the ORC admin via a support ticket to document your need for an account and group access.
3. Attend an in-person training session at ORC after your account is approved.

### 2. Setting up SSH Multiplexing

1. Follow the instructions on [SSH Multiplexing Wiki](https://rc.byu.edu/wiki/index.php?page=SSH+Multiplexing).
2. For Windows users:
   - Install the Windows Subsystem for Linux (WSL) and Ubuntu.
   - Configure SSH multiplexing using the appropriate configuration files.

### 3. Connecting to the Supercomputer

1. Open a terminal, type `ubuntu` first in PowerShell for Windows, and then type `ssh orc`.
2. Authenticate using the BYU Duo Mobile app:
   - Set up a third-party account with `<your_username>@ssh.rc.byu.edu` to receive a valid verification code.
   - Enter the code to gain access. You will automatically be given a new code every 30 seconds. Any of them will work.

### 4. File Management on the Supercomputer

Use either terminal commands or a graphical user interface for efficient file management:

| Command                      | Description               |
| ---------------------------- | ------------------------- |
| `scp <from_file> <to_file>`  | Upload/download files     |
| `rm <file>`                  | Remove a file             |
| `mv <from_file> <to_file>`   | Move a file               |
| `sbatch <submission_script>` | Submit a SLURM job script |
| `scancel <submission_ID>`    | Cancel submitted jobs     |
| `squeue -l -u <username>`    | Check job status          |
| `fslquota`                   | Check system quota        |

For Windows, it is recommended to use [WinSCP](https://winscp.net/) for a more intuitive file management interface.

Please keep the supercomputer files intuitive and clean.

---

## Software Requirements

The following software is required on your machine for docking on the supercomputer:

- PyMOL&#x20;

  - Used for visualizing the receptor protein and ligand molecule
  - Used for visualizing the search space with a box. The ligand will only be simulated docking within the area of the box as long as you add the correct config file to the supercomputer and include it in the command.

- Python 3 (preferably a recent version)

The following software is required for docking that is run on your local machine. This is only recommended if you have no access to the BYU supercomputer or for single docking.

- `autodock_vina`

- OpenBabel

- MGLTools (compatible with Python 2.7)

If you are docking on the BYU supercomputer, all of this necessary software is pre-installed on the supercomputer. No need to also have it locally.

---

## Cheat Sheet for Supercomputer

| Shortcut                                     | Description                       |
| -------------------------------------------- | --------------------------------- |
| `<username>@ssh.rc.byu.edu:/home/<username>` | Supercomputer user home directory |
| `scp <from_file> <to_file>`                  | Upload/download files\*           |
| `rm <file>`                                  | Remove a file\*                   |
| `mv <from_file> <to_file>`                   | Move a file\*                     |
| `sbatch <submission_script>`                 | Submit a SLURM job script         |
| `scancel <submission_ID>`                    | Cancel submitted jobs             |
| `squeue -l -u <username>`                    | Check job status                  |
| `fslquota`                                   | Check system quota                |

\*These actions are recommended to be done with [WinSCP](https://winscp.net/) if possible

---

## Description

`autodock.py` is a Python script for molecular docking that:

1. Defines the search space in PyMOL
2. Prepares receptor and ligand files for input.
3. Runs `autodock_vina` for docking simulations.

For help with the script, use the following command:

```bash
python3 autodock.py -h
```

### Required Components for Docking Simulations

1. A defined search box with center coordinates.
2. PDB files for:
   - Receptor protein (prepared into PDBQT files which are required for autodock\_vina. Only do this on the supercomputer).
   - Ligand molecule(s) (prepared into PDBQT files which are required for autodock\_vina. Only do this on the supercomputer).

---

## Detailed Instructions

### 1. Setting Up the Working Directory

Again, [WinSCP](https://winscp.net/) is recommended for file I/O

Use the directory `~/groups/grp_MolecularDock/nobackup/archive/autodock_vina`. The `~` symbol refers to `/home/<username>`.

You can create folders to organize inputs, outputs, and logs. For example:

```bash
python3 autodock.py -s test True -f 1
```

- `-s`: Set up folders.
- `True`: Turn on the numbering mechanism.
- `-f`: Specify a working folder number.

#### Examples

1. Specify folder name with numbering mechanism on:

   ```bash
   python3 autodock.py -s test True
   ```

   This will create folders like `test01/`, `test02/`, etc.

2. Use `-f` to overwrite the numbering:

   ```bash
   python3 autodock.py -s test True -f 8
   ```

   Creates `test08/`.

3. No folder name specified (default is `Dock`):

   ```bash
   python3 autodock.py -s
   ```

   Creates `Dock01/`, `Dock02/`, etc.

### 2. Identifying the Search Box and Center Position

1. Load structure(s) into PyMOL

2. Visualize residues:

   1. Type into the PyMOL terminal:
      ```bash
      show sticks
      ```

3. Select residues:

   - Click residues in PyMOL.
   - To select multiple residues, hold `Shift` and drag to draw a box.
     - It is recommended to simply select all residues. 
   - To unselect, click the empty space or the selected residue again.

4. Define the search box:

   - Navigate to the working directory where autodock.py is located on your machine in PyMOL:
     ```bash
     cd /path/to/working/directory
     ```
   - Run the script:
     ```bash
     run autodock.py
     ```
   - Visualize the search space with a box: 
     ```bash
     Box('sele', X\_size, Y\_size, Z\_size, save='config.txt')
     ```
   - Replace `X_size`, `Y_size`, and `Z_size` with the desired box dimensions.

5. Fine-tune the search box center:

   - Manually move the white sphere (search box center) in PyMOL.
   - Save the updated configuration file by running the `Box` command again.

### 3. Preparing Receptor and Ligand Files

**Important:** Receptor and ligand preparation must be done on the supercomputer.

1. Prepare a single receptor or ligand:

   ```bash
   python3 autodock.py -p -r receptor.pdb -l ligand.pdb
   ```

2. Prepare multiple ligands:

   - Place ligands in the `Ligands/` folder.
   - Use:
     ```bash
     python3 autodock.py -p -l Ligands/*.pdb
     ```

3. Prepare ZINC ligands:

   - Download ligands from [ZINC15](https://zinc15.docking.org).
   - Run:
     ```bash
     python3 autodock.py -Z file.gz.wget working_folder 1000 False your_email@example.com
     ```

---

### 4. Running Docking Simulations

1. Single docking:

   ```bash
   python3 autodock.py -d -r receptor.pdbqt -l ligand.pdbqt -c config.txt
   ```

2. Library docking:
   Create a SLURM job script:

   ```bash
   python3 autodock.py -u vs your_email@example.com 9 Dock01 32 submit_test.sh
   ```

3. Submit the SLURM job script:

   ```bash
   sbatch submit_test.sh
   ```

### 5. Data Analysis

- Structural results are saved in `output/`.
- Energy results are saved in `logs/`.
- For virtual screening, results are in

 `logs/Virtual_screen_result.out`.

To visualize docking results in PyMOL:

1. Load the receptor and result files.
2. Combine molecules to evaluate polar interactions (e.g., hydrogen bonds).

---

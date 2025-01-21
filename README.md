# README for Docking Simulations with autodock.py 

---

## Preface for BYU Affiliates

Molecular docking through `autodock.py` can be done in a few ways. The easiest for BYU students is through the supercomputer on campus. That way, all of the software is already installed correctly and set up for you. Using the supercomputer, you would likely only need PyMOL on your local PC to visualize the input receptor, input ligand, search space (Box), and output.

---

## How to Interface with the Supercomputer Using SSH Multiplexing

1. **Get an Account:**
   - Request an account with the BYU Office of Research Computing (ORC) through their website ([rc.byu.edu](https://rc.byu.edu)). Use the button in the top right corner to fill out the prompted form.
   - For molecular docking, request access to the group named `grp_MolecularDock` and its contents. Obtain your advisor's documented permission.
     - Your advisor must send a confirmation message in a group support ticket to the ORC admin.
   - After approval, attend in-person training from ORC staff.

2. **Set Up SSH Multiplexing:**
   - Follow the instructions at [SSH Multiplexing Guide](https://rc.byu.edu/wiki/index.php?page=SSH+Multiplexing).
   - Ensure you have the correct configuration file and, if on Windows, use the Windows Subsystem for Linux (WSL) with Ubuntu.

3. **Access the Supercomputer:**
   - Open your terminal and start an SSH session:
     - On macOS or Linux, type: `ssh orc`
     - On Windows: Open PowerShell, type `ubuntu` to begin your Linux environment, and then `ssh orc`.
   - Enter your password and verification code:
     - Use the Duo Mobile app to generate the verification code. Set up a third-party account with `<your_username>@ssh.rc.byu.edu`.

4. **File Navigation:**
   - Use terminal commands for Linux environments or WinSCP for a graphical interface on Windows.

> **Note:** If you need assistance, ORC staff are available during work hours, and additional resources are on the [ORC website](https://rc.byu.edu).

---

## Command Line Syntax

Commands will look something like this:
```bash
python3 autodock.py -p -r path/to/receptor.pdb -l path/to/ligand.pdb
```

These commands can be executed locally (if all software is installed) or on the supercomputer through a job script.

> **Caution:** Never use the `sudo` command on the supercomputer. Doing so can result in disciplinary action.

---

## Software Requirements
- PyMOL
- Python 3

## Additional Requirements to Run Docking Locally
- autodock_vina
- OpenBabel
- MGLTools 

> **Note:** On the BYU supercomputer, autodock_vina, OpenBabel, and MGLTools are pre-installed. In this case, simply use PyMOL locally to visualize input/output structures and the search space (Box).

---

## Supercomputer Command Cheat Sheet

| Command                                      | Description                                  |
|----------------------------------------------|----------------------------------------------|
| `<username>@ssh.rc.byu.edu:/home/<username>` | Supercomputer user home address             |
| `scp <from_file> <to_file>`                  | Upload or download files                    |
| `rm <file>`                                  | Remove a file                               |
| `mv <from_file> <to_file>`                   | Move files                                  |
| `sbatch <submission_script>`                 | Submit a SLURM submission script           |
| `scancel <submission_ID>`                    | Cancel submitted SLURM jobs                |
| `squeue -l -u <username>`                    | Check the status of submitted jobs          |
| `fslquota`                                   | Check the system quota                      |
> **Note:** It is recommended to us WinSCP for file management on the supercomputer if possible. It gives a more traditional user interface so that it will look more familiar. There are equivalent softwares for macOS if needed.

---

## Description

This README covers the usage of the `autodock.py` script. This script is used to:

1. Define the search space of the docking simulation. This is done with a config file which contains an X, Y, and Z coordinate for the center of the search space and the size of the box in every direction.
2. Prepare receptor and ligand inputs (for single or library ligands). This should only be done on the supercomputer. 
3. Run `autodock_vina` simulations.
4. Give output for these simulations. 

### Requirements:
1. A search box and center position.
2. PDB file of receptor (turned into a PDBQT and prepared only on the supercomputer).
3. PDB file of ligand(s) (turned into a PDBQT and prepared only on the supercomputer).

For more information, navigate to the directory containing `autodock.py` and run:
```bash
python3 autodock.py -h
```

---
Edited up to here. Please continue editing, then review with the word doc to ensure completeness
## Instructions

### 1. Set Up Working Directory

- For BYU supercomputer users, use the directory `~/groups/grp_MolecularDock/nobackup/archive/autodock_vina`.
- Create folders to organize inputs, outputs, and logs using:
```bash
python3 autodock.py -s <XXX> True -f <Number>
```

#### Examples:
1. **Folder with numbering:**
    ```bash
    python3 autodock.py -s test True  # Creates test01/
    python3 autodock.py -s test True  # Creates test02/
    ```

2. **Without numbering:**
    ```bash
    python3 autodock.py -s test  # Overwrites previous folder "test/"
    ```

3. **Default folder:**
    ```bash
    python3 autodock.py -s  # Creates Dock01/
    ```

### 2. Identify Search Box and Center Position

#### Using PyMOL:
1. Load structures:
    ```
    fetch XXXX  # XXXX is the 4-digit PDB ID.
    ```
   Or drag and drop files into PyMOL.

2. Visualize residues:
    ```
    show sticks
    ```

3. Calculate the search box:
    ```python
    run autodock.py
    Box('sele', X_size, Y_size, Z_size)
    ```
   Save configuration:
    ```python
    Box('sele', X_size, Y_size, Z_size, save='WORKING_FOLDER/config.txt')
    ```

4. Fine-tune the center location by manually adjusting the white sphere in PyMOL. Once satisfied, redraw the box using:
    ```python
    Box('sele', X_size, Y_size, Z_size)
    ```

---

### 3. Preparation of Receptor and Ligand PDBQT Files

> **Important:** Preparations must be done on the supercomputer.

1. **Single Receptor or Ligand:**
   ```bash
   python3 autodock.py -p -r path/to/receptor.pdb -l path/to/ligand.pdb
   ```
   Use `-r` or `-l` individually if preparing only one file.

2. **Multiple Ligands:**
   ```bash
   python3 autodock.py -p -l path/to/Ligands/*.pdb
   ```

3. **Library of Ligands (ZINC or FDA-approved):** Follow additional instructions for downloading and segmenting files as described in the original documentation.

---

### 4. Run Autodock

1. **Single Docking:**
   ```bash
   python3 autodock.py -d -r Dock01/Receptor/receptor.pdbqt -l Dock01/Ligand/ligand.pdbqt -c Dock01/config.txt
   ```

2. **SLURM Array System for Multiple Dockings:**
   Generate a submission script:
   ```bash
   python3 autodock.py -u vs email@example.com 9 Dock01 32 submit_test.sh
   ```
   Submit the script using SLURM:
   ```bash
   sbatch submit_test.sh
   ```

---

### 5. Data Analysis

- Results will be stored in the `output` and `logs` folders.
- For visual analysis, combine receptor and ligand structures in PyMOL and use commands like:
  ```
  zoom organic
  set transparency, 0.6
  show surface
  ```
- Analyze polar contacts and interactions as described in the original document.

---

### 6. Data Packaging

Compress results into a `.tar.gz` file:
```bash
python3 autodock.py --package <working_folder> <tar_out> <master> <result_file_or_folder> <output_file_or_folder>
```

Use `tar` commands to manage compressed files, e.g., to extract or view contents.

---

For further details or troubleshooting, refer to the original document or consult the ORC resources.


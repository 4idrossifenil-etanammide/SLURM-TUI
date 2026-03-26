# SLURM-TUI

A lightweight terminal UI for building and submitting SLURM jobs. No more memorizing `sbatch` flags.

**Zero dependencies** -- pure Python 3 standard library (`curses`). Works on any Linux system with Python 3.7+ and SLURM installed.

<!-- If you have a screenshot, replace this block with: ![screenshot](screenshot.png) -->
```
                              SLURM Job Builder
 Up/Down/Tab:Navigate  Left/Right:Cycle  Type:Edit  ^S:Save  ^R:Run  ^Q:Quit

 --- Job Identity ---
        Job Name: my_training_run            (Name shown in squeue)
       *Account: < iscrc_ugen >              (< > to cycle)
      *Partition: < boost_usr_prod >          (< > to cycle)
             QOS: < normal >                  (< > to cycle)
 --- Resources ---
           Nodes: 1                           (Number of nodes)
  Tasks per Node: 1                           (MPI tasks per node)
   CPUs per Task: 4                           (CPU cores per task)
   GPUs per Node: 1                           (GPUs per node)
        GPU Type: < a100 >                    (< > to cycle)
      Time Limit: 02:00:00                    (Max walltime HH:MM:SS)
          Memory: 64G                         (Memory per node)
 --- Output & Notifications ---
     Output File: slurm-%j.out               (%j = job ID)
      Error File: slurm-%j.err               (%j = job ID)
 --- Shell Commands (runs after #SBATCH) ---
        Commands: [3 lines] Enter=edit
                    module load python/3.12
                    source venv/bin/activate
                    srun python train.py

            [^S] Save Script
            [^R] Submit (sbatch)
            [^P] Preview
            [^Q] Quit
```

## Features

- **Auto-detects** your accounts, partitions, and QOS from the cluster (via `sacctmgr` and `sinfo`)
- **Form-based UI** with all common SLURM fields: job name, account, partition, QOS, nodes, tasks, CPUs, GPUs (with type), time, memory, output/error files, mail notifications
- **Built-in command editor** for writing the script body directly in the TUI
- **Save to .sh file** or **submit directly** via `sbatch`
- **Pre-fill from existing scripts**: edit an existing SLURM script in the TUI
- **Selector fields** cycle through detected cluster values with arrow keys

## Installation

It's a single file. Copy it somewhere in your `$PATH`:

```bash
# From this repo
curl -fsSL https://raw.githubusercontent.com/4idrossifenil-etanammide/SLURM-TUI/main/slurm-tui -o ~/.local/bin/slurm-tui
chmod +x ~/.local/bin/slurm-tui
```

Or clone and symlink:

```bash
git clone git@github.com:4idrossifenil-etanammide/SLURM-TUI.git
ln -s "$(pwd)/SLURM-TUI/slurm-tui" ~/.local/bin/slurm-tui
```

> Make sure `~/.local/bin` (or wherever you place it) is in your `$PATH`.

## Usage

```bash
# Start fresh with auto-detected defaults
slurm-tui

# Pre-fill from an existing SLURM script
slurm-tui my_job.sh
```

## Keybindings

### Main form

| Key | Action |
|-----|--------|
| `Up` / `Down` / `Tab` / `Shift-Tab` | Navigate between fields |
| `Left` / `Right` | Cycle through selector options |
| Any letter/number | Start editing a text field directly |
| `Backspace` | Delete last character |
| `Enter` | Open command editor (Commands field) / Cycle option (selectors) |
| `Ctrl+S` | Save script to `.sh` file |
| `Ctrl+R` | Submit job via `sbatch` |
| `Ctrl+P` | Preview generated script |
| `Ctrl+Q` | Quit |

### Command editor

| Key | Action |
|-----|--------|
| `Ctrl+S` | Save and close |
| `Esc` | Cancel (discard changes) |
| `Enter` | New line |
| `Ctrl+U` | Clear current line |
| Arrow keys, Home, End | Move cursor |

### Save prompt

| Key | Action |
|-----|--------|
| `Enter` | Confirm filename |
| `Esc` | Cancel save, return to form |

## Generated script example

```bash
#!/bin/bash
#SBATCH --job-name=my_training_run
#SBATCH --account=iscrc_ugen
#SBATCH --partition=boost_usr_prod
#SBATCH --qos=normal
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=4
#SBATCH --gres=gpu:a100:1
#SBATCH --time=02:00:00
#SBATCH --mem=64G
#SBATCH --output=slurm-%j.out
#SBATCH --error=slurm-%j.err

# -- Environment --
echo "Job $SLURM_JOB_ID started on $(hostname) at $(date)"
echo "Nodes: $SLURM_JOB_NODELIST"

# -- User commands --
module load python/3.12
source venv/bin/activate
srun python train.py

echo "Job $SLURM_JOB_ID finished at $(date)"
```

## Requirements

- Python 3.7+
- SLURM CLI tools (`sbatch`, `sinfo`, `sacctmgr`) -- for auto-detection and job submission
- A terminal that supports curses

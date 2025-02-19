# dist2nwk

Version: 0.0.1  
Author: GenPat  
Contact: genpat@izs.it

## Overview

MST Builder is a high-performance tool for constructing Minimum Spanning Trees (MSTs) from symmetric distance matrices. The tool uses Numba JIT compilation and parallelization to significantly accelerate tree construction and branch recrafting operations.

## Features

- **Numba-accelerated algorithms**: Core numerical functions are JIT-compiled for maximum performance
- **Parallel processing**: Uses multiple CPU cores for faster computation on large matrices
- **Zero-distance grouping**: Special handling for grouping nodes with zero Hamming distance
- **Branch recrafting**: Optional verification and correction of branch lengths
- **Progress monitoring**: Real-time progress tracking with tqdm progress bars
- **Detailed logging**: Comprehensive timing and performance metrics
- **Newick format output**: Produces standard Newick tree format for compatibility with phylogenetic tools

## Requirements

- Python 3.7+
- NumPy
- Numba
- tqdm

## Installation

```bash
# Clone the repository
git clone https://github.com/genpat-it/dist2nwk.git
cd dist2nwk

# Install required packages
pip install -r requirements.txt
```

## Usage

```bash
python dist2nwk.py input_matrix.tsv output_tree.nwk [options]
```

### Command-line Arguments

| Argument | Description |
|----------|-------------|
| `input_file` | TSV file containing the symmetric distance matrix |
| `output_file` | Output file for the Newick tree |
| `--recraft` | Enable branch recrafting to verify/correct branch lengths |
| `--threads N` | Number of threads to use (0=auto-detect) |
| `--quiet` | Minimize progress output (disable progress bars) |
| `--version` | Show version information and exit |

### Input Format

The input file should be a tab-separated values (TSV) file with:
- First row: Column headers with sample names (first cell can be empty)
- First column: Sample names
- Remaining cells: Distance values between samples

Example:
```
	Sample1	Sample2	Sample3
Sample1	0	0.5	0.8
Sample2	0.5	0	0.6
Sample3	0.8	0.6	0
```

## Algorithm Details

1. **Central Node Selection**: Identifies the optimal starting node with minimum sum of distances
2. **MST Construction**: Builds tree structure by repeatedly adding nearest neighbor nodes
3. **Zero-distance Grouping**: Groups samples with zero distance into a single node
4. **Branch Recrafting** (optional): Verifies and corrects branch lengths by comparing tree distances with input matrix

## Performance Optimizations

- Uses Numba JIT compilation for core numerical functions
- Parallelizes distance calculations with `prange`
- Optimizes memory access patterns for Numba compatibility
- Uses vectorized numpy operations where possible
- Provides thread count control via command line

## Example

```bash
# Basic usage
python dist2nwk.py distance_matrix.tsv output_tree.nwk

# With branch recrafting and 8 threads
python dist2nwk.py distance_matrix.tsv output_tree.nwk --recraft --threads 8

# Quiet mode (minimal output)
python dist2nwk.py distance_matrix.tsv output_tree.nwk --quiet
```

## Output

The tool produces:
1. A Newick format tree file at the specified output path
2. Progress information and timing statistics in the console output

## Benchmark

Below are performance results from running dist2nwk on a large dataset with 5,000 nodes.

**System Specifications:**
```bash
$ lscpu | grep -E "Model name|Socket|Core|Thread"
Thread(s) per core:  1
Core(s) per socket:  48
Socket(s):           4
Model name:          Intel(R) Xeon(R) Gold 6252N CPU @ 2.30GHz

$ uname -a
Linux gtc-collab-int.izs.intra 4.18.0-553.el8_10.x86_64 #1 SMP Fri May 24 08:32:12 EDT 2024 x86_64 x86_64 x86_64 GNU/Linux
```

**Benchmark Results:**
```
$ time python dist2nwk.py benchmarks/distance_matrix_5000.tsv 5000.nwk --threads 32
MST Builder v0.0.1
Author: GenPat
Contact: genpat@izs.it
--------------------------------------------------
[+] Running with 32 threads
[+] Numba version: 0.58.0
[+] NumPy version: 1.25.0
[+] Reading distance matrix from benchmarks/distance_matrix_5000.tsv
[+] Loaded 4999x4999 distance matrix in 11.16 seconds
[+] Building MST for 4999 nodes with Numba acceleration
[+] Finding optimal central node...
[+] Selected node Sample_3275 as central node (index 3274)
[+] Building MST tree structure...
Building MST:  10%|████████▏                                                                         | 498/4998 [00:18<02:56, 25.49node/s][+] Added 499/4999 nodes to MST (10.0%)
Building MST:  20%|████████████████▎                                                                 | 996/4998 [00:39<03:09, 21.07node/s][+] Added 998/4999 nodes to MST (20.0%)
Building MST:  30%|████████████████████████▏                                                        | 1496/4998 [01:04<02:58, 19.60node/s][+] Added 1497/4999 nodes to MST (29.9%)
Building MST:  40%|████████████████████████████████▎                                                | 1994/4998 [01:30<02:31, 19.85node/s][+] Added 1996/4999 nodes to MST (39.9%)
Building MST:  50%|████████████████████████████████████████▍                                        | 2492/4998 [01:53<01:49, 22.83node/s][+] Added 2495/4999 nodes to MST (49.9%)
Building MST:  60%|████████████████████████████████████████████████▍                                | 2990/4998 [02:12<01:08, 29.33node/s][+] Added 2994/4999 nodes to MST (59.9%)
Building MST:  70%|████████████████████████████████████████████████████████▌                        | 3492/4998 [02:27<00:34, 43.19node/s][+] Added 3493/4999 nodes to MST (69.9%)
Building MST:  80%|████████████████████████████████████████████████████████████████▌                | 3985/4998 [02:35<00:12, 80.92node/s][+] Added 3992/4999 nodes to MST (79.9%)
Building MST:  89%|███████████████████████████████████████████████████████████████████████▌        | 4469/4998 [02:39<00:02, 211.09node/s][+] Added 4491/4999 nodes to MST (89.8%)
Building MST:  97%|█████████████████████████████████████████████████████████████████████████████▉  | 4868/4998 [02:39<00:00, 751.47node/s][+] Added 4990/4999 nodes to MST (99.8%)
Building MST: 100%|█████████████████████████████████████████████████████████████████████████████████| 4998/4998 [02:39<00:00, 31.27node/s]
[+] MST construction completed in 161.16 seconds
[+] Added 4999 nodes to tree
[+] Converting tree to Newick format...
[+] Converted to Newick format in 0.01 seconds
[+] Writing Newick tree to 5000.nwk
[+] MST processing completed in 172.33 seconds
[+] Final Newick tree saved to 5000.nwk
[+] MST Builder v0.0.1 (GenPat) completed successfully
[+] Contact: genpat@izs.it
real    2m53.166s
user    2m51.651s
sys     0m15.955s
```

**Performance Summary:**
- **Total processing time:** 2m 53s 
- **Matrix loading:** 11.16s
- **MST construction:** 161.16s 
- **Newick conversion and file output:** 0.01s

The benchmark demonstrates excellent scalability with multi-threading, processing nearly 5,000 nodes in under 3 minutes. Note how the processing speed increases significantly as the algorithm progresses, going from ~20 nodes/s initially to over 750 nodes/s in the final stage.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

This is a free and open-source software that allows anyone to:
- Use the software for any purpose
- Study how the software works and modify it
- Redistribute the software
- Improve the software and release your improvements to the public
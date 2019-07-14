# worm-branching

To run on a specific simulation cluster pass the directory containing the UMMAP simulation output, simulation name, and cluster number

    python main.py -D PATH-TO-UMMAP-OUTPUT -S SIMULATION-NAME -c CLUSTER-NUMBER

for example, to run on cluster 0 of `Adams_2017_SLE1S` located in the directory `UMMAP`

    python main.py -D UMMAP -S Adams_2017_SLE1S -c 0

This will look for coordinate frame data in

    UMMAP/Adams_2017_SLE1S/ClusterAnalysisConnectionFiles0/UMMAPOUT/Snaps/Cluster_Shape

 and connection frame data in

    UMMAP/Adams_2017_SLE1S/ClusterAnalysisConnectionFiles0/UMMAPOUT/Data/Basic_Cluster/Connect


The program will match all available coordinate and connection files and save a pickle file for each frame to `cache/Adams_2017_SLE1S/cluster0` in addition to a JSON file `cache/Adams_2017_SLE1S/cluster0.json` which contains metrics and logging data for all frames in the cluster.

## Additional Flags

- The number of CPU cores to use can be specified with the `-C/--cores` flag. The default is all available cores.
- The output directory can be specified with the `--cache` flag. The default is `cache` in the directory from which the program is run.
- To stop on knwon errors pass the `--debug` flag. The default is to catch benign errors and store them in the frame log.
- To overwrite any existing clusters pass the `-f/--force` flag. The default is to increment and append `_i` for each existing cluster.

## JSON Data

Each JSON file contains the following information for each frame

- `name` name (number) of the frame
- `path` path to pickle file relative to where the program was run
- `micelle_count` number of micelles in the frame
- `branched_count` number of branched micelles in the frame
- `avg_length` average length of micelles in the frame
- `max_length` maximum length of micelles in the frame
- `min_length` minimum length of micelles in the frame
- `total_length` sum of the lengths of micelles in the frame
- `total_max_cap_length` sum of maximum pairwise distances between cap odes of all micelles in the frame
- `principal_micelle_length` sum of principal lengths of all micelles in the frame as point clouds
- `principal_curve_length` sum of principal lengths of all micelles in the frame as skeletons
- `errors` log of errors encountered during runtime
- `micelles` dictionary of additional information for each micelle in the frame (see below)

Each entry of the `micelles` field contains the following

- `name` name of the micelle (*FRAME#.MICELLE#*)
- `point_count` number of points in the micelle
- `branch_nodes` number of branch nodes in the micelle skeleton
- `length` length of the micelle skeleton
- `cap_lengths` all pairwise distances between skeleton cap nodes
- `max_cap_length` maximum pairwise distance between skeleton cap nodes
- `principal_micelle_length` length of the micelle along its principal axis (computed using PCA on the micelle point cloud)
- `principal_curve_length` length of the skeleton along its principal axis (computed using PCA on the points of the micelle skeleton)
- `log` log of errors and additional information for debugging

A JSON file located at `cache/Adams_2017_SLE1S/cluster0.json` can be accessed from the Python shell as follows

```python
import json

with open('cache/Adams_2017_SLE1S/cluster0.json', 'rb') as f:
    data = json.load(f)
```

The total length of all micelles in frame 6000 may now be accessed as follows

```python
data['6000']['total_length']
```

Additional information for micelle 6000.0 is accessed as follows

```python
data['6000']['micelles']['0']
```

with the following output

```python
{
    'name': '6000.0',
    'length': 147.73919144591096,
    'cap_lengths': [51.294568672669435, 49.39351472503379, 50.72837905798269],
    'max_cap_length': 51.294568672669435,
    'principal_micelle_length': 56.59817887127396,
    'principal_curve_length': 51.71297890655339,
    'branch_nodes': 1,
    'point_count': 7080,
    'log': {
        'merge': {
            '((0, 1, 0, 0), (0, 1, 1, 0), (1, 1, 1, 0))' : {
                '((0, 1, 0, 0), (0, 1, 1, 0), (1, 1, 1, 0))' : 'MERGE [
                    ((0, 1, 0, 0), (0, 1, 1, 0)),
                    ((0, 1, 0, 0), (1, 1, 1, 0)),
                    ((0, 1, 1, 0), (1, 1, 1, 0))
                ]'
            }
        }
    }
}
```

## Visualization and JSON Exploration (In Progress)

The JSON data and pickle files produced for each simulation can be accessed by a convenient visualization program, which can be run by passing `viz` as the first argument, followed by the location of the simulation output folder:

    python main.py viz cache/Adams_2017_SLE1S

This will bring up the following command line interface

    Cluster
    --------
    cluster0
    cluster1
    cluster2

    [Adams_2017_SLE1S]

To select cluster 2 type `cluster 2` or `load 2`, or specify the cluster from the initial command with the `-c/--cluster` flag:

    python main.py viz cache/Adams_2017_SLE1S -c 2

This will bring up a new command line interface

    [Adams_2017_SLE1S] cluster 2

      Frame | # Micelle | # Branched | Max Length | Total Length | Total Principal Length | Total Length Ratio
      ----- | --------- | ---------- | ---------- | ------------ | ---------------------- | ------------------
      6000  | 5         | 1          | 147.7392   | 205.8536     | 105.8663               | 1.9445
      5996  | 5         | 1          | 148.2506   | 207.6234     | 107.3193               | 1.9346
      5992  | 4         | 1          | 152.5904   | 208.6851     | 106.7325               | 1.9552
      5988  | 4         | 1          | 152.3626   | 210.6985     | 108.0004               | 1.9509
      5984  | 5         | 1          | 150.6501   | 210.2784     | 109.2119               | 1.9254
      5980  | 5         | 1          | 152.3478   | 210.2222     | 108.3034               | 1.9410
      5976  | 5         | 1          | 149.4091   | 209.0205     | 109.4947               | 1.9090
      5972  | 4         | 1          | 150.8426   | 209.6251     | 109.6172               | 1.9123
      5968  | 5         | 1          | 153.3009   | 210.1829     | 106.7203               | 1.9695
      5964  | 4         | 1          | 151.3191   | 206.0309     | 107.6856               | 1.9133

    [Adams_2017_SLE1S.cluster2]

By default, information on the last 10 frames is displayed. To view all type `ls` or `list`. To view the last 100 frames type `ls 100` or `list 100`.

__TODO__
- queries
- custom sorting

To select and plot frame 6000 type `frame 6000` or `load 6000`

    [Adams_2017_SLE1S.cluster2] frame 6000

      Micelle | Points | Branch Nodes | Length   | Principal Micelle Length | Length Ratio
      ------- | ------ | ------------ | -------- | ------------------------ | ------------
      0       | 7080   | 1            | 147.7392 | 56.5982                  | 2.6103
      1       | 1700   | 0            | 34.7245  | 26.9919                  | 1.2865
      2       | 976    | 0            | 20.6638  | 19.7621                  | 1.0456
      3       | 4      | 0            | 1.4068   | 1.3852                   | 1.0156
      4       | 4      | 0            | 1.3192   | 1.1288                   | 1.1687

    [Adams_2017_SLE1S.cluster2:6000]

In this scope, all micelles in the frame and their skeletons are plotted. To plot micelle 0 type `micelle 0` or `load 0`.

To move back in scope press enter to pass an empty line, or type `exit`. To quit type `quit`.

## Animate

Animations of a cluster can be created from the command line

    python main.py viz cache/Adams_2017_SLE1S -c 2 -a --exit

or in the frame scope by typing `animate`. The above command would save the animation to `animations/Adams_2017-cluster2.mp4`.

![Animation](https://github.com/shirtd/worm-branching/blob/master/examples/Adams_2017_SLE1S-cluster2_720.gif)


## Usage

    usage: main.py [-h] {run, viz} ...

    Micelle Segmentation.

    optional arguments:
      -h, --help  show this help message and exit

    cmd:
      {run, viz}  run or visualize micelle segmentation. default: run

#### Run

    usage: main.py run [-h] [-S SIMULATION] [-D ROOT_DIR] [-c CLUSTER]
                       [-a CONNECT_FILE [CONNECT_FILE ...]]
                       [-m MICELLE_FILE [MICELLE_FILE ...]] [-d DEPTH] [-r RATIO]
                       [-b EIGEN_BOUND] [-n NAME] [--match MATCH]
                       [--coord_match COORD_MATCH] [--connect_match CONNECT_MATCH]
                       [-N FRAMES] [-R] [-C CORES] [--cache CACHE] [--debug]
                       [--no-clean] [--no-cycles] [-f]

    optional arguments:
      -h, --help            show this help message and exit
      -S SIMULATION, --simulation SIMULATION
                            simulation directory. will set connect_file and
                            micelle_file
      -D ROOT_DIR, --root-dir ROOT_DIR
                            simulation root directory. default: ../UMMAP
      -c CLUSTER, --cluster CLUSTER
                            cluster analysis #. default: 0
      -a CONNECT_FILE [CONNECT_FILE ...], --connect_file CONNECT_FILE [CONNECT_FILE ...]
                            the list of files containing the list of \*indices\* for
                            connections in the shape file.
      -m MICELLE_FILE [MICELLE_FILE ...], --micelle_file MICELLE_FILE [MICELLE_FILE ...]
                            the list of files containing the Micelle coordinates.
      -d DEPTH, --depth DEPTH
                            recursive segmentation depth. default: 1
      -r RATIO, --ratio RATIO
                            segment repair ratio threshold. default: 5
      -b EIGEN_BOUND, --eigen-bound EIGEN_BOUND
                            segment split eigenvalue threshold. default: 0.015
      -n NAME, --name NAME  micelle name.
      --match MATCH         regular expression for matching coordinate and
                            connection files. default (positive integer without
                            leading 0s): '0*(\\d+)'
      --coord_match COORD_MATCH
                            regular expression for finding coordinate files.
                            default (.pdb or .dat file ending in a positive
                            integer): '.\*\\d+\\.[dat|pdb]'
      --connect_match CONNECT_MATCH
                            regular expression for finding coordinate files.
                            default (.dat file ending in a positive integer):
                            '.\*\\d+\\.dat'
      -N FRAMES, --frames FRAMES
                            Number of frames to process. default: all
      -R, --reverse         reverse frame order.
      -C CORES, --cores CORES
                            number of cores to use.
      --cache CACHE         cache directory. default: cache
      --debug               raise logged errors.
      --no-clean            check if vertices of all edges are within unit distance.
      --no-cycles           remove all cycles.
      -f, --force           overwrite existing files.

#### Visualization

      usage: main.py viz [-h] [-c CLUSTER] [-a] [-e] [-A ANIMATION_DIR] [dir]

      positional arguments:
        dir                   pickle file to load

      optional arguments:
        -h, --help            show this help message and exit
        -c CLUSTER, --cluster CLUSTER
                              specify cluster
        -a, --animate         animate cluster
        -e, --exit            exit after execution
        -A ANIMATION_DIR, --animation-dir ANIMATION_DIR
                              animation directory

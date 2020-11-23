<p align="center"><img height="110px" src="https://ghicons.github.com/assets/images/blue/png/Private%20repos.png"/></p>
<h1 align="center">How to keep Git monorepos manageable</h1>
<h4 align="center">Keep monorepos manageable using grafting</h4>
<h5 align="center">@droidpl & @selkins13</h5>

<p align="center">
  <a href="#mega-prerequisites">Prerequisites</a> •  
  <a href="#books-resources">Resources</a>
</p>

---

## :mega: Prerequisites

Before joining the workshop, there are a few items that you will need to install or bring with you.

- A clone or fork of a repository you would like to reduce in size
- Time it takes to clone the repository  
    `time git clone [url]`
- Install [git-sizer](https://github.com/github/git-sizer/#getting-started)
- Install [git-filter-repo](https://github.com/newren/git-filter-repo)
- Clone the this [workshop repository](https://github.com/githubuniverseworkshops/grafting-monorepos)


## :scroll: Activity 1: History (10 minutes)

We will take a look at some of the characteristics of your own repository in preparation of the analyzation phase.

### History of a single file

Let's look at the history of a single file within your repository.  Pick a file that has been around for a while. Using a command line tool, run the following command on it:  
```
git log --oneline --graph -- [filename]
```

*Be sure you are in the location of that file inside your command line tool.*

<details><summary>Sample results</summary>
```bash
  * a7ddcea58ae2 Drop all 00-INDEX files from Documentation/
  * 4b290af0b3a4 Docs: Added a pointer to the formatted docs to README
  * 9f4a68e416a5 README: Improve documentation descriptions
  * 6bef44b9b969 README: add a new README file, pointing to the Documentation/
  * 9d85025b0418 docs-rst: create an users manual book
  * 44b10006a97e README: convert it to ReST markup
  * 35db7e94cdee README: Delete obsolete i386 info + update arch/i386/ paths
  * cfaf790f932b README: remove trailing whitespace
  * 3047bcc537cf README: cosmetic fixes
  * ad29fff88976 README: Add ARC architecture
  * 6609b638353c README: GTK+ is a acronym
  * 1913c6f4488e README: Change gzip/bzip2 to xz compression format
  * 49d86dc92c6e README: Update version number reference
  *   3bd7bf1f0fe1 Merge branch 'master' into for-next
  |\  
  | * 80b810b276cf localmodconfig: Document localmodconfig in README
  * | fc0d1b93fe7b README: document "make olddefconfig"
  |/  
  * 3773b4540721 README: Remove sporadic tabs
  * 7f65e924c0cf README: Consolidate discussions of -stable patches
  * c072c3f0e14f README: Capitalize start of sentence
  * 88f7a642cf0d README: More consistent and readable white space
  * 6d12760c9f3d README: `Alternately` -> `Alternatively`
  * a6144bb9e7b4 README: Better comma usage
  * a20e3a795b1a README: Grammar: `me has' -> `I have'
  * 5b4285fbd469 README: Use `X' and `x' consistently
  * 0466dcbeda72 Update version number references in README
  * cddb5de05317 Readme: Add architecture
  * b3235fe420ed README: cite nconfig
  * b2d8993026a2 README: fix misleading pointer to the defconf directory
  * 590a5857291e kconfig: add a note about the deps to the 'silentoldconfig' help
```

</details>
  <br>  

### How many times has the file changed?

Let's find out how many times that file has changed.  On the same file, run the following command to see how may commits have been made for that single file:  
```bash
git log --oneline [filename] | wc -l
```


<details><summary>Sample results</summary>

  ```bash
  ❯ git log --oneline README | wc -l
    46
  ```

</details>
<br>


### Report out

Report your findings in comments section of the [Activity 1: History Issue](https://github.com/githubuniverseworkshops/grafting-monorepos/issues/1)
  - Include answers to the following questions in your comments:
    - How often was that file changed?
    - Do you find any pattern?
    - Are the commit messages relevant enough to know why the file changed? Do you follow a standard?

> :warning: Make sure during all this exercise you don't post any private information that should not be shared publicly.

## :bar_chart: Activity 2: Analysis (20 minutes)

We will use different analysis tools to identify wrong practices in a repository. To do it we will use the following commands:
- **git-sizer**
- **git-find-dirs-many-files**
- **git-find-lfs-extensions**
- **git-find-dirs-unwanted**
- **git-filter-repo**

Before starting any analysis, **pick one repository of your preference** that you would like to analyze.

> :warning: Make sure during all this exercise you don't post any private information that should not be shared publicly.

Clone this repository as we have added all the tools into the it for making the workshop more convenient:
```bash
# Clone the repository
git clone https://github.com/githubuniverseworkshops/grafting-monorepos.git


#  or use the GitHub CLI
gh repo clone githubuniverseworkshops/grafting-monorepos
```

### Stats of repo size: [`git-sizer`](https://github.com/github/git-sizer)

1. [Download](https://github.com/github/git-sizer/releases/latest) the corresponding compiled version of `git-sizer`.
> Optionally you can install git-sizer using Homebrew if you are on Mac.

2. Run the tool from the root of the repository to analyze:
```bash
/path/to/git-sizer --verbose
```

<details><summary>See the result of running <code>git-sizer</code></summary>

``` bash
time ../git-sizer-1.3.0-darwin-amd64/git-sizer --verbose

Processing blobs: 2173019                        
Processing trees: 4613547                        
Processing commits: 967035                        
Matching commits to trees: 967035                        
Processing annotated tags: 671                        
Processing references: 676                        
| Name                         | Value     | Level of concern               |
| ---------------------------- | --------- | ------------------------------ |
| Overall repository size      |           |                                |
| * Commits                    |           |                                |
|   * Count                    |   967 k   | *                              |
|   * Total size               |   752 MiB | ***                            |
| * Trees                      |           |                                |
|   * Count                    |  4.61 M   | ***                            |
|   * Total size               |  12.8 GiB | ******                         |
|   * Total tree entries       |   373 M   | *******                        |
| * Blobs                      |           |                                |
|   * Count                    |  2.17 M   | *                              |
|   * Total size               |  77.6 GiB | ********                       |
| * Annotated tags             |           |                                |
|   * Count                    |   671     |                                |
| * References                 |           |                                |
|   * Count                    |   676     |                                |
|                              |           |                                |
| Biggest objects              |           |                                |
| * Commits                    |           |                                |
|   * Maximum size         [1] |  72.7 KiB | *                              |
|   * Maximum parents      [2] |    66     | ******                         |
| * Trees                      |           |                                |
|   * Maximum entries      [3] |  2.19 k   | **                             |
| * Blobs                      |           |                                |
|   * Maximum size         [4] |  13.5 MiB | *                              |
|                              |           |                                |
| History structure            |           |                                |
| * Maximum history depth      |   162 k   |                                |
| * Maximum tag depth      [5] |     1     |                                |
|                              |           |                                |
| Biggest checkouts            |           |                                |
| * Number of directories  [6] |  4.74 k   | **                             |
| * Maximum path depth     [7] |    13     | *                              |
| * Maximum path length    [8] |   134 B   | *                              |
| * Number of files        [9] |  70.7 k   | *                              |
| * Total size of files    [9] |   918 MiB |                                |
| * Number of symlinks    [10] |    40     |                                |
| * Number of submodules       |     0     |                                |

[1]  91cc53b0c78596a73fa708cceb7313e7168bb146
[2]  2cde51fbd0f310c8a2c5f977e665c0ac3945b46d
[3]  648d5c2e00672dfe3c217d25b8c73802744f3d3b (refs/heads/master:arch/arm/boot/dts)
[4]  29af5167cd0057fcfbfab150378322008d3d2667 (refs/heads/master:drivers/gpu/drm/amd/include/asic_reg/nbio/nbio_6_1_sh_mask.h)
[5]  5dc01c595e6c6ec9ccda4f6f69c131c0dd945f8c (refs/tags/v2.6.11)
[6]  47bb244d15374c342ffe8ede1fcf15b46f08827f (8e4c309f9f33b76c09daa02b796ef87918eee494^{tree})
[7]  78a269635e76ed927e17d7883f2d90313570fdbc (dae09011115133666e47c35673c0564b0a702db7^{tree})
[8]  b0da5ce619daec8138cf92dfcf00e7a51ce856a9 (d8763340d2cb6262fb86424315a1f92cabc0e23c^{tree})
[9]  5bf53b0c610a5abf0cd874d0687f4e731009fda4 (9c75b68b91ff010d8d4c703b93954f605e2ef516^{tree})
[10] f29a5ea76884ac37e1197bef1941f62fda3f7b99 (f5308d1b83eba20e69df5e0926ba7257c8dd9074^{tree})
../git-sizer-1.3.0-darwin-amd64/git-sizer --verbose  248.41s user 28.56s system 136% cpu 3:22.24 total
```
</details>

### Find files that should be in LFS: [`git-find-lfs-extensions`](https://github.com/githubuniverseworkshops/grafting-monorepos/blob/main/scripts/git-find-lfs-extensions)

1. Checkout the `grafting-monorepos` repository
2. Run the tool from the root of the repository to analyze:
```bash
/path/to/grafting-monorepos/scripts/git-find-lfs-extensions
```

<details><summary>See the results of running <code>git-find-lfs-extensions</code></summary>

```bash
time ../grafting-monorepos/scripts/git-find-lfs-extensions

Type           Extension              LShare    LCount     Count      Size       Min       Max
-------        ---------              -------   -------   -------   -------   -------   -------
all            *                        0 %        73     70569       917         0        13
text           h                        0 %        62     21370       304         0        13
text           c                        0 %         7     29461       523         0         0
text           json                     1 %         2       332         7         0         0
text           s                        0 %         1      1290         8         0         0
text w/o ext   maintainers             50 %         1         2         0         0         0

Add to .gitattributes:

../grafting-monorepos/scripts/git-find-lfs-extensions  3.18s user 5.41s system 80% cpu 10.619 total
```
</details>

### Print directories with the number of files contained: [`git-find-dirs-many-files`](https://github.com/githubuniverseworkshops/grafting-monorepos/blob/main/scripts/git-find-dirs-many-files)

1. Checkout the `grafting-monorepos` repository
2. Run the tool from the root of the repository to analyze:
```bash
/path/to/grafting-monorepos/scripts/git-find-dirs-many-files
```

<details><summary>See the results of running <code>git-find-dirs-many-files</code></summary>

```bash
time ../grafting-monorepos/scripts/git-find-dirs-many-files | head -n 20

   70602 .
   28698 ./drivers
   15946 ./arch
    7486 ./Documentation
    5416 ./include
    5085 ./drivers/gpu
    4996 ./drivers/gpu/drm
    4922 ./drivers/net
    4569 ./tools
    4492 ./arch/arm
    4071 ./Documentation/devicetree
    4064 ./Documentation/devicetree/bindings
    2453 ./include/linux
    2438 ./drivers/media
    2366 ./drivers/net/ethernet
    2279 ./sound
    2214 ./arch/arm/boot
    2204 ./drivers/staging
    2196 ./tools/testing
    2184 ./arch/arm/boot/dts
../grafting-monorepos/scripts/git-find-dirs-many-files  12.07s user 45.92s system 135% cpu 42.640 total
head -n 20  0.00s user 0.00s system 0% cpu 42.630 total
```
</details>

### Find dirs that should not be committed: [`git-find-dirs-unwanted`](https://github.com/githubuniverseworkshops/grafting-monorepos/blob/main/scripts/git-find-dirs-unwanted)

1. Checkout the `grafting-monorepos` repository
2. Run the tool from the root of the repository to analyze:
```bash
/path/to/grafting-monorepos/scripts/git-find-dirs-unwanted | head -n 15            
```

<details><summary>See the results of running <code>git-find-dirs-unwanted</code></summary>

```bash
time ../grafting-monorepos/scripts/git-find-dirs-unwanted | head -n 15

    4569 tools/
    4064 Documentation/devicetree/bindings/
    2088 tools/testing/selftests/
    1385 arch/x86/
    1268 tools/perf/
     632 tools/testing/selftests/bpf/
     448 lib/
     368 arch/x86/include/asm/
     327 tools/perf/util/
     326 Documentation/devicetree/bindings/sound/
     325 tools/testing/selftests/bpf/progs/
     313 tools/perf/pmu-events/
     283 include/dt-bindings/clock/
     269 Documentation/devicetree/bindings/clock/
     240 arch/x86/kernel/
/path/to/grafting-monorepos/scripts/git-find-dirs-unwanted  81.27s user 13.35s system 114% cpu 1:22.32 total
head -n 15  0.00s user 0.00s system 0% cpu 1:22.31 total
```
</details>

### Analyze the repository: `git-filter-repo --analyze`

1. Clone the [`git-filter-repo` tool](https://github.com/newren/git-filter-repo)
2. Execute the tool from the linux repository
```bash
/path/to/git-filter-repo/git-filter-repo --analyze
```

<details><summary>See the results of running <code>git filter-repo --analyze</code></summary>

```bash
time ../git-filter-repo/git-filter-repo --analyze

Processed 7754272 blob sizes
Processed 967008 commitswarning: inexact rename detection was skipped due to too many files.
warning: you may want to set your diff.renameLimit variable to at least 817 and retry the command.
Processed 967035 commits
Writing reports to .git/filter-repo/analysis...done.
../git-filter-repo/git-filter-repo --analyze  1104.40s user 45.86s system 105% cpu 18:11.40 total
```
</details>

You can see the results of the analysis in the [`linux-filter-repo` folder](./linux-filter-repo)

### Report out

Report your findings from the above commands in comments section of the [Activity 2: Analysis](https://github.com/githubuniverseworkshops/grafting-monorepos/issues/2).  Be sure to include answers to the following questions in your comments, if possible:
    - Do you find any patterns?
    - Was there anything surprising? 

## :seedling: Activity 3: Graft a repository (20 minutes)

### Grafting commands
- **Step 1**: Clean your repository
  - LFS
  - Unneeded or unwanted files
  - Add dependencies to dependency management
  - Remove independent components
  - Remove any binaries
  - Review automation in the repository
- **Step 2**: [Create a new repository](https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/creating-a-new-repository)
- **Step 3**: Prepare for grafting
  - Communicate properly to the affected teams
  - Merge all in progress work
  - If something is not merged, it will not be moved
  - Call [GitHub Professional Services](https://services.github.com/) if things go south
- **Step 4**: Delete your history
```bash
# Delete the git folder that contains git objects
rm -rf .git

# Initialize a new history
git init

# Set the new repository
git remote add origin git@github.com:githubuniverseworkshops/grafting-repo.git
```
- **Step 5**: Write a commit referencing to the previous repository as your first commit in the new repository
```bash
# Add all files to the stage
git add --all

# Add changes to history
git commit -m "Previous repo can be found on https://github.com/torvalds/linux"

# Submit your changes to upstream
git push --set-upstream origin main
```

### Working with the new repository
To preserve the history while working with the new repository, follow the grafting command:
```bash
# Fetch the old history
git fetch git@github.com:torvalds/linux.git

# See you only have one commit in it
git log --oneline

# See the commits we are replacing
git rev-parse --short HEAD
git rev-parse --short FETCH_HEAD

# Perform the grafting operation replacing HEAD with FETCH_HEAD
git replace HEAD FETCH_HEAD
```

Check that all the changes that have happened to the repository are local and don't get pushed when new code goes to the repository:
```bash
# Check that the new commit goes to the right repo
# Modify a file
echo "Test" > test.txt
git add --all
git commit -m "Adding a test commit"

# Check that you can navigate the history
git log --oneline | head -n 10

# Push the change and see the number of commits is still 2
git push
```

### Analysis after grafting

**Git sizer**:
<details><summary>Git sizer analysis on linux:</summary>

```bash
time ../git-sizer-1.3.0-darwin-amd64/git-sizer --verbose

Processing blobs: 68974                        
Processing trees: 4703                        
Processing commits: 2                        
Matching commits to trees: 2                        
Processing annotated tags: 0                        
Processing references: 3                        
| Name                         | Value     | Level of concern               |
| ---------------------------- | --------- | ------------------------------ |
| Overall repository size      |           |                                |
| * Commits                    |           |                                |
|   * Count                    |     2     |                                |
|   * Total size               |  2.18 KiB |                                |
| * Trees                      |           |                                |
|   * Count                    |  4.70 k   |                                |
|   * Total size               |  2.81 MiB |                                |
|   * Total tree entries       |  74.0 k   |                                |
| * Blobs                      |           |                                |
|   * Count                    |  69.0 k   |                                |
|   * Total size               |   908 MiB |                                |
| * Annotated tags             |           |                                |
|   * Count                    |     0     |                                |
| * References                 |           |                                |
|   * Count                    |     3     |                                |
|                              |           |                                |
| Biggest objects              |           |                                |
| * Commits                    |           |                                |
|   * Maximum size         [1] |  1.10 KiB |                                |
|   * Maximum parents      [1] |     1     |                                |
| * Trees                      |           |                                |
|   * Maximum entries      [2] |  2.19 k   | **                             |
| * Blobs                      |           |                                |
|   * Maximum size         [3] |  13.5 MiB | *                              |
|                              |           |                                |
| History structure            |           |                                |
| * Maximum history depth      |     2     |                                |
| * Maximum tag depth          |     0     |                                |
|                              |           |                                |
| Biggest checkouts            |           |                                |
| * Number of directories  [4] |  4.71 k   | **                             |
| * Maximum path depth     [4] |    10     | *                              |
| * Maximum path length    [4] |   100 B   | *                              |
| * Number of files        [4] |  69.3 k   | *                              |
| * Total size of files    [4] |   909 MiB |                                |
| * Number of symlinks     [4] |    31     |                                |
| * Number of submodules       |     0     |                                |

[1]  3fd90e0a58ed50bb7bdb0add2d6e05fb7fc4cac7 (refs/heads/master)
[2]  648d5c2e00672dfe3c217d25b8c73802744f3d3b (refs/heads/master:arch/arm/boot/dts)
[3]  29af5167cd0057fcfbfab150378322008d3d2667 (refs/heads/master:drivers/gpu/drm/amd/include/asic_reg/nbio/nbio_6_1_sh_mask.h)
[4]  4367680f5832eb1c626ea6f6ca5d283f66c686bd (refs/heads/master^{tree})
../git-sizer-1.3.0-darwin-amd64/git-sizer --verbose  0.48s user 0.13s system 176% cpu 0.346 total
```
</details>

**Git find LFS extensions**:
<details><summary>Find lfs extensions on linux:</summary>

```bash
time ../grafting-monorepos/scripts/git-find-lfs-extensions

Type           Extension                                          LShare    LCount     Count      Size       Min       Max
-------        ---------                                         -------   -------   -------   -------   -------   -------
all            *                                                     0 %        72     69269       908         0        13
text           h                                                     0 %        62     21369       304         0        13
text           c                                                     0 %         7     29460       523         0         0
text           json                                                  1 %         2       332         7         0         0
text w/o ext   maintainers                                          50 %         1         2         0         0         0

Add to .gitattributes:

../grafting-monorepos/scripts/git-find-lfs-extensions  2.68s user 4.58s system 99% cpu 7.293 total
```
</details>

**Find many dirs**:
<details><summary>Find many dirs analysis on linux:</summary>

```bash
time ../grafting-monorepos/scripts/git-find-dirs-many-files | head -n 20
   69309 .
   28688 ./drivers
   14720 ./arch
    7485 ./Documentation
    5416 ./include
    5085 ./drivers/gpu
    4996 ./drivers/gpu/drm
    4921 ./drivers/net
    4518 ./tools
    4232 ./arch/arm
    4070 ./Documentation/devicetree
    4063 ./Documentation/devicetree/bindings
    2453 ./include/linux
    2438 ./drivers/media
    2366 ./drivers/net/ethernet
    2279 ./sound
    2204 ./drivers/staging
    2201 ./arch/arm/boot
    2184 ./arch/arm/boot/dts
    2167 ./tools/testing
../grafting-monorepos/scripts/git-find-dirs-many-files  11.45s user 43.86s system 134% cpu 41.070 total
head -n 20  0.00s user 0.00s system 0% cpu 41.058 total
```
</details>

**Find dirs unwanted**:
<details><summary>Find dirs unwanted analysis on linux:</summary>

```bash
time ../grafting-monorepos/scripts/git-find-dirs-unwanted | head -n 15
    4518 tools/
    4063 Documentation/devicetree/bindings/
    2059 tools/testing/selftests/
    1255 tools/perf/
    1239 arch/x86/
     630 tools/testing/selftests/bpf/
     448 lib/
     368 arch/x86/include/asm/
     327 tools/perf/util/
     326 Documentation/devicetree/bindings/sound/
     325 tools/testing/selftests/bpf/progs/
     313 tools/perf/pmu-events/
     283 include/dt-bindings/clock/
     269 Documentation/devicetree/bindings/clock/
     230 tools/lib/
```
</details>

**Git filter repo analyze**:
<details><summary>Git filter repo analysis on linux:</summary>

```bash
time ../git-filter-repo/git-filter-repo --analyze

Processed 73679 blob sizes
Processed 2 commits
Writing reports to .git/filter-repo/analysis...done.
../git-filter-repo/git-filter-repo --analyze  5.74s user 2.23s system 101% cpu 7.865 total
```
</details>

Find the details in the [`linux-filter-repo-grafted` folder](./linux-filter-repo-grafted)

## :books: Resources

- Linux Kernel open source repository - [https://github.com/torvalds/linux](https://github.com/torvalds/linux)
- Git SCM reference manual - [https://git-scm.com/docs](https://git-scm.com/docs)
- Git Sizer - [https://github.com/github/git-sizer/#getting-started](https://github.com/github/git-sizer/#getting-started)
- Platform samples scripts - [https://github.com/github/platform-samples/tree/master/scripts](https://github.com/github/platform-samples/tree/master/scripts)
- Git filter repo analysis tool - [https://github.com/newren/git-filter-repo](https://github.com/newren/git-filter-repo)

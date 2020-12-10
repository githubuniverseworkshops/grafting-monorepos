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
- Install [Git LFS](https://docs.github.com/en/free-pro-team@latest/github/managing-large-files/installing-git-large-file-storage)
- Install [git-sizer](https://github.com/github/git-sizer/#getting-started)
- Install [git-filter-repo](https://github.com/newren/git-filter-repo)
- Clone the this [workshop repository](https://github.com/githubuniverseworkshops/grafting-monorepos)


## :scroll: Activity 1: History (10 minutes)

We will take a look at some of the characteristics of your own repository in preparation of the analyzation phase.

### History of your repository

Let's look at the history of your repository.  Using a command line tool, run the following command on it:  
```
git log --oneline
```

*Be sure you are in the location of that file inside your command line tool.*

<details><summary>Sample results</summary>

```
❯ git log --oneline

  c2e7554e1b85 (HEAD -> master, origin/master, origin/HEAD) Merge tag 'gfs2-v5.10-rc4-fixes' of git://git.kernel.org/pub/scm/linux/kernel/git/gfs2/linux-gfs2
  ce228d459424 Merge tag 'nfsd-5.10-2' of git://linux-nfs.org/~bfields/linux
  f86fee1845ee Merge tag 'linux-kselftest-kunit-fixes-5.10-rc5' of git://git.kernel.org/pub/scm/linux/kernel/git/shuah/linux-kselftest
  20b329129009 gfs2: Fix regression in freeze_go_sync
  0fa8ee0d9ab9 Merge branch 'for-linus' of git://git.kernel.org/pub/scm/linux/kernel/git/dtor/input
  111e91a6df50 Merge tag 's390-5.10-4' of git://git.kernel.org/pub/scm/linux/kernel/git/s390/linux
  ed129cd75ac1 Merge tag 'mips_fixes_5.10_1' of git://git.kernel.org/pub/scm/linux/kernel/git/mips/linux
  be1dd6692adb Merge tag 'perf-tools-fixes-for-v5.10-2020-11-17' of git://git.kernel.org/pub/scm/linux/kernel/git/acme/linux
  9dacf44c3837 Merge branch 'urgent-fixes' of git://git.kernel.org/pub/scm/linux/kernel/git/paulmck/linux-rcu
  ac3b57adf87a MIPS: Alchemy: Fix memleak in alchemy_clk_setup_cpu
  61a2f1aecf60 MIPS: kernel: Fix for_each_memblock conversion
  9c87c9f41245 Merge tag 'arm-soc-fixes-v5.10-2' of git://git.kernel.org/pub/scm/linux/kernel/git/soc/soc
  a5698b3835f5 Merge tag 'hyperv-fixes-signed' of git://git.kernel.org/pub/scm/linux/kernel/git/hyperv/linux
  a08f4523243c Merge tag 'for_linus' of git://git.kernel.org/pub/scm/linux/kernel/git/mst/vhost
  568beb27959b perf test: Avoid an msan warning in a copied stack.
  1c756cd429d8 perf inject: Fix file corruption due to event deletion
  cc05af8e2e91 Merge tag 'imx-fixes-5.10-4' of git://git.kernel.org/pub/scm/linux/kernel/git/shawnguo/linux into arm/fixes
  39c8d39c04bb Merge tag 'renesas-fixes-for-v5.10-tag1' of git://git.kernel.org/pub/scm/linux/kernel/git/geert/renesas-devel into arm/fixes
  09162bc32c88 (tag: v5.10-rc4) Linux 5.10-rc4
  efd838fec17b vhost scsi: Add support for LUN resets.
  18f1becb6948 vhost scsi: add lun parser helper
  47a3565e8bb1 vhost scsi: fix cmd completion race
  25b98b64e284 vhost scsi: alloc cmds per vq instead of session
```

</details>
  <br>  

  ### Explore a commit

  Choose a commit in your repositories history and let's explore that single commit.  Run the following command to see the details of the commit.
  ```
  git log --pretty=fuller [commit hash]
  ```
  *Be sure you are in the location of that file inside your command line tool.*

  <details><summary>Sample results</summary>

  ```
  ❯ git log --pretty=fuller efd838fec17b

    commit efd838fec17bd8756da852a435800a7e6281bfbc
    Author:     Mike Christie <michael.christie@oracle.com>
    AuthorDate: Mon Nov 9 23:33:23 2020 -0600
    Commit:     Michael S. Tsirkin <mst@redhat.com>
    CommitDate: Sun Nov 15 17:30:55 2020 -0500

    vhost scsi: Add support for LUN resets.

    In newer versions of virtio-scsi we just reset the timer when an a
    command times out, so TMFs are never sent for the cmd time out case.
    However, in older kernels and for the TMF inject cases, we can still get
    resets and we end up just failing immediately so the guest might see the
    device get offlined and IO errors.

    For the older kernel cases, we want the same end result as the
    modern virtio-scsi driver where we let the lower levels fire their error
    handling and handle the problem. And at the upper levels we want to
    wait. This patch ties the LUN reset handling into the LIO TMF code which
    will just wait for outstanding commands to complete like we are doing in
    the modern virtio-scsi case.

    Note: I did not handle the ABORT case to keep this simple. For ABORTs
    LIO just waits on the cmd like how it does for the RESET case. If
    an ABORT fails, the guest OS ends up escalating to LUN RESET, so in
    the end we get the same behavior where we wait on the outstanding
    cmds.

    Signed-off-by: Mike Christie <michael.christie@oracle.com>
    Link: https://lore.kernel.org/r/1604986403-4931-6-git-send-email-michael.christie@oracle.com
    Signed-off-by: Michael S. Tsirkin <mst@redhat.com>
    Acked-by: Stefan Hajnoczi <stefanha@redhat.com>
  ```

  </details>
    <br>



### How many changeset are in your repository

Let's find out how many changes your repository has. Run the following command to see how may commits have been made in your repository:

```
git log --oneline | wc -l
```
*Be sure you are in the location of that file inside your command line tool.*

<details><summary>Sample results</summary>

  ```
  ❯ git log --oneline | wc -l
    967467
  ```

</details>
<br>


### Report out

Report your findings in comments section of the [Activity 1: History Issue](https://github.com/githubuniverseworkshops/grafting-monorepos/issues/1)
  - Include answers to the following questions in your comments:
    - How many commits have been made on your repository?
    - Do you find any patterns?
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

Clone this repository as we have added all the tools into it for making the workshop more convenient:
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

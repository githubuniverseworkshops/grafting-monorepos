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

## :warning: Troubleshooting

### Windows Certificate Issues

It seems that there may be some trouble with [Windows 10 not having the correct DigiCert](https://docs.microsoft.com/en-us/troubleshoot/mem/configmgr/connectivity-issues-digicert-global-root-g2-not-installed) intermediate certificate which could cause a clone of this repository to fail when retrieving the LFS portions of the repository.

To diagnose this issue, try running the following:

```bash
openssl s_client -connect github.com:443 -showcerts
openssl s_client -connect github-cloud.s3.amazonaws.com:443 -showcerts
```

You should get the following results:

<details><summary> Open to see <code>openssl s_client -connect github.com:443 -showcerts</code> results</summary>

```bash
$ openssl s_client -connect github-cloud.s3.amazonaws.com:443 -showcerts
CONNECTED(00000005)
depth=2 C = IE, O = Baltimore, OU = CyberTrust, CN = Baltimore CyberTrust Root
verify return:1
depth=1 C = US, O = DigiCert Inc, OU = www.digicert.com, CN = DigiCert Baltimore CA-2 G2
verify return:1
depth=0 C = US, ST = Washington, L = Seattle, O = "Amazon.com, Inc.", CN = *.s3.amazonaws.com
verify return:1
---
Certificate chain
 0 s:/C=US/ST=Washington/L=Seattle/O=Amazon.com, Inc./CN=*.s3.amazonaws.com
   i:/C=US/O=DigiCert Inc/OU=www.digicert.com/CN=DigiCert Baltimore CA-2 G2
-----BEGIN CERTIFICATE-----
MIIG3DCCBcSgAwIBAgIQCC32junGkxW+v3IHmzgQ/TANBgkqhkiG9w0BAQsFADBk
MQswCQYDVQQGEwJVUzEVMBMGA1UEChMMRGlnaUNlcnQgSW5jMRkwFwYDVQQLExB3
d3cuZGlnaWNlcnQuY29tMSMwIQYDVQQDExpEaWdpQ2VydCBCYWx0aW1vcmUgQ0Et
MiBHMjAeFw0xOTExMDkwMDAwMDBaFw0yMTAzMTIxMjAwMDBaMGwxCzAJBgNVBAYT
AlVTMRMwEQYDVQQIEwpXYXNoaW5ndG9uMRAwDgYDVQQHEwdTZWF0dGxlMRkwFwYD
VQQKExBBbWF6b24uY29tLCBJbmMuMRswGQYDVQQDDBIqLnMzLmFtYXpvbmF3cy5j
b20wggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDJW86RuSlYnitcFHiQ
298DPDFqEaC10ttw23m1Y+20aHic+GM9iUD+r2XO7LavEoZ0LORnEF5jM6bjt4aH
oXqZ6rB9fj6oMhtlQ8M7w72+WdM7aJ/6ZcqO3+NB+RB56pitqSQjukvSI88mjY6k
Mk/Qg2ZEABA3a/DwMfVS1Mcznqaw/yt+t+r+e33/WFTAZ3A0jmyXMCTIVrcpWulv
kza70D/oiF5Peiwlc8oey4x+A/QDfrmQHv/wu7sJegFj0hlQ96LqfjdHu48t6que
2yE2S8Ae6+2QXMrvloR9jOKK0EarN5bxSOpU1HriwiNwO3/bGEYjSnL0xLkpTKv/
eGHzAgMBAAGjggOAMIIDfDAfBgNVHSMEGDAWgBTAErIodGhGZ+lwJXQaAEVbBn1c
RDAdBgNVHQ4EFgQU3fImAGS3yvdcppam16zL4ScVDBMwLwYDVR0RBCgwJoISKi5z
My5hbWF6b25hd3MuY29tghBzMy5hbWF6b25hd3MuY29tMA4GA1UdDwEB/wQEAwIF
oDAdBgNVHSUEFjAUBggrBgEFBQcDAQYIKwYBBQUHAwIwgYEGA1UdHwR6MHgwOqA4
oDaGNGh0dHA6Ly9jcmwzLmRpZ2ljZXJ0LmNvbS9EaWdpQ2VydEJhbHRpbW9yZUNB
LTJHMi5jcmwwOqA4oDaGNGh0dHA6Ly9jcmw0LmRpZ2ljZXJ0LmNvbS9EaWdpQ2Vy
dEJhbHRpbW9yZUNBLTJHMi5jcmwwTAYDVR0gBEUwQzA3BglghkgBhv1sAQEwKjAo
BggrBgEFBQcCARYcaHR0cHM6Ly93d3cuZGlnaWNlcnQuY29tL0NQUzAIBgZngQwB
AgIweQYIKwYBBQUHAQEEbTBrMCQGCCsGAQUFBzABhhhodHRwOi8vb2NzcC5kaWdp
Y2VydC5jb20wQwYIKwYBBQUHMAKGN2h0dHA6Ly9jYWNlcnRzLmRpZ2ljZXJ0LmNv
bS9EaWdpQ2VydEJhbHRpbW9yZUNBLTJHMi5jcnQwDAYDVR0TAQH/BAIwADCCAX0G
CisGAQQB1nkCBAIEggFtBIIBaQFnAHYApLkJkLQYWBSHuxOizGdwCjw1mAT5G9+4
43fNDsgN3BAAAAFuTXY7MAAABAMARzBFAiEAifHUKVkZIWkFHtFhj/67xabxyzSu
d6PCFBt38GXC6XkCIDMPF2N2mKzbst5SXda23BFBufDKzr65+2FfwBi/kAbuAHYA
RJRlLrDuzq/EQAfYqP4owNrmgr7YyzG1P9MzlrW2gagAAAFuTXY7KAAABAMARzBF
AiEArv1CKeorl1PX81umtl6pGG1EHBMmMutjOBoqy/uRtDcCIEKX1COA9LAdsGba
c+OaQuDJ6NgVI21BJvp1+tp2nkMLAHUAu9nfvB+KcbWTlCOXqpJ7RzhXlQqrUuga
kJZkNo4e0YUAAAFuTXY7NQAABAMARjBEAiAbg6xfQS9eHRVZuZJWoA+B8ppvSsIj
wmuCOOsVEGZEagIgQcPbXr7jb+KbF7vZGOUNsRUwOU+BFrIVFmRFZ1i4S7EwDQYJ
KoZIhvcNAQELBQADggEBAAyJqLR4IsJWivGmNp4NrBQyQJCHDeHkhCrINnG+kwKl
8Z+YtkNT37eghIQtnpu1Pf8TZXAB19hW/Pqc9hHUTflDocABgr3NQAQky3nIa6xG
P6x2yyYoBTxOBsLXD2Hfb9uy5MOZCWnJxi+OjlE4SyterDtrODalRruhsZLFclcf
68pfpR0+jrIDM4px3FRjrdIdf2wV2iT/gxRmeDCiO4kE3ClbzwDt3oPJxGZfzrKI
/jBUKommu5hnhLgdvOA8dXuaGLOT98kd8jgvNr7xwB/FIvAPyEoZMzgI6R5AICXd
S+ZudBk1TeOZGidtNVBrxg93SBwwGsP/Wgva0PQtmmM=
-----END CERTIFICATE-----
 1 s:/C=US/O=DigiCert Inc/OU=www.digicert.com/CN=DigiCert Baltimore CA-2 G2
   i:/C=IE/O=Baltimore/OU=CyberTrust/CN=Baltimore CyberTrust Root
-----BEGIN CERTIFICATE-----
MIIEYzCCA0ugAwIBAgIQAYL4CY6i5ia5GjsnhB+5rzANBgkqhkiG9w0BAQsFADBa
MQswCQYDVQQGEwJJRTESMBAGA1UEChMJQmFsdGltb3JlMRMwEQYDVQQLEwpDeWJl
clRydXN0MSIwIAYDVQQDExlCYWx0aW1vcmUgQ3liZXJUcnVzdCBSb290MB4XDTE1
MTIwODEyMDUwN1oXDTI1MDUxMDEyMDAwMFowZDELMAkGA1UEBhMCVVMxFTATBgNV
BAoTDERpZ2lDZXJ0IEluYzEZMBcGA1UECxMQd3d3LmRpZ2ljZXJ0LmNvbTEjMCEG
A1UEAxMaRGlnaUNlcnQgQmFsdGltb3JlIENBLTIgRzIwggEiMA0GCSqGSIb3DQEB
AQUAA4IBDwAwggEKAoIBAQC75wD+AAFz75uI8FwIdfBccHMf/7V6H40II/3HwRM/
sSEGvU3M2y24hxkx3tprDcFd0lHVsF5y1PBm1ITykRhBtQkmsgOWBGmVU/oHTz6+
hjpDK7JZtavRuvRZQHJaZ7bN5lX8CSukmLK/zKkf1L+Hj4Il/UWAqeydjPl0kM8c
+GVQr834RavIL42ONh3e6onNslLZ5QnNNnEr2sbQm8b2pFtbObYfAB8ZpPvTvgzm
+4/dDoDmpOdaxMAvcu6R84Nnyc3KzkqwIIH95HKvCRjnT0LsTSdCTQeg3dUNdfc2
YMwmVJihiDfwg/etKVkgz7sl4dWe5vOuwQHrtQaJ4gqPAgMBAAGjggEZMIIBFTAd
BgNVHQ4EFgQUwBKyKHRoRmfpcCV0GgBFWwZ9XEQwHwYDVR0jBBgwFoAU5Z1ZMIJH
WMys+ghUNoZ7OrUETfAwEgYDVR0TAQH/BAgwBgEB/wIBADAOBgNVHQ8BAf8EBAMC
AYYwNAYIKwYBBQUHAQEEKDAmMCQGCCsGAQUFBzABhhhodHRwOi8vb2NzcC5kaWdp
Y2VydC5jb20wOgYDVR0fBDMwMTAvoC2gK4YpaHR0cDovL2NybDMuZGlnaWNlcnQu
Y29tL09tbmlyb290MjAyNS5jcmwwPQYDVR0gBDYwNDAyBgRVHSAAMCowKAYIKwYB
BQUHAgEWHGh0dHBzOi8vd3d3LmRpZ2ljZXJ0LmNvbS9DUFMwDQYJKoZIhvcNAQEL
BQADggEBAC/iN2bDGs+RVe4pFPpQEL6ZjeIo8XQWB2k7RDA99blJ9Wg2/rcwjang
B0lCY0ZStWnGm0nyGg9Xxva3vqt1jQ2iqzPkYoVDVKtjlAyjU6DqHeSmpqyVDmV4
7DOMvpQ+2HCr6sfheM4zlbv7LFjgikCmbUHY2Nmz+S8CxRtwa+I6hXsdGLDRS5rB
bxcQKegOw+FUllSlkZUIII1pLJ4vP1C0LuVXH6+kc9KhJLsNkP5FEx2noSnYZgvD
0WyzT7QrhExHkOyL4kGJE7YHRndC/bseF/r/JUuOUFfrjsxOFT+xJd1BDKCcYm1v
upcHi9nzBhDFKdT3uhaQqNBU4UtJx5g=
-----END CERTIFICATE-----
---
Server certificate
subject=/C=US/ST=Washington/L=Seattle/O=Amazon.com, Inc./CN=*.s3.amazonaws.com
issuer=/C=US/O=DigiCert Inc/OU=www.digicert.com/CN=DigiCert Baltimore CA-2 G2
---
No client certificate CA names sent
Server Temp Key: ECDH, P-256, 256 bits
---
SSL handshake has read 3395 bytes and written 322 bytes
---
New, TLSv1/SSLv3, Cipher is ECDHE-RSA-AES128-GCM-SHA256
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES128-GCM-SHA256
    Session-ID: 83B0540EA3E3401D39A74A25FC24B428E3A418E0B1A9EEB26D02355F3FEA33B6
    Session-ID-ctx:
    Master-Key: E4509EC796A88A0E54849D82B0D0D98ECC32FCFEED4298CC782055FDD23EB502C9D05AC75CE3E88E1D68BEE4489D8FD7
    Start Time: 1607618046
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
---
closed
```

</details>

and

<details><summary> Open to see <code>openssl s_client -connect github-cloud.s3.amazonaws.com:443 -showcerts</code> results</summar>


```bash
$ openssl s_client -connect github.com:443 -showcerts
CONNECTED(00000005)
depth=2 C = US, O = DigiCert Inc, OU = www.digicert.com, CN = DigiCert High Assurance EV Root CA
verify return:1
depth=1 C = US, O = DigiCert Inc, OU = www.digicert.com, CN = DigiCert SHA2 High Assurance Server CA
verify return:1
depth=0 C = US, ST = California, L = San Francisco, O = "GitHub, Inc.", CN = github.com
verify return:1
---
Certificate chain
 0 s:/C=US/ST=California/L=San Francisco/O=GitHub, Inc./CN=github.com
   i:/C=US/O=DigiCert Inc/OU=www.digicert.com/CN=DigiCert SHA2 High Assurance Server CA
-----BEGIN CERTIFICATE-----
MIIG1TCCBb2gAwIBAgIQBVfICygmg6F7ChFEkylreTANBgkqhkiG9w0BAQsFADBw
MQswCQYDVQQGEwJVUzEVMBMGA1UEChMMRGlnaUNlcnQgSW5jMRkwFwYDVQQLExB3
d3cuZGlnaWNlcnQuY29tMS8wLQYDVQQDEyZEaWdpQ2VydCBTSEEyIEhpZ2ggQXNz
dXJhbmNlIFNlcnZlciBDQTAeFw0yMDA1MDUwMDAwMDBaFw0yMjA1MTAxMjAwMDBa
MGYxCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQHEw1T
YW4gRnJhbmNpc2NvMRUwEwYDVQQKEwxHaXRIdWIsIEluYy4xEzARBgNVBAMTCmdp
dGh1Yi5jb20wggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC7MrTQ2J6a
nox5KUwrqO9cQ9STO5R4/zBUxxvI5S8bmc0QjWfIVAwHWuT0Bn/H1oS0LM0tTkQm
ARrqN77v9McVB8MWTGsmGQnS/1kQRFuKiYGUHf7iX5pfijbYsOkfb4AiVKysKUNV
UtgVvpJoe5RWURjQp9XDWkeo2DzGHXLcBDadrM8VLC6H1/D9SXdVruxKqduLKR41
Z/6dlSDdeY1gCnhz3Ch1pYbfMfsTCTamw+AtRtwlK3b2rfTHffhowjuzM15UKt+b
rr/cEBlAjQTva8rutYU9K9ONgl+pG2u7Bv516DwmNy8xz9wOjTeOpeh0M9N/ewq8
cgbR87LFaxi1AgMBAAGjggNzMIIDbzAfBgNVHSMEGDAWgBRRaP+QrwIHdTzM2WVk
YqISuFlyOzAdBgNVHQ4EFgQUYwLSXQJf943VWhKedhE2loYsikgwJQYDVR0RBB4w
HIIKZ2l0aHViLmNvbYIOd3d3LmdpdGh1Yi5jb20wDgYDVR0PAQH/BAQDAgWgMB0G
A1UdJQQWMBQGCCsGAQUFBwMBBggrBgEFBQcDAjB1BgNVHR8EbjBsMDSgMqAwhi5o
dHRwOi8vY3JsMy5kaWdpY2VydC5jb20vc2hhMi1oYS1zZXJ2ZXItZzYuY3JsMDSg
MqAwhi5odHRwOi8vY3JsNC5kaWdpY2VydC5jb20vc2hhMi1oYS1zZXJ2ZXItZzYu
Y3JsMEwGA1UdIARFMEMwNwYJYIZIAYb9bAEBMCowKAYIKwYBBQUHAgEWHGh0dHBz
Oi8vd3d3LmRpZ2ljZXJ0LmNvbS9DUFMwCAYGZ4EMAQICMIGDBggrBgEFBQcBAQR3
MHUwJAYIKwYBBQUHMAGGGGh0dHA6Ly9vY3NwLmRpZ2ljZXJ0LmNvbTBNBggrBgEF
BQcwAoZBaHR0cDovL2NhY2VydHMuZGlnaWNlcnQuY29tL0RpZ2lDZXJ0U0hBMkhp
Z2hBc3N1cmFuY2VTZXJ2ZXJDQS5jcnQwDAYDVR0TAQH/BAIwADCCAXwGCisGAQQB
1nkCBAIEggFsBIIBaAFmAHUAKXm+8J45OSHwVnOfY6V35b5XfZxgCvj5TV0mXCVd
x4QAAAFx5ltprwAABAMARjBEAiAuWGCWxN/M0Ms3KOsqFjDMHT8Aq0SlHfQ68KDg
rVU6AAIgDA+2EB0D5W5r0i4Nhljx6ABlIByzrEdfcxiOD/o6//EAdQAiRUUHWVUk
VpY/oS/x922G4CMmY63AS39dxoNcbuIPAgAAAXHmW2nTAAAEAwBGMEQCIBp+XQKa
UDiPHwjBxdv5qvgyALKaysKqMF60gqem8iPRAiAk9Dp5+VBUXfSHqyW+tVShUigh
ndopccf8Gs21KJ4jXgB2AFGjsPX9AXmcVm24N3iPDKR6zBsny/eeiEKaDf7UiwXl
AAABceZbahsAAAQDAEcwRQIgd/5HcxT4wfNV8zavwxjYkw2TYBAuRCcqp1SjWKFn
4EoCIQDHSTHxnbpxWFbP6v5Y6nGFZCDjaHgd9HrzUv2J/DaacDANBgkqhkiG9w0B
AQsFAAOCAQEAhjKPnBW4r+jR3gg6RA5xICTW/A5YMcyqtK0c1QzFr8S7/l+skGpC
yCHrJfFrLDeyKqgabvLRT6YvvM862MGfMMDsk+sKWtzLbDIcYG7sbviGpU+gtG1q
B0ohWNApfWWKyNpquqvwdSEzAEBvhcUT5idzbK7q45bQU9vBIWgQz+PYULAU7KmY
z7jOYV09o22TNMQT+hFmo92+EBlwSeIETYEsHy5ZxixTRTvu9hP00CyEbiht5OTK
5EiJG6vsIh/uEtRsdenMCxV06W2f20Af4iSFo0uk6c1ryHefh08FcwA4pSNUaPyi
Pb8YGQ6o/blejFzo/OSiUnDueafSJ0p6SQ==
-----END CERTIFICATE-----
 1 s:/C=US/O=DigiCert Inc/OU=www.digicert.com/CN=DigiCert SHA2 High Assurance Server CA
   i:/C=US/O=DigiCert Inc/OU=www.digicert.com/CN=DigiCert High Assurance EV Root CA
-----BEGIN CERTIFICATE-----
MIIEsTCCA5mgAwIBAgIQBOHnpNxc8vNtwCtCuF0VnzANBgkqhkiG9w0BAQsFADBs
MQswCQYDVQQGEwJVUzEVMBMGA1UEChMMRGlnaUNlcnQgSW5jMRkwFwYDVQQLExB3
d3cuZGlnaWNlcnQuY29tMSswKQYDVQQDEyJEaWdpQ2VydCBIaWdoIEFzc3VyYW5j
ZSBFViBSb290IENBMB4XDTEzMTAyMjEyMDAwMFoXDTI4MTAyMjEyMDAwMFowcDEL
MAkGA1UEBhMCVVMxFTATBgNVBAoTDERpZ2lDZXJ0IEluYzEZMBcGA1UECxMQd3d3
LmRpZ2ljZXJ0LmNvbTEvMC0GA1UEAxMmRGlnaUNlcnQgU0hBMiBIaWdoIEFzc3Vy
YW5jZSBTZXJ2ZXIgQ0EwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC2
4C/CJAbIbQRf1+8KZAayfSImZRauQkCbztyfn3YHPsMwVYcZuU+UDlqUH1VWtMIC
Kq/QmO4LQNfE0DtyyBSe75CxEamu0si4QzrZCwvV1ZX1QK/IHe1NnF9Xt4ZQaJn1
itrSxwUfqJfJ3KSxgoQtxq2lnMcZgqaFD15EWCo3j/018QsIJzJa9buLnqS9UdAn
4t07QjOjBSjEuyjMmqwrIw14xnvmXnG3Sj4I+4G3FhahnSMSTeXXkgisdaScus0X
sh5ENWV/UyU50RwKmmMbGZJ0aAo3wsJSSMs5WqK24V3B3aAguCGikyZvFEohQcft
bZvySC/zA/WiaJJTL17jAgMBAAGjggFJMIIBRTASBgNVHRMBAf8ECDAGAQH/AgEA
MA4GA1UdDwEB/wQEAwIBhjAdBgNVHSUEFjAUBggrBgEFBQcDAQYIKwYBBQUHAwIw
NAYIKwYBBQUHAQEEKDAmMCQGCCsGAQUFBzABhhhodHRwOi8vb2NzcC5kaWdpY2Vy
dC5jb20wSwYDVR0fBEQwQjBAoD6gPIY6aHR0cDovL2NybDQuZGlnaWNlcnQuY29t
L0RpZ2lDZXJ0SGlnaEFzc3VyYW5jZUVWUm9vdENBLmNybDA9BgNVHSAENjA0MDIG
BFUdIAAwKjAoBggrBgEFBQcCARYcaHR0cHM6Ly93d3cuZGlnaWNlcnQuY29tL0NQ
UzAdBgNVHQ4EFgQUUWj/kK8CB3U8zNllZGKiErhZcjswHwYDVR0jBBgwFoAUsT7D
aQP4v0cB1JgmGggC72NkK8MwDQYJKoZIhvcNAQELBQADggEBABiKlYkD5m3fXPwd
aOpKj4PWUS+Na0QWnqxj9dJubISZi6qBcYRb7TROsLd5kinMLYBq8I4g4Xmk/gNH
E+r1hspZcX30BJZr01lYPf7TMSVcGDiEo+afgv2MW5gxTs14nhr9hctJqvIni5ly
/D6q1UEL2tU2ob8cbkdJf17ZSHwD2f2LSaCYJkJA69aSEaRkCldUxPUd1gJea6zu
xICaEnL6VpPX/78whQYwvwt/Tv9XBZ0k7YXDK/umdaisLRbvfXknsuvCnQsH6qqF
0wGjIChBWUMo0oHjqvbsezt3tkBigAVBRQHvFwY+3sAzm2fTYS5yh+Rp/BIAV0Ae
cPUeybQ=
-----END CERTIFICATE-----
---
Server certificate
subject=/C=US/ST=California/L=San Francisco/O=GitHub, Inc./CN=github.com
issuer=/C=US/O=DigiCert Inc/OU=www.digicert.com/CN=DigiCert SHA2 High Assurance Server CA
---
No client certificate CA names sent
Server Temp Key: ECDH, X25519, 253 bits
---
SSL handshake has read 3435 bytes and written 289 bytes
---
New, TLSv1/SSLv3, Cipher is ECDHE-RSA-AES128-GCM-SHA256
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES128-GCM-SHA256
    Session-ID: D3DEBDE2E2556A8B3A2FD17184081ED53F41F1BA252B8CA808320D4BF4D03CF8
    Session-ID-ctx:
    Master-Key: 1057E58DD379B7E4B0BDF73F8995EDB32E16DC80338F0829134EF918862FCEF192A08F205D9877DC9B1B818A3C0A0A33
    Start Time: 1607618103
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
---
closed
```

</details>

If your's doesn't come back the same, it could be that your machine is missing some root certificates as noted [here](https://docs.microsoft.com/en-us/troubleshoot/mem/configmgr/connectivity-issues-digicert-global-root-g2-not-installed).
```
depth=2 C = IE, O = Baltimore, OU = CyberTrust, CN = Baltimore CyberTrust Root
verify return:1
depth=1 C = US, O = DigiCert Inc, OU = www.digicert.com, CN = DigiCert Baltimore CA-2 G2
```

There are a couple of things that you can do (at your own risk):

1. Add the missing certs following the steps [here](https://www.hyperpac.de/?p=2359)

1. Work around the missing certs by:
    `git clone -c http.sslverify=false https://github.com/githubuniverseworkshops/grafting-monorepos.git`
   - A note here though, what it will do is ignore any SSL verifications your network/machine may have set up for **ONLY** this repository.

## :books: Resources

- Linux Kernel open source repository - [https://github.com/torvalds/linux](https://github.com/torvalds/linux)
- Git SCM reference manual - [https://git-scm.com/docs](https://git-scm.com/docs)
- Git Sizer - [https://github.com/github/git-sizer/#getting-started](https://github.com/github/git-sizer/#getting-started)
- Platform samples scripts - [https://github.com/github/platform-samples/tree/master/scripts](https://github.com/github/platform-samples/tree/master/scripts)
- Git filter repo analysis tool - [https://github.com/newren/git-filter-repo](https://github.com/newren/git-filter-repo)

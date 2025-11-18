# patch-tracker

Helper repository for tracking upstream changes in Sophgo and MilkV repositories and creating patches for aa-proxy Buildroot.

# Background
To understand the purpose of this repository, here's some context. The vendor (Sophgo) provides its clients with a template or base for a BSP (called SDK in MilkV). They then customize it for their needs.

We start with Linux upstream (LTS version with long-term maintenance). Sophgo takes it, adds its patches and drivers. Then MilkV, using Sophgo's SDK, also adds its own patches.

So, in the end, we have:
- Linux upstream
- Sophgo changes
- MilkV changes

Of course, it can also happen that we need to apply our own changes.

Sophgo publishes each element (kernel, bootloader, etc.) as a separate repository, while MilkV keeps everything in a single repository (details: [MilkV SDK version info](https://github.com/milkv-duo/duo-buildroot-sdk-v2/blob/main/.version/version.md)).

This repository was created to maintain order and provide full visibility for comparing branches at each stage. It was originally created during the porting of MilkV DuoS from the official BSP to our project/Buildroot setup and for our specific needs. However, the same workflow can be applied to other projects as well, so the explanations below can be used as a general how-to.

# How it works
Everything is organized into branches.

The main MilkV branch is `duo-main` — the main repository from MilkV ([duo-buildroot-sdk-v2](https://github.com/milkv-duo/duo-buildroot-sdk-v2)).

Example: extracting FSBL patches:

```bash
git checkout duo-main
# Split only the fsbl subdirectory into a new branch
git subtree split --prefix=fsbl -b duo-main-fsbl
```

Next, get the upstream (Sophgo):

```bash
git remote add sophgo-fsbl https://github.com/sophgo/fsbl.git
git fetch sophgo-fsbl
git checkout -b sophgo-main-fsbl sophgo-fsbl/sg200x-dev
```

Compare the branches:

```bash
git diff sophgo-main-fsbl..duo-main-fsbl
```

If Duo hasn't updated from Sophgo, create an auxiliary branch:

```bash
git checkout duo-main-fsbl
git checkout -b duo-main-fsbl-updated
git cherry-pick <list-of-changes-from-sophgo>
```

The easiest workflow is to add individual Sophgo repositories directly in Buildroot and apply MilkV patches + any of our own.

Finally, switch to the Sophgo branch and create a branch for final patches:

```bash
git checkout sophgo-main-fsbl
git checkout -b final-patches-fsbl
```

Cherry-pick MilkV + own changes. To generate the last two commits as patches:

```bash
git format-patch -2
```

Edit the files to remove `[... 1/2]` in the subject, leaving only `[PATCH]` for easier maintenance, comparison, and sorting in Buildroot.

# make-diff-lockbox

This project is for users of [Torizon OS](https://www.torizon.io/torizon-os) who are assumed to be familiar with the concept of a [Lockbox](https://developer.toradex.com/torizon/torizon-platform/torizon-updates/how-to-use-secure-offline-updates-with-torizoncore/#lockbox).

## Overview

This project provides a Bash script, namely `make-diff-lockbox`, designed to create a "differential" Lockbox – one containing the artifacts required for upgrading a device running a particular version of the OS (A) to a new version (B). Such a Lockbox would normally be considerably smaller than a regular/self-contained Lockbox for installing version B.

While a Lockbox typically contains multiple package types (OS, application, and generic), it should be noted that this script currently only supports Lockboxes containing an OSTree repository (it may contain other artifacts/packages though).


## Dependencies

For running the script, ensure the following dependencies are installed (for Debian):

- coreutils
- findutils
- sed
- grep
- ostree


## Usage

```bash
make-diff-lockbox LOCKBOX_A LOCKBOX_B LOCKBOX_DIFF [OPTIONS]
```

To learn about all available options, run:

```bash
make-diff-lockbox --help
```

## General workflow

To run `make-diff-lockbox`, users must provide two Lockboxes which need to be created (defined and downloaded) beforehand.

Assuming the input Lockboxes are available in directories `lockbox_a` and `lockbox_b`, corresponding to versions A and B of the OS, respectively, the A→B differential Lockbox would be created in directory `lockbox_diff` by running:

```bash
make-diff-lockbox lockbox_a lockbox_b lockbox_diff
```

The output Lockbox would be produced in directory `lockbox_diff`. Users can then install it on a device in place of `lockbox_b` provided that the device is running the equivalent of `lockbox_a` (i.e. version A of the OS). Note that devices not running that version would fail to update via the differential Lockbox.

By default, the script creates a Lockbox leveraging the static-delta feature of OSTree (which normally provides large reductions in Lockbox size). Therefore, a prerequisite for the installation to succeed is that the OS must have support for loading Lockboxes containing static-deltas. That support exists in Torizon OS since version `7.6.0-devel-20260327+build.428`.

> [!WARNING]
> For properly loading Lockboxes with static-deltas, a device must be running Torizon OS version `7.6.0-devel-20260327+build.428` or newer.


## How the script works

Here's an outline of the operations performed by the script **when using static-deltas**:

- Load the commits from both input OSTree repositories into a temporary repository.
- Generate the static-deltas between the two commits inside the temporary repository.
- Repeat the following operations multiple times (simulation runs):
  - Create a "pull-test" repository and import the first commit.
  - Pull the second commit into the "pull-test" repository, collecting the list of files fetched during the operation.
- Produce the final repository containing only the static-deltas and any files accessed during the "pull-test" operations.

The reason for performing these simulation runs is to address the non-deterministic behavior of OSTree when fetching multiple artifacts in parallel.

The operations performed by the script **when not using static-deltas** are just:

- Go over each object present in the second OSTree repository checking if the same file exists in the first repository and collecting that information.
- Produce the output repository including only the files that exist in the second repository but do not exist in the first one.


## Known limitations

- Only the OSTree part of a Lockbox is handled by the script; it will fail if the Lockbox does not contain an OSTree repository in it.


## License

This project is licensed under the terms of the MIT license - see the LICENSE file for details.

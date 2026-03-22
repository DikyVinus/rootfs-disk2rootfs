rootfs-disk2rootfs

Convert arbitrary Linux root filesystem images into a clean, normalized, compressed rootfs archive.

---

Overview

"rootfs-disk2rootfs" ingests heterogeneous rootfs sources and produces a standardized output:

- Disk images: ".img", ".raw", ".qcow2"
- Archives: ".tar", ".tar.gz", ".tar.xz", ".tar.zst"
- Containers: ".zip", ".7z"

It extracts the actual filesystem, removes container overhead, and outputs a normalized ".tar.xz" rootfs.

---

Key Features

- Converts disk → filesystem → tar
- Removes:
  - unused blocks
  - partition tables
  - container artifacts
- Preserves:
  - ownership (UID/GID)
  - permissions
  - symlinks
  - xattrs / ACLs
- Recompresses using multithreaded "xz"

---

Why Use This

Official rootfs/images often include:

- padding / unused space
- embedded partition layouts
- suboptimal compression
- container-specific structure

This tool:

- reduces size (~10–70% typical)
- produces deterministic output
- standardizes rootfs for:
  - "chroot"
  - containers
  - CI pipelines

---

Supported Inputs

Type| Examples
Tar archives| ".tar", ".tar.gz", ".tar.xz", ".tar.zst"
Disk images| ".img", ".raw", "disk.raw"
VM images| ".qcow2", ".qcow"
Containers| ".zip", ".7z"

---

Output

- Format: "rootfs.tar.xz"
- Characteristics:
  - normalized filesystem
  - no disk/container layers
  - ready for "chroot", containers, or repackaging

---

Internal Workflow

1. Download input
2. Extract archive (if applicable)
3. Convert "qcow → raw"
4. Detect filesystem:
   - direct mount (ext4 / xfs / f2fs)
   - or partition scan ("losetup")
5. Copy filesystem contents
6. Remove intermediate artifacts
7. Repack using:
   - "tar"
   - "xz -T0 -3"

---

Usage

Extract rootfs

sudo tar --numeric-owner -xJf rootfs.tar.xz -C rootfs

Enter chroot

sudo mount --bind /dev rootfs/dev
sudo mount --bind /proc rootfs/proc
sudo mount --bind /sys rootfs/sys
sudo chroot rootfs /bin/bash

---

Compression Strategy

Default:

xz -T0 -3

- multithreaded
- balanced speed vs size

Alternatives:

- "-6" → smaller, slower
- "-1" → faster, larger
- "zstd" → faster, slightly larger output

---

Limitations

- Requires root privileges for:
  - extraction
  - mounting
  - ownership restoration
- Large images may exceed CI disk limits
- Non-Linux filesystems are not supported

---

Safety Model

- Intermediate disk images are removed only after successful extraction
- Rootfs validity is verified ("/etc", "/bin")
- Cleanup is bounded to prevent filesystem corruption

---

Typical Size Reduction

Input Type| Reduction
Raw disk images| 30–70%
".tar.gz" → ".tar.xz"| 10–30%
Already optimized ".tar.xz"| minimal

---

Use Cases

- CI pipelines
- container base image generation
- chroot environments
- rootfs normalization
- converting VM images → portable rootfs

---

Summary

"rootfs-disk2rootfs" transforms heterogeneous rootfs sources into a clean, minimal, and portable archive by removing disk-layer overhead and applying consistent compression.

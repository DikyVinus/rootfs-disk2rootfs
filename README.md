rootfs-disk2rootfs

Convert arbitrary Linux root filesystem images into a clean, normalized, compressed rootfs archive.

---

Overview

"rootfs-disk2rootfs" ingests various rootfs sources:

- disk images (".img", ".raw", ".qcow2")
- archives (".tar", ".tar.gz", ".tar.xz", ".tar.zst")
- mixed containers (".zip", ".7z")

It extracts the actual filesystem, removes container overhead, and outputs a normalized ".tar.xz" rootfs.

---

Key Behavior

- Converts disk → filesystem → tar
- Removes unused blocks and container artifacts
- Preserves:
  - ownership (UID/GID)
  - permissions
  - symlinks
  - xattrs / ACLs
- Recompresses using optimized multithreaded "xz"

---

Why Use This

Official rootfs or images often include:

- padding / unused space
- partition tables
- inefficient compression
- container-specific structure

This tool:

- reduces size (commonly 10–70%)
- produces deterministic filesystem layout
- standardizes output for chroot / containers

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
- Properties:
  - normalized filesystem
  - no disk/container layers
  - ready for "chroot", containers, or repackaging

---

Workflow (internal)

1. Download input
2. Extract archive (if needed)
3. Convert qcow → raw
4. Detect filesystem:
   - direct mount (ext4/xfs/f2fs)
   - or partition scan via "losetup"
5. Copy filesystem contents
6. Remove intermediate disk/container files
7. Repack with:
   - "tar"
   - "xz -T0 -3"

---

Example Usage

Extract and repack

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

Alternative:

- "-6" → smaller, slower
- "zstd" → faster, slightly larger

---

Limitations

- Requires root privileges for:
  - extraction
  - mounting
  - correct ownership restoration
- Very large images may exceed CI disk limits
- Non-Linux filesystems are not supported

---

Safety Model

- Intermediate disk images are removed only after successful extraction
- Rootfs validity is checked ("/etc", "/bin")
- Cleanup is bounded to avoid filesystem corruption

---

Typical Size Reduction

Input| Reduction
Raw disk images| 30–70%
".tar.gz" → ".tar.xz"| 10–30%
Already optimized ".tar.xz"| minimal

---

Use Cases

- CI pipelines
- container base generation
- chroot environments
- rootfs normalization
- converting VM images into portable rootfs

---

Summary

"rootfs-disk2rootfs" converts heterogeneous rootfs sources into a clean, minimal, and portable archive by stripping disk-layer overhead and applying consistent compression.

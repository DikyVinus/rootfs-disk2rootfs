# rootfs-disk2rootfs

Lightweight rootfs normalization tool that converts arbitrary Linux images into a clean, portable `.tar.xz` filesystem.

---

### Overview

`rootfs-disk2rootfs` ingests mixed rootfs formats and reconstructs them into a normalized archive suitable for:

- chroot environments  
- container bases  
- CI pipelines  

It removes disk/container overhead and standardizes filesystem layout.

---

# Supported Inputs

#### Disk Images
- `.img`  
- `.raw`  
- `disk.raw`  

#### Virtual Machine Images
- `.qcow2`  
- `.qcow`  

#### Tar Archives
- `.tar`  
- `.tar.gz`  
- `.tar.xz`  
- `.tar.zst`  

#### Container Archives
- `.zip`  
- `.7z`  

---

# Processing Model

#### Step-by-step pipeline

1. Download input  
2. Extract archive (if applicable)  
3. Convert `qcow → raw`  
4. Detect filesystem:
   - direct mount (ext4 / xfs / f2fs)  
   - or partition scan (`losetup`)  
5. Copy filesystem contents  
6. Remove intermediate artifacts  
7. Repack into compressed rootfs  

---

# Output

#### Archive format
- `rootfs.tar.xz`

#### Properties
- no partition tables  
- no unused blocks  
- no container layers  
- normalized filesystem structure  

---

# Compression

#### Default
```bash
xz -T2 -3
```

multithreaded

balanced speed / size


### Alternatives

xz -1 → faster, larger

xz -6 → slower, smaller

zstd → fastest, slightly larger



---

### Filesystem Integrity

Preserved attributes:

ownership (UID/GID)

permissions

symlinks

xattrs

ACLs



---

# Usage

### Extract rootfs
```bash
sudo tar --numeric-owner -xJf rootfs.tar.xz -C rootfs
```

### Enter chroot
```bash
sudo mount --bind /dev rootfs/dev
sudo mount --bind /proc rootfs/proc
sudo mount --bind /sys rootfs/sys
sudo chroot rootfs /bin/bash
```


---

# Size Characteristics

### Typical reduction

raw disk images → 30–70% smaller

.tar.gz → .tar.xz → 10–30% smaller

.tar.xz → minimal


### Source of reduction

removal of unused blocks

elimination of padding

stronger compression



---

# Limitations

### requires root privileges:

extraction

mounting

metadata restoration


large images may exceed CI disk limits

non-Linux filesystems unsupported



---

### Safety Model

intermediate images removed only after successful extraction

rootfs validity checked (/etc, /bin)

cleanup bounded to prevent corruption



---

### Use Cases

CI pipelines

container base generation

chroot environments

VM image conversion

rootfs normalization



---

### Summary

rootfs-disk2rootfs converts heterogeneous rootfs inputs into a minimal, portable archive by removing disk-level structures and applying consistent compression.

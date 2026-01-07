# File Systems Deep Dive - Complete Understanding

## Table of Contents
1. [What is a File System?](#what-is-a-file-system)
2. [File System Structure and Organization](#file-system-structure-and-organization)
3. [Inodes - File Metadata](#inodes---file-metadata)
4. [Directory Structure and Navigation](#directory-structure-and-navigation)
5. [File Operations - Read, Write, Delete](#file-operations---read-write-delete)
6. [File System Types - FAT, NTFS, ext4, etc.](#file-system-types---fat-ntfs-ext4-etc)
7. [Journaling - Ensuring Consistency](#journaling---ensuring-consistency)
8. [File Permissions and Access Control](#file-permissions-and-access-control)
9. [File System Performance](#file-system-performance)
10. [Distributed File Systems](#distributed-file-systems)

---

## What is a File System?

### The Problem

**Raw Disk:**
```
Just blocks of data
No organization
No way to find files
No structure
```

**Need:**
- **Organization**: Structure for files
- **Naming**: Name files
- **Location**: Find files
- **Metadata**: Size, permissions, dates

### The Solution: File System

**File System**: Method of storing and organizing files on storage device.

**Functions:**
- **Organize data**: Structure for files and directories
- **Track files**: Know where files are stored
- **Manage space**: Allocate and free space
- **Provide interface**: Read, write, delete files

### Real-World Analogy

**File System = Library Organization:**
- **Books**: Files
- **Shelves**: Directories
- **Catalog**: File system metadata
- **Librarian**: File system manages access

---

## File System Structure and Organization

### Basic Components

**1. Superblock:**
```
File system metadata
Size, type, block size
Location of inodes
```

**2. Inode Table:**
```
File metadata
One inode per file
```

**3. Data Blocks:**
```
Actual file data
Content stored here
```

**4. Directory Entries:**
```
Map names to inodes
Directory structure
```

### File System Layout

```
┌─────────────┐
│ Superblock  │ File system info
├─────────────┤
│ Inode Table │ File metadata
├─────────────┤
│ Data Blocks │ File content
├─────────────┤
│ Free Blocks │ Available space
└─────────────┘
```

---

## Inodes - File Metadata

### What is an Inode?

**Inode (Index Node)**: Data structure storing file metadata.

**Contains:**
- **File type**: Regular, directory, symlink
- **Permissions**: Read, write, execute
- **Owner**: User and group
- **Size**: File size in bytes
- **Timestamps**: Created, modified, accessed
- **Pointers**: To data blocks

### Inode Structure

```
Inode:
  - File type and permissions
  - Owner (UID, GID)
  - Size
  - Timestamps
  - Direct blocks (pointers to data)
  - Indirect blocks (pointers to blocks of pointers)
  - Double indirect blocks
  - Triple indirect blocks
```

### How Inodes Work

**File Access:**
```
1. Look up filename in directory
2. Get inode number
3. Read inode
4. Follow pointers to data blocks
5. Read file content
```

**Example:**
```
File: /home/user/document.txt
1. Look up "document.txt" in /home/user directory
2. Get inode number: 12345
3. Read inode 12345
4. Inode says: Data in blocks 100, 101, 102
5. Read blocks 100, 101, 102
6. Return file content
```

---

## Directory Structure and Navigation

### What is a Directory?

**Directory**: Special file containing list of files and their inode numbers.

**Structure:**
```
Directory entry:
  - Filename
  - Inode number
```

**Example:**
```
/home/user/ directory:
  document.txt → inode 12345
  photo.jpg → inode 12346
  subdir/ → inode 12347
```

### Directory Tree

**Hierarchical Structure:**
```
/ (root)
├── home/
│   ├── user1/
│   │   ├── documents/
│   │   └── photos/
│   └── user2/
├── etc/
├── var/
└── usr/
```

**Path Resolution:**
```
/home/user/document.txt

1. Start at root (/)
2. Look up "home" → inode
3. Read home directory
4. Look up "user" → inode
5. Read user directory
6. Look up "document.txt" → inode
7. Read file
```

---

## File Operations - Read, Write, Delete

### Reading a File

**Process:**
```
1. Open file: Get inode number
2. Read inode: Get data block pointers
3. Read data blocks: Get file content
4. Close file: Release resources
```

**Optimization:**
- **Buffer cache**: Cache frequently accessed files
- **Read ahead**: Prefetch next blocks
- **Block size**: Read in blocks, not bytes

### Writing a File

**Process:**
```
1. Open file: Get inode number
2. Allocate data blocks: Find free blocks
3. Write data: Write to blocks
4. Update inode: Update size, timestamps
5. Update directory: If new file
6. Close file: Flush buffers
```

**Challenges:**
- **Allocation**: Find free blocks
- **Fragmentation**: Blocks may be scattered
- **Consistency**: Ensure data written correctly

### Deleting a File

**Process:**
```
1. Remove directory entry: Unlink filename
2. Decrement link count: In inode
3. If link count = 0: Free inode and blocks
4. Mark blocks as free: Add to free list
```

**Note:**
- **Data not immediately erased**: Just marked as free
- **Recovery possible**: Until overwritten

---

## File System Types - FAT, NTFS, ext4, etc.

### FAT (File Allocation Table)

**Characteristics:**
- **Simple**: Basic file system
- **Compatible**: Works on many systems
- **Limitations**: File size, fragmentation

**Structure:**
```
FAT: Table mapping clusters
Directory: File entries
Data: Clusters
```

### NTFS (New Technology File System)

**Characteristics:**
- **Windows**: Default on Windows
- **Advanced**: Journaling, permissions, compression
- **Large files**: Supports very large files

**Features:**
- **Journaling**: Consistency
- **Permissions**: Access control
- **Compression**: Transparent compression

### ext4 (Fourth Extended File System)

**Characteristics:**
- **Linux**: Default on many Linux systems
- **Journaling**: Consistency
- **Large files**: Up to 16 TB files
- **Large volumes**: Up to 1 EB volumes

**Features:**
- **Extents**: Efficient allocation
- **Delayed allocation**: Better performance
- **Journaling**: Consistency

### Comparison

| Feature | FAT32 | NTFS | ext4 |
|---------|-------|------|------|
| **Max file size** | 4 GB | 16 EB | 16 TB |
| **Max volume** | 2 TB | 256 TB | 1 EB |
| **Journaling** | No | Yes | Yes |
| **Permissions** | Basic | Advanced | Advanced |
| **OS** | All | Windows | Linux |

---

## Journaling - Ensuring Consistency

### The Problem

**Crash During Write:**
```
Writing file
System crashes
File system inconsistent
Data corruption
```

### The Solution: Journaling

**Journaling**: Log changes before applying.

**Process:**
```
1. Write to journal: "I'm about to write block X"
2. Write actual data: Write to file system
3. Mark journal entry complete: "Write done"
```

**On Recovery:**
```
Read journal
Replay completed entries
File system consistent
```

### Journaling Modes

**1. Writeback:**
```
Journal metadata only
Faster, but data may be lost
```

**2. Ordered:**
```
Journal metadata
Write data first, then metadata
Balanced
```

**3. Data:**
```
Journal everything
Safest, but slower
```

---

## File Permissions and Access Control

### Unix Permissions

**Three Types:**
- **Read (r)**: Can read file
- **Write (w)**: Can modify file
- **Execute (x)**: Can execute file

**Three Classes:**
- **Owner**: File owner
- **Group**: File group
- **Others**: Everyone else

**Example:**
```
-rwxr-xr--  user group  file.txt
│││││││││
│││└─┴─┴─ Others: r--
│└─┴─┴─ Group: r-x
└─┴─┴─ Owner: rwx
```

### Access Control Lists (ACL)

**Extended Permissions:**
```
More granular control
Specific users/groups
Beyond owner/group/others
```

---

## File System Performance

### Factors Affecting Performance

**1. Block Size:**
```
Larger blocks: Fewer I/O operations, but more waste
Smaller blocks: More I/O, but less waste
```

**2. Fragmentation:**
```
Files scattered across disk
Slower access
Defragmentation needed
```

**3. Caching:**
```
Buffer cache: Cache frequently accessed files
Reduces disk I/O
```

**4. Allocation Strategy:**
```
Contiguous: Fast but hard to find space
Linked: Flexible but slower
Indexed: Balance
```

---

## Distributed File Systems

### What are Distributed File Systems?

**Distributed File System**: File system spread across multiple servers.

**Examples:**
- **NFS**: Network File System
- **HDFS**: Hadoop Distributed File System
- **GlusterFS**: Distributed file system

**Challenges:**
- **Consistency**: Multiple copies
- **Performance**: Network latency
- **Availability**: Server failures

---

## Summary

File systems are fundamental to how operating systems organize and access data. Understanding file system structure, inodes, operations, and types is essential for backend engineers.

**Key Takeaways:**
- File systems organize data on storage
- Inodes store file metadata
- Directories map names to inodes
- Different file systems for different needs
- Journaling ensures consistency
- Permissions control access
- Performance depends on many factors
- Distributed file systems for scale

**Next Steps:**
- Understand your system's file system
- Learn file system tools
- Monitor file system performance
- Plan for capacity and performance


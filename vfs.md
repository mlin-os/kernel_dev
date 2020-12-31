### Reading List

1. https://www.kernel.org/doc/html/latest/filesystems/vfs.html

### Questions List

1. What's the relationship of inode, dentry, file superblock and etc.?

2. How to register and mount a filesystem?

### Flowcharts content

* `register_filesystem`, `find_filesystem`, `file_systems_lock`, `file_systems`;
* `struct file_system_type` and fields `next` and `struct hlist_head fs_supers`;

### [Introduction to VFS](https://www.kernel.org/doc/html/latest/filesystems/vfs.html)?

> The Virtual File System (VFS) is the software layer in the kernel that provides the filesystem interface to userspace programs. It also provides an abstraction within the kernel which allows different filesystem implementations to coexist.

#### Questions

1. What's the meaning of the first part?

### [Directory Entry Cache](https://www.kernel.org/doc/html/latest/filesystems/vfs.html)

> directory entry cache or dcache provides a very fast look-up mechanism to translate a pathname (filename) into a specific dentry. Dentries live in RAM and are never saved to disc: they exist only for performance.

#### Questions

1. How does dcache work? recursively

### [The Inode Object](https://www.kernel.org/doc/html/latest/filesystems/vfs.html)

> An inode object represents an object within the filesystem.
> Inodes are filesystem objects such as regular files, directories, FIFOs and other beasts. They live either on the disc (for block devices filesystem) or in the memory (for pseudo filesystems).
> An individual dentry usually has a pointer to an inode. A single inode can be pointed to by multiple dentries (hard links).

#### Questions

1. What's the relationship between inode, dentry and file?

### [The File Object](https://www.kernel.org/doc/html/latest/filesystems/vfs.html)

> Opening a file requires another operation: allocation of a file structure (this is the kernel-side implementation of file descriptors). The freshly allocated file structure is initialized with a pointer to the dentry and a set of file operation member functions.

### [The Superblock Object](https://www.kernel.org/doc/html/latest/filesystems/vfs.html)

> A superblock object represents a mounted filesystem.

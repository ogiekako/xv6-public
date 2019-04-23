主なデータ構造 (struct) をメモする。さらに数を絞ったものは、[important structs](important_structs.md) を参照。

bio.c

```
// Disk buffer cache.
struct {
  struct spinlock lock;
  struct buf buf[NBUF];

  // Linked list of all buffers, through prev/next.
  // head.next is most recently used.
  struct buf head;
} bcache;
```

buf.h

```
// Represents a 512-byte block
struct buf {
  int flags;
  uint dev;
  uint blockno;
  struct sleeplock lock;
  uint refcnt;
  struct buf *prev; // LRU cache list
  struct buf *next;
  struct buf *qnext; // disk queue
  uchar data[BSIZE];
};
```

file.c

```
// All open files in the system.
struct {
  struct spinlock lock;
  struct file file[NFILE];
} ftable;
```

file.h

```
// Each open file.
struct file {
  enum { FD_NONE, FD_PIPE, FD_INODE } type;
  int ref; // reference count
  char readable;
  char writable;
  struct pipe *pipe;
  struct inode *ip;
  uint off;
};
```

```
// In-memory copy of a struct dinode on disk.
struct inode {
  uint dev;           // Device number
  uint inum;          // Inode number
  int ref;            // Reference count
  struct sleeplock lock;
  int flags;          // I_VALID

  short type;         // copy of disk inode
  short major;
  short minor;
  short nlink;
  uint size;
  uint addrs[NDIRECT+1];
};
```

fs.c

```
// In memory inode cache.
struct {
  struct spinlock lock;
  struct inode inode[NINODE];
} icache;
```

fs.h

```
// Metadata about file system stored in the block 1.
struct superblock {
  uint size;         // Size of file system image (blocks)
  uint nblocks;      // Number of data blocks
  uint ninodes;      // Number of inodes.
  uint nlog;         // Number of log blocks
  uint logstart;     // Block number of first log block
  uint inodestart;   // Block number of first inode block
  uint bmapstart;    // Block number of first free map block
};
```

```
// On-disk inode structure
struct dinode {
  short type;           // File type
  short major;          // Major device number (T_DEV only)
  short minor;          // Minor device number (T_DEV only)
  short nlink;          // Number of links to inode in file system
  uint size;            // Size of file (bytes)
  uint addrs[NDIRECT+1];   // Data block addresses
};
```

```
// Directory is a file containing a sequence of dirent structures.
struct dirent {
  ushort inum;
  char name[DIRSIZ];
};
```

ioapic.c

```
// IO APIC (Advanced programmable interrupt controller) MMIO (memory mapped I/O) structure: write reg, then read or write data.
struct ioapic {
  uint reg;
  uint pad[3];
  uint data;
};
```

kalloc.c

```
// Represents a free page.
struct run {
  struct run *next;
};
```

```
// Starting point of the free memory list.
struct {
  struct spinlock lock;
  int use_lock;
  struct run *freelist;
} kmem;
```

log.c

```
// Contents of the header block, used for both the on-disk header block
// and to keep track in memory of logged block# before commit.
struct logheader {
  int n;
  int block[LOGSIZE];
};

struct log {
  struct spinlock lock;
  int start;
  int size;
  int outstanding; // how many FS sys calls are executing.
  int committing;  // in commit(), please wait.
  int dev;
  struct logheader lh;
};
```

mmu.h

```
// Segment Descriptor
struct segdesc {
  ...
};
```

```
// Task state segment format
struct taskstate {
  ...
};
```

```
// Gate descriptors for interrupts and traps
struct gatedesc {
  ...
};
```

mp.h (multi processor)

```
struct mp {             // floating pointer
  uchar signature[4];           // "_MP_"
  void *physaddr;               // phys addr of MP config table
  ...
};

struct mpconf {         // configuration table header
  ...
};

struct mpproc {         // processor table entry
...
};

struct mpioapic {       // I/O APIC table entry
...
};
```

pipe.c

```
struct pipe {
  ...
}
```

proc.c

```
// Processes table.
struct {
  struct spinlock lock;
  struct proc proc[NPROC];
} ptable;
```

proc.h

```
// Per-CPU state
struct cpu {
  uchar apicid;                // Local APIC ID
  struct context *scheduler;   // swtch() here to enter scheduler
  struct taskstate ts;         // Used by x86 to find stack for interrupt
  struct segdesc gdt[NSEGS];   // x86 global descriptor table
  volatile uint started;       // Has the CPU started?
  int ncli;                    // Depth of pushcli nesting.
  int intena;                  // Were interrupts enabled before pushcli?
  struct proc *proc;           // The process running on this cpu or null
};
```

```
struct context {
  uint edi;
  uint esi;
  uint ebx;
  uint ebp;
  uint eip;
};
```

```
// Per-process state
struct proc {
  uint sz;                     // Size of process memory (bytes)
  pde_t* pgdir;                // Page table
  char *kstack;                // Bottom of kernel stack for this process
  enum procstate state;        // Process state
  int pid;                     // Process ID
  struct proc *parent;         // Parent process
  struct trapframe *tf;        // Trap frame for current syscall
  struct context *context;     // swtch() here to run process
  void *chan;                  // If non-zero, sleeping on chan
  int killed;                  // If non-zero, have been killed
  struct file *ofile[NOFILE];  // Open files
  struct inode *cwd;           // Current directory
  char name[16];               // Process name (debugging)
};
```

sleeplock.h

```
// Long-term locks for processes
struct sleeplock {
  uint locked;       // Is the lock held?
  struct spinlock lk; // spinlock protecting this sleep lock
  
  // For debugging:
  char *name;        // Name of lock.
  int pid;           // Process holding lock
};
```

spinlock.h

```
// Mutual exclusion lock.
struct spinlock {
  uint locked;       // Is the lock held?

  // For debugging:
  char *name;        // Name of lock.
  struct cpu *cpu;   // The cpu holding the lock.
  uint pcs[10];      // The call stack (an array of program counters)
                     // that locked the lock.
};
```

stat.h

```
// Information about the object a file descriptor refers to.
struct stat {
  short type;  // Type of file
  int dev;     // File system's disk device
  uint ino;    // Inode number
  short nlink; // Number of links to file
  uint size;   // Size of file in bytes
};
```

x86.h

```
// Layout of the trap frame built on the stack by the
// hardware and by trapasm.S, and passed to trap().
struct trapframe {
  // registers as pushed by pusha
  ...

  // rest of trap frame
  ...

  // below here defined by x86 hardware
  ...

  // below here only when crossing rings, such as from user to kernel
  ...
};
```

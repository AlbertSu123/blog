---
title: 'Operating Systems'
date: '2021-10-22'
tags: ['operating-systems', 'class', 'berkeley']
draft: false
summary: 'Notes from CS 162 from UC Berkeley. Contains information about Address Translation, Caching, Distributed Systems, File Systems, Input/Output, Memory Management, Multiprocessor Scheduling, Queuing Theory, Real Time Scheduling, Reliable Storage, Uniprocessor Scheduling, and Virtual Memory.'
---

# Address Translation

## Motivation

- Address Translation allows for process isolation, interprocess communication(via sharing a common memory region)
- Shared code segments(ie sharing a library between programs)
- Program initialization(starting program before all code is loaded from memory to disk)
- Efficient dynamic memory allocation (only allocate memory for what is needed)
- Cache management (change how programs are positioned in physical memory to improve cache efficiency)
- Program Debugging (address translation prevents the rewriting of the code section, making things easier to debug)
- Efficient I/O (allows data to be safely transferred between user mode apps and I/O devices)
- Memory Mapped Files (map files into address space so contents of files can be referenced with program instructions)
- Virtual memory (fool the application into thinking that there is more memory than is physically present on computer)
- Checkpointing + Restarting (Can restart program from the saved state)
- Persistent Data Structures (Region of memory that survives program + system crashes)
- Process Migration (can move executing program from on server to another, ie load balancing)
- Distributed Storage (Turn network of servers into shared memory using address translation)

## Possible Places to put address translation implementation

- Specialized hardware in system
- trusted compiler, linker, or byte-code interpreter
- application translates pointers
- hybrid, with address translations in software and hardware

## Simple Address Translation

- At its core, it is a black box that translates a virtual address to a physical address
- Possible implementations could be an array, a tree, a hash table, or pretty much anything

**Goals of an address translation black box:**

1. **Memory Protection -** Limit access of a process to certain regions of memory(ie code)
2. **Memory Sharing -** Allow multiple processes to share memory (ie program code, file)
3. **Flexible Memory placement -** Put process anywhere to pack physical memory + use caches better
4. **Sparse Addresses -** Programs have multiple dynamic memory regions such as the heap for data objects, a stack for each thread, and memory mapped files
5. **Runtime lookup efficiency -** Fetching addresses should be fast
6. **Compact Translation Table -** Black box should not take much memory
7. **Portability -** Works on all types of architectures

## Symbol Table and Dynamically Linked Library(DLL)

- **Problem:** Multiprogramming requires address translation for absolute addresses, such as goto the start of the procedure or loading a global variable.
- Old Solution: Early OS used relocating loaders, after picking an empty region of physical memory, the loader would modify instructions that used an absolute addresses. The modifications were tracked in the _symbol table_.
- Current Solution: Generate the symbol table, which tracks which values need to be modified when files are assembled together by the linker. Also, commercial operating systems use a dynamically linked library, which links a library into a program when the program first calls into the library.

## Flexible Address Translation

- The simplest model of memory management is base + bound. Each process has a base and bound to put everything between.

## Segments

- **Segmentation:** Instead of keeping only one pair of base and bound, we can instead keep an array of base and bounds.
- Segments are contiguous in memory, however, the different segments are not contiguous to each other, this would defeat the point. There will likely be gaps in the segments.
- A **segmentation fault** is a failure to a part of memory outside the legal segments.
- Segments are powerful, they are used in x86 and allows two processes to share code as their two segment tables point to the same base and bounds for their segments. They can also be used to share library routines.
- **Fork:** Makes a copy of the parent's segment table then copy the data there.
- **Zero on reference:** The operating system allocates memory for the heap but only zeroes out the first few kb of the segment. It then sets a bound register to limit the program to the zeroed part of memory. If the program wants more memory, the OS will zero out existing memory before resuming execution.
- **Cons:** There is a large overhead for managing the large number of variable size memory segments. Also, external fragmentation can occur where the small chunks between the segments have enough free space for a new application but the free space is not contiguous.
- **Questions left:** How much memory should we allocate per segment?

## Paged Segments

- **Paged Memory:** Memory that is allocated in fixed sized chunks called page frames.
- **Page Table:** Similar to segment table,its entries contain pointers to page frames. No need for a bound since all page sizes are the same.
- To share memory between processes, we set the page table entry for each process sharing a page to point to the same physical page frame using a **core map** - a data structure that tracks info about each physical page frame such as which page table entries point to it.
- **Data breakpoint:** request to stop the execution of a program when it references or modifies a memory location. It is implemented by making that location read only, so that every change can check if the instruction causing the write on read-only memory exception is the breakpoint.
- **Cons:** While physical memory management is easier, virtual address space becomes more challenging since compilers expect the stack to be contiguous(for virtual addresses) and of arbitrary size.
- **Questions:** How big should a page frame be? A larger page frame wastes space if not all memory inside frame is used. A small page frame means there are tons of page frames, meaning that the page table is very large.

# Multi Level Translation

- Instead of using an array for lookup, we can use a tree instead for the following reasons.
- **Efficient Memory Allocation:** Use a bitmap to store free space
- **Efficient Disk Transfers:** Hardware disks are partitioned into fixed size regions known as _sectors_, we can make page size a multiple of the _sector_ to simplify transfers, loading, reading, and writing.
- **Efficient Lookup:** Use a _translation lookaside buffer_ to make lookups fast
- **Efficient Reverse Lookup:** Easy to go from physical page frame to set of virtual addresses. This is crucial for the infinite virtual memory illusion.
- **Page protection and sharing granularity:** Each page table entry has its own access permissions

## Paged Segmentation

- Only two levels in the tree.
- Each segment table entry points to a page table, which points to the memory backing that segment.

### Multi Level Paged Segmentation

- Use a _Global Descriptor Table_, aka a segment table.
- For 32 bit systems, First 10 bits-page directory, Second 10 bits-second level page table, final 12 bits-offset within a page
- For 64 bit systems, only the first 48 bits are used

## Portability

## Motivation

Main memory density is increasing by almost a bit per year, for a multi-level page table to map all of memory, an extra level of the page table is needed every decade to keep up with main memory size.

For things like copy-on-write, zero-on-reference, fill-on-reference, we need a different view for hardware and virtual address space.

We also need:

- **List of memory objects:** We need to known which memory region represents program code, library code, shared data, copy-on-write, or memory-mapped file.
- **Virtual to physical translation:** In exceptions and system call parameter copying, the kernel needs to translate virtual to physical addresses to check if invalid page is truly invalid. (it could be just not yet loaded or doing some copy-on-write magic)
- **Physical to virtual translation:** The core map. Which processes map to a specific physical memory location so that when the kernel updates a page status, it also updates every page table entry that refers to that page.
- OSX uses a hash table rather than a tree for software translation data. This hash table is known as an inverted hash table

```
inverted page table entry = \{
  process or memory object ID,
  virtual page number,
  physical page frame number,
  access permissions
\}
```

## Efficient Address Translation

## Motivation

Page tables, Segmentations, etc. all require multiple extra memory references before reaching the physical memory location. This greatly increases the overhead per instruction.

Thus, we can use a cache to improve performance. A cache stores certain data to increase the speed of future requests, usually by taking advantage of temporal locality or spatial locality.

## Translation Lookaside Buffer (TLB)

- **Problem:** Instructions are usually next to each other both in virtual memory and physical memory, it would be pointless to load instructions one at a time from physical memory since they are on the same page in physical memory.
- **TLB:** A small hardware table that contains results of a recent address translation. Each entry maps a virtual page to a physical page.

```
TLB entry = \{
          virtual page number,
          physical page frame number,
          access permissions
            \}
```

- On a lookup, the TLB hardware checks all of the entries against the virtual page. If there is a match, the TLB returns the physical address. (TLB hit) If none of the entries match, the hardware does the full address translation and is added to the TLB, usually replacing the least recently used entry.

- The cost for a TLB lookup is  
  `Cost (address translation) = Cost (TLB lookup) + Cost (full translation) × (1 - P(hit))`

## Superpages

- **Superpage:** Set of contiguous pages in physical memory that map to the same contiguous region in virtual memory. For example, two 4 Kb pages that are next to each other form an 8 Kb superpage.
- **Pros:** Reduces the number of TLB entries needed since one superpage can map many pages.
- **Cons:** Complicates the OS by requiring the system to allocate chunks of memory in different sizes.
- **Implementation:** Each entry in the TLB has a flag signifying whether the entry is a page or superpage. When matching against a superpage, the TLB ignores the offset within a superpage and only considers the most significant bits.
- **Uses:** When the computer display is being withdrawn, a normal TLB isn't big enough to hold all the entries. Large Matrices in scientific code can be sped up via superpages.

## TLB Consistency Issues

1. **Process context switch:** The virtual addresses of the old process are no longer valid. Otherwise, the new process would be reading the old process' data structures, causing it to crash or even worse, reading sensitive info such as passwords stored in memory.

We need to get rid of the TLB's copies of the old process' page translations and permission. We can flush the TLB(discard all contents), or flush the tagged TLB, where entries contain the process ID that produced each translation.

```
tagged TLB entry = \{
  process ID,
  virtual page number,
  physical page frame number,
  access permissions
\}
```

2. **Permission Reduction:** When the OS modifies an entry in a page table, the special purpose hardware keeps the cached data consistent. Software is needed so a single modification can affect multiple TLB entries and the OS ensures the TLB does not contain an incorrect mapping.

When adding permissions to virtual address space, the TLB can be left alone.
When reducing permissions to a page, the kernel checks if the TLB has a copy of the old page. If it does, the kernel must update permissions.

3. **TLB Shootdown:** Multiprocessors might have a cached copy of a translation in its TLB. Thus, whenever a page table entry is modified, the corresponding entry in every processor's TLB must be discarded. This requires interrupting the processor. (very expensive!)

## Virtually Addressed Caches

- Stores a copy of the contents of physical memory, indexed by the virtual address.
- On modern processors, these are split up located near the processor core, one for instruction lookups and one for data.
- Process context switch, permission reduction, and shootdown are all issues with virtually addressed caches.
- **Memory address alias:** Different processes have different virtual addresses that point to the same memory location. Each process has its own TLB entry for that memory, and the virtual cache stores a copy of the memory for each process. However, when a process modifies its own copy, how does the system know to update the other copy?

The solution is to store the physical address with the virtual address in the virtual cache.

## Physically Addressed Caches

- Consulted as a second level cache after the virtually addressed cache and the TLB but before main memory.
- Once the physical address of the memory address is located, we look at the Physically Address cache to see if there is a match. Thus we can avoid going to main memory

## Software Protection

- Naive software only protection: A machine code interpreter implemented in software simulates each instruction and only runs it if the addresses in the page table are permitted. (this is extremely slow!)

Goals of software protection:

1. **Simplify Hardware:** Is it possible to have eliminate hardware address translation to limit hardware complexity and runtime overhead from computers.
2. **Application level protection:** A browser wants to protect itself and the OS from malicious code in applications it runs.
3. **Protection inside the kernel:** Running third party code inside the kernel is dangerous, we need to protect the code in software.
4. **Portable Security:** Protection as a part of the runtime system allows us to download and run different applications without feat that the application will corrupt the OS.

- Thus, we want a _sandbox_ to execute untrusted code so that it can work without harming the rest of the system.

## Single Language OS

- Restrict all applications to be written in a single, carefully designed programming language(like solidity -> EVM?)
- For example, Javascript in modern web browsers is sandboxed. If a JavaScript program attempts to call a procedure that does not exist or reference arbitrary memory locations, the interpreter causes a runtime exception and stop the program.
- **Cons:** Using a single language based software protection relies on trusting the interpreter. Since interpreted code is slow, many interpreted systems put their functionality into system libraries that are compiled into machine code and run directly on the processor. A JavaScript program could cause a library routine to overwrite the end of a buffer for a buffer overflow attack.

Thus, many systems use both software and hardware protection.

## Language Independent Software Fault Isolation

- **Motivation:** Single Language OS requires trusting every compiler for every possible language. Is there a way we can isolate application code in a language independent fashion without using hardware?
- **Implementation:** Use a base + bound method. First, insert machine instructions into executable and check that each address is in base + bound. Second, use control + data flow analysis to remove checks that are not strictly necessary for the sandbox to be correct.

### Checking that each address is in base + bound

- For store + load instructions, just check to make sure that the addresses are within the correct region of data.
- For indirect branch instructions, make sure the program cannot branch outside of the sandbox except for predefined entry + exit points.

### Control + Data flow analysis

This allows us to eliminate extra inserted instructions if they are not needed. Possible optimizations include:

1. **Loop invariants:** If a loop strides through memory, we can use math to ensure all memory accesses in loop will be in the protected region.
2. **Return Values:** If static analysis can prove a procedure does not modify the return program counter on the stack, the return can be made safely.
3. **Cross procedure checks:** If code analysis can prove a parameter is always checked before it is passed as an argument to a subroutine, it does not need to be checked inside the subroutine.

### Sandboxes via intermediate code

Instead of creating a sandbox for x86 or ARM code, we can check a form of intermediate code and run that into the sandbox(kind of like a virtual machine)

For example, the JVM can be used to safely contain java code.

## Future

1. **Very large memory systems:** Ever increasing memories will require larger TLBs to hide the depth of the lookup trees.
2. **Multiprocessors:** TLB consistency will be more and more expensive as more processors will be added.
3. **User level sandboxes:** Each application has its own sandbox which is enforced in hardware.

# Caching

## Examples of Caches

1. **TLBs:** Caches recent results of address translation from page tables.
2. **Virtually addressed caches:** Stores memory value associated with a virtual address
3. **Physically addressed caches:** This is a part of the virtually addressed cache. Each entry stores the memory value associated with a physical memory location.
4. DNS caches, Web content caches, web search, email clients, incremental compilation, just in time translation, virtual memory, file systems, conditional branch prediction all use caches.

## Design Challenges of Caches

- **Locating the cached copy:** Check if the value you are looking for is in the cache
- **Replacement Policy:** Which elements should be replaced when the cache is full?
- **Coherence:** What should we do when the cache is out of date?

## Naive Cache Implementation

- An (address, value) pair stored in a map. When looking for something, we check if it exists in the map and return the value if it does(aka a cache hit). If it doesn't this is a cache miss.
- The cost of retrieving data out of the cache must be less than fetching the data from memory. The likelihood of a cache hit must be high enough to be worth the effort.
- Two sources of predictability are temporal locality and spatial locality. Programs tend to reference the same instructions and data they recently referenced(Temporal Locality). Programs also tend to reference data that is near recently referenced data.
- `Latency(read request)=Prob(cache hit) × Latency(cache hit)+ Prob(cache miss) × Latency(cache miss)`
- **Write through cache:** All updates are sent immediately onward to memory
- **Write back cache:** Updates are stored in the cache and only sent to memory when the cache runs out of space.

## Memory Hierarchy + When to use Caches

- Fundamental tradeoff: the smaller memory is, the faster it can be. The slower memory is, the cheaper it can be.

### Working Sets

- A sufficiently large cache will have a high cache hit rate, since if you can fit all the memory and data, the miss rate will be 0.
- Most programs have an inflection point where a critical mass of program data can just barely fit inside the cache. This is known as a _working set_
- **Thrashing:** Occurs when a cache is too small to fit inside the working set.

### Zipf Model

- **Motivation:** New data is constantly being added to the web, it is likely that your cache is outdated when you request something from it. There is no working set for the web, a small amount of pages might be useful(gmail, facebook, youtube) but there is a huge amount of web pages that are only visited from time to time.
- Zipf can be described as
  `Frequency of visits to the kth most popular page ∝ 1 / kα`
  and can model the popularity of library books, the population of cities, salary distribution, number of facebook friends, etc.
- **Heavy tailed distribution:** A significant number of references will be the most popular items, but a substantial portion will be less popular ones.

## Cache Design

- **Fully associative:** Addresses can be stored anywhere in the cache, there is no ordering. When looking for an address, we check every entry in the cache: there is a cache hit if any of the entries match.
  Cons: This is really slow as caches becomes bigger.
- **Direct mapped:** Each address is hashed then the data stored at the location of the hash.
  Cons: If two different addresses hash to the same entry, the system will thrash.
- **Set associative:** The cache hashes the address to get a location, then checks the entry in each table in parallel. It returns the value if any entries match the address.

## Replacement Policies

**Motivation:** If we look up an address and find that it is a miss, how do we choose which memory block to replace?

### Random

- **How it works:** Choose a random block to replace.
- **Pros:** Randomly evicting a block is fast, thus it is good for first-level hardware caches. Non-pessimal, randomness will never be the worst possible choice on average.
- **Cons:** Unpredictable, it might counteract an application that tries to take advantage of the cache.

### First in First Out(FIFO)

- **How it works:** Evict the cache block or page that has been in memory the longest.
- **Pros:** Each page spends an equal amount of time in the cache.
- **Cons:** Imagine a program that cycles through an array, where the array is too large to fit into the cache. Then the cache is useless.

### Optimal Cache Replacement (MIN)

- **How it works:** Replace whichever block that is used farthest in the future.
- **Pros:** This is optimal. (aka the best possible algorithm)
- **Cons:** Impossible, MIN requires knowledge of the future.

### Least Recently Used (LRU)

- **How it works:** Evict the block that has not been used for the longest period of time. In software, this is done with a linked list. In hardware, it is approximated with the clock algorithm
- **Pros:** Takes advantage of temporal locality
- **Cons:** Weak when the task is repeated scans through an array, the optimal strategy there is FIFO

### Least Frequently Used(LFU)

- **How it works:** Discard the page used least often
- **Pros:** Good in web caching. Ie just because we recently looked at spas in Iceland doesn't mean we will frequently look at spas in Iceland. Frequency is a better approximation than time.
- **Cons:** If an item is referenced repeatedly for a short period of time, this will stay in the cache long after it should have been evicted.

### Belady's Anomaly

Intuitively, adding space to a cache should always improve or maintain the cache hit rate. However, _Belady's Anomaly_ says this isn't the case.

For some replacement policies like FIFO, adding more space will drastically change the contents of the cache, thus making the hit rate worse.

## Memory Mapped Files

**Demand Paging:** Applications can access more memory than is physically present on the machine by using memory pages as a cache for disk blocks.
**Memory Mapped File:** The program directly accesses and modifies the contents of the file, treating memory as a write-back cache for disk.

### Pros

- **Transparency:** The program can write directly into the file as if it was memory
- **Zero copy I/O:** The OS does not need to copy file data from kernel buffers into user memory and back, it can just change the program's page table entry to point to the physical page frame containing that part of the file.
- **Pipelining:** The program can operate on data in the file as soon as page tables are set up, no need to wait for the entire file to be read into memory.
- **Interprocess communication:** Two processes can share memory via a memory mapped file.
- **Large Files:** The only size limit of a memory mapped file is the size of the virtual address space.

### Implementation

OS provides a system call where the kernel initializes a set of page table entries for that region of virtual address space, setting each entry to invalid.

When the process gives an instruction that touches an invalid mapped address:

1. The TLB miss occurs, triggering a full page table lookup in hardware.
2. The hardware finds the page table entry and finds that it is invalid, causing a _hardware page fault_ into the OS kernel.
3. The OS kernel converts the address to a file offset.
4. The OS kernel allocates an empty page frame and reads the required file block into the empty page frame.
5. The disk interrupts the processor when the file block has been read and the scheduler resumes handling the page fault exception.
6. The kernel updates the page table entry to point to the page frame allocated for the block and sets the entry to valid.
7. The OS resumes execution of the process.
8. The TLB misses, trigger a full page table lookup.
9. The hardware returns the page frame to the processor, loads the TLB with the new translation, evicts the previous TLB entry, then uses the translation to construct a physical address.

To create the empty page frame that holds the incoming page from disk, the OS:

1. Selects a page to evict using the current replacement policy.
2. Finds page table entries that point to the evicted page, usually through a core map.
3. Set each page table entry to invalid, to prevent other processes from using the evicted page while it is being evicted.
4. Copy back any changes to the evicted page.

## Approximating LRU

Keeping a linked list to track when pages were last used in hardware is prohibitively expensive, thus, we will use the clock algorithm as an approximation instead.

### Clock algorithm

The OS scans through the core map of physical memory pages. 2. For each page frame, it records the value of the use bit and clears the use bit. 3. The OS does a shootdown for each page frame that has its use bit cleared.

A common usage of the collected values of the use bit is the **not recently used/kth chance algorithm** where the kernel picks a page that has not been set for the last k sweeps of the clock algorithm.

## Virtual Memory

By backing every memory segment with a file on disk, we create virtual memory.

### Self Paging

**Problem:** There is a malicious program that tries to grab as much memory as possible by using multiple threads to cycle through memory and add those pages.
**Solution:** Self paging assigns every process its fair share of page frames. As the system starts to page, it evicts the page from whichever process has the most pages allocated to it.
**Cons:** If there are two processes, one with a working set of 2/3 of memory and the other with 1/3 of memory, the ideal split is for the first process to get 2/3 of memory and the other to get 1/3 of memory. Unfortunately, self-paging will results in both process splitting the memory, causing the first process to thrash

### Swapping

**Problem:** Sometimes, there is simply too much data in all of the process' working sets to effectively run them at once. If we try to run things normally, we will end up thrashing with every process.
**Solution:** We can evict an entire process from memory (aka swapping)
**Cons:** The process that just got evicted from memory will be sad :(

## Future

1. **Low latency backing store:** Things that were once on hard disk are moving onto SSDs, things that are on SSDs will move onto even faster storage devices.
2. **Variable page sizes:** We will move toward smaller effective page sizes due to low latency solid state storage and cluster memory.
3. **Memory aware applications:** Applications can be tuned to how much memory is in the first level cache to increase performance.

## Questions

1. How does set associative caches work?
2. How does virtual memory work?

# Distributed Systems

## LAN

- All hosts in a LAN can share the same physical communication media
- Each frame is delivered to every host
- If the host is not the intended recipient, it drops the frame
- Hosts in the same LAN can also be connected by switches

## Media Access Control Protocols(MAC)

- Problem: How do hosts acess broadcast media? How do they avoid collisions?

1. Channel partitioning protocols
   - Allocate I/N bandwidth to every host
   - Share channel efficiently and fairly at high load
   - Inefficient at low load where load = # of senders
   - Used for optical networks
2. Taking turns protocols
   - Pass a token around active hosts, a host can only send data if it has the token
   - More efficient at low loads since single node can use more than I/N bandwidth
   - Drawbacks: Overhead to acquire the token, vulnerable to failures(lost token or failed node)
3. Random Access
   - Efficient at low load since single node can fully utilize channel.
   - At high load, there is a collision overhead
   - Imagine you are in a crowded room:
   - Carrier sense(CS): Listen before speaking and don't interrupt
   - Collision detection(CD): if someone else starts talking at the same time, stop
   - Randomness: don't start talking right away

## Network Layer(3) - Connectivity

- Deliver packets to specified IP addresses across multiple datalink layers
- Wide Area Network: network that covers a vast area, ie city or state
- Router: Forward each packet received on an incoming link to an outgoing link based on packet destination IP address. _Forwarding Table_: mapping between IP address and output link

### Why should we use IP addresses instead of MAC addresses which are also unique?

- Doesn't scale. MAC addresses are like social security numbers, IP addresses are like unreadable home addresses
- MAC addresses are uniquely associated to the device for the entire lifetime of the device
- IP addresses change as the device location changes
- Since IP addresses usually specify where you are, we can keep a smaller networking table while we would need a huge one for MAC addresses.

- Internet Protocol(IP) is a best effort packet delivery
- Port Number: 16 bit number identifying the end point of a transport connection

## Drawbacks on layering

- Layer N may duplicate layer N-1 functionality
- Layers may need the same information
- Layering can hurt performance
- Layers are not cleanly separated
- Headers start to get really big(ie bigger than content)

# File Systems

## What does a storage system need?

1. **Reliability:** User data should be safely stored even if a machine's power is turned off or the OS crashes.
2. **Large Capacity and Low Cost** It takes 350 MB for an hour of music, 1 GB for 300 photos, and 4GB to store an hour long video. Many individuals own 1 TB or more of storage for personal files.
3. **High performance:** Programs must have quick access to data to satiate users(ie computer starting up, youtube serving video, amazon processing orders)
4. **Named data:** Data must be organized for future retrieval, the easiest way is to name the file.
5. **Controlled Sharing:** Users want to share stored data, but only with the right people.

## How does a file system help?

Non-volatile storage is much slower than DRAM, also, access must be done in coarse grained units, 512 bytes or more at a time.

- Goal: High performance
  - Physical Characteristic: Large cost to initiate I/O access
  - Organize data placement with files, directories, the free space bitmap(free map), and placement heuristics so that storage is accessed in large sequential units
  - Use Caching to avoid accessing persistent storage
- Goal: Named data
  - Physical Characteristic: Storage has large capacity, survive crashes, and is shared across programs
  - Support files and directories with meaningful names
- Goal: Controlled Sharing
  - Physical Characteristic: Device stores many users' data
  - Include access control metadata with files
- Goal: Reliable Storage
  - Physical Characteristic: Crashes can occur during updates. Storage devices can fail. Flash memory cells can wear out
  - Use transactions to make a set of updates atomic
  - Use redundancy to detect and correct failures
  - Move data to different storage locations to evenly wear out the disk

## Why is it important to know how file systems work even if I'm not building a file system?

- **Performance:** Even though file systems allow existing bytes in a file to be overwritten, inserting new bytes may require rewriting the entire file. Thus, autosaving may take as much as a second.
- **Corrupt Files** When overwriting the existing file with updated data, an untimely crash can leave the file in an inconsistent state, containing a combination of old and new versions.
- **Lost files** If instead of overwriting the document file, the applications writes to a new file, then deletes the original file, then moves the new file to the original file location, an untimely crash can leave the system with no copies of the document at all.

## The File System Abstraction

An OS abstraction that provides persistent named data.

- **File:** named collection of data in a file system. It is an arbitrary size, consisting of metadata and data. From the POV of the file system, a file's data is just an array of untyped bytes.
- **Directory:** Provides names for files. Contains a list of human readable names and a mapping from each name to a specific file or directory
- **Hard link:** Mapping between a name and the underlying file
- **Soft link:** Mapping from a file name to another file name. Unfortunately, this can cause dangling links: ie you link /a to point to /b which points to hi.txt then you unlink /b, /a will dangle)
- **Volume:** A collection of physical storage resources that form a logical storage device. This is usually an abstraction that corresponds to a logical disk.
- **Mounting:** Create a mapping from some path in the existing file system to the root directory of the mounted volume's file path. Used when plugging in USB drive to your computer

## The File System API

## Creating and Deleting Files

- `Create()` creates a new file that has initial metadata but no other data and it creates a name for the file in a directory
- `Link()` creates a hard link(new path name for existing file). After calling link(), there should be multiple path names that refer to the same underlying file. You cannot call link() on a directory
- `Unlink()` removes a file from its directory. If there are multiple links to a file, unlink() removes the specified name. If there is only one link to a file, unlink() also deletes the underlying file and frees the resources.
- `mkdir()` and `rmdir()` creates and deletes directories.

## Open and close

- `open()` a process calls open() to get a file descriptor it can use to refer to the open file.
- `close()` releases the open file record in the OS

## File access

- `read()` starts from the process's current file position and advances it by the number of bytes successfully read or written. Reads bytes
- `write()`starts from the process's current file position and advances it by the number of bytes successfully read or written. Writes bytes
- `seek()` changes a process' current position for a specified open file
- `mmap()` establish a mapping between a region of the process's virtual memory and some region of the file so that memory loads and stores to that virtual memory region will use the kernel's file cache or triggers a Page fault exception which causes the kernel to fetch the desire page from the file system to memory
- `munmap()` remove mapping created by mmap()
- `fsync()` Ensures all pending updates for a file are written to persistent storage before the call returns. Used to ensure that updates are durable and will not be lost in case of crash. If fsync() is called twice, the first is written to persistent storage before the second.

## Software Layers

## API and Performance

### System calls and libraries

Application libraries wrap syscalls mentioned above in the file system API to add additional functionality such as buffering.

### Block Cache

Since storage devices are much slower than DRAM, the OS has a block cache that caches recently read blocks and buffers recently written blocks.

### Prefetching

When a process reads the first two blocks of a file, the OS may prefetch the first 10 blocks. When the predictions are accurate, it can reduce latency from future reads(by returning the contents in the cache), reduce device overhead(by replacing a large number of small requests with one large one), and improve parallelism(by allowing hardware to process multiple requests at once in parallel).

Some disadvantages of prefetching include cache pressure(each prefetched block might displace another block in the block cache which might be more useful), I/O contention(prefetching costs IO, other requests might need to wait behind prefetch requests), and wasted effort(the prefetched blocks are not actually used).

## Device Drivers

- Used to translate the high level abstractions implemented by the OS and IO devices.
- Layering helps simplify OS by providing a common way to access various classes of devices.

### Memory Mapped IO

- IO devices typically have a controller with a set of registers that can be written and read to transmit commands and data to and from the device.
- Memory Mapped IO maps each device's control registers to a range of physical addresses on the memory bus. Reads and writes by the CPU to this physical address range go to the IO device's controllers instead of main memory

### DMA

- Most IO devices transfer data in bulk. Rather than requiring the CPU to read or write each word of a large transfer, IO devices can use direct memory accesses(DMA).
- DMA allows the IO device to copy a block of data between its own internal memory and the system's main memory.
- The OS uses memory mapped IO to set up a DMA transfer, then the device copies data to or from the target address without additional processor involvement
- After setting up a DMA transfer, the OS must not use the target physical pages for any other purpose until the DMA transfer is done.

## Lifecycle of a Disk Request

- When a process issues a system call like read() to read data from disk into the process’s memory, the operating system moves the calling thread to a wait queue.
- Then, the operating system uses memory-mapped I/O both to tell the disk to read the requested data and to set upDMA so that the disk can place that data in the kernel’s memory. - The disk then reads the data and DMAs it into main memory; once that is done, the disk triggers an interrupt.
- The operating system’s interrupt handler then copies the data from the kernel’s buffer into the process’s address space.
- Finally, the operating system moves the thread the ready list.
- When the thread next runs, it will returns from the system call with the data now present in the specified buffer.

## Files and Directories

Motivation: How do we go from a file name and offset to a block number?
Naive Approach: The file system is a dictionary that maps keys(file name and offset) to values(block number)

## Challenges with building a file system

1. **Performance** File systems need good spatial locality, where blocks that are accessed together are stored sequentially. The naive approach would just map block numbers in random places.
2. **Flexibility** File systems allow apps to share data, and thus must let people access small files, large files, short-lived files, and anything in between.
3. **Persistence** File systems must maintain and update both user data and internal data structures through OS crashes and power failures
4. **Reliability** File systems must store data for long periods of time

## File System Implementation Overview

Most file systems have directories, index structures, free space maps, and locality heuristics.

- **Directories** A way to map human readable file names to file numbers
- **Index structures** A way to translate the file number to locate the blocks of the file. This is usually some form of tree.
- **Free space maps** Tracks which storage blocks are free and which are in use as files grow and shrink. Also, blocks returned must have good spatial locality. Usually implemented as bitmaps
- **Locality Heuristics** Policies that group data to optimize performance. Some file systems group by directory, others periodically defragment their storage by rewriting existing files so that each file is stored in sequential blocks.

## Directories

- **Purpose** Translate human readable names to internal file numbers in a mapping
- **Implementation** Stored as a file
- The root directory's file number is agreed upon ahead of time, for the Unix Fast File System it is 2.
- To read `/home/albert/hi.txt` we search the root directory by reading the file associated with file number 2. In file2 , we search for the name `home` and find that /home is stored in file number 880. In file 880, we search for the name `albert` and find /home/albert is stored in 526. In file 526, we search for `hi.txt`, we find /home/albert/hi.txt is in file number 469
- The directory API is different from the standard open/read/close/write, since the file name to file number mapping cannot be corrupted
- The syscalls that modify directories are `mkdir`, `link`, `unlink`, and `rmdir`. Mkdir and rmdir create and delete the directory, `link("hi.txt", "new.txt")` creates a hard link to an existing file, `unlink("hi.txt")` removes that hard link.
- Processes can read the contents of directory files using the standard file read syscall

### Hard vs Soft Links

- **Hard Links** Multiple file directory entries that map different path names to the same file number. File systems must ensure that a file is only deleted when the last hard link is removed.
- File systems use reference counts that count the number of hard links to each file, starting with 1 at creation. Each call to `link()` increases the reference count by 1 and each call to `unlink()` decreases the reference count by 1
- **Soft Links** are directory entries that map one name to another name
- If the file system supports hard links, storing file metadata in directory entries would be problematic since whenever the file size changes, all the file's directory entries need to be updated

## Files

Purpose: Translate a file number to the blocks that belong to the file
Goals:

1. Put data blocks sequentially to maximize spatial locality
2. Provide efficient random access to any file block
3. Limit overhead to be efficient for smaller files
4. Be scalable to support large files
5. Provide a place to store metadata, ie reference count, owner, access control list, size

| File System        | Index Structure              | Free Space Management     | Locality Heuristics          | Index Structure Granularity |
| ------------------ | ---------------------------- | ------------------------- | ---------------------------- | --------------------------- |
| FAT(USB drives)    | Linked List                  | FAT array                 | Defragmentation              | Block                       |
| FFS(Unix)          | Tree(fixed, assymmetric)     | Bitmap(fixed)             | Block groups, reserve space  | Block                       |
| NTFS(Windows)      | tree(dynamic)                | Bitmap in file            | Best fit, defragmentation    | Extent                      |
| ZFS(Copy on write) | tree(copy on write, dynamic) | Space map(log-structured) | Write anywhere, block groups | Block                       |

### FAT Filesystem

- **Techniques** Uses a extremely simple index structure(linked list). FAT stands for file allocation table, its array of 32 bit entries in a reserved area of the volume. Each file corresponds to a FAT entry in the array, which is either the last block or contains a pointer to the next block.
- **Directories** map file names to file numbers. The file's number is the index of the file's first entry in the FAT. We can then parse the linked list to find the rest of the file's blocks.
- **Free space tracking** If data block i is free, then FAT[i] = 0
- **Locality Heuristics** Some FAT implementations use a next fit algorithm that sequentially scans the file allocation table starting from the last entry and returns the next free entry. This fragments the file, so there is often a defragmentation tool that reads files from their existing locations and rewrites them for better spatial locality.
- **Limitations**: Poor locality(badly fragmented files), Poor random access(need to sequentially traverse the FAT entries), limited metadata and no access control, no support for hard links, limitations on volume and file size(max file size if 4GB), and a lack of support for reliability techniques(does not support transactional updates).

### Unix Fast File System(FFS)

- **Techniques** FFS uses a carefully structured tree that locates any block of a file that is efficient for both large and small files. Each file is a tree with fixed size blocks as its leaves. Each file's tree is rooted in an _inode_ that contains the file metadata.
  - Typically the file's inode contains 15 pointers: the first 12 pointers are direct pointers that point directly to the first 12 data blocks of a file
  - The 13th pointer in the _inode_ is an indirect pointer which points to an array of direct pointers.
  - The 14th pointer in the \*inode is a double indirect pointer which points to an internal node which points to an array of indirect pointers, each pointing to an array of direct pointers.
  - The 15th pointer in the inode is a triple indirect pointer that contains an array of double indirect pointers.
  - All of the file system's inodes are located in an inode array. The file's file number, aka _inumber_ is an index into the inode array.
  - This asymmetric setup allows for efficient support for both large and small files.
- **Free Space Management** FFS allocates a bitmap with one bit per storage block, the ith bit in the bitmap indicates whether the ith block is free or in use.
- **Locality Heuristics** FFS uses block group placement and reserved space
  - _block group placement_ divides the disk into block groups so the seek time between any blocks in a block group will be small
  - Each block group holds a portion of the metadata structures
  - FFS puts a directory and its files in the same block group
  - Within a block group, FFS writes to the first free block in the file's block group. This helps locality in the long term since fragmentation is reduced
- When the disk's space is almost full, the block group heuristic will perform poorly since new writes will be scattered randomly around the disk. Thus, FFS reserves a portion of the disk's space and presents a slightly reduced disk size to applications

### Windows New Technology File System(NTFS)

Instead of using fixed trees like FFS, NTFS uses extents and flexible trees

- **Extent** variable sized regions of files that are stored in a contiguous region on the storage device
- **Flexible tree and master file table** Each file in NTFS is represented by a variable depth tree. The extent pointers for a file with a small number of extents can be stored in a shallow tree, deeper trees are only needed if the file becomes badly fragmented.
- **Master File Table(MFT)** stores an array of 1KB MFT records, each of which stores a sequence of variable size attribute records.
- **Resident attribute** Stores its contents directly in the MFT record
- **Non-resident attribute** Stores extent pointers in its MFT record and stores its contents in those extents.
- **MFT record** Includes standard information(creation time, last modified, owner ID), file name, and attribute list
- **Four stages of file growth**
  1. Small files can have their contents included in the MFT record as a resident data attribute.
  2. Normal file data has a small number of extents tracked by a single non-resident data attribute.
  3. Large files combined with a fragmented file system can make a file have so many extents that the extent pointers will not fit in a single MFT record. These files can have multiple non-resident data attributes in multiple MFT records, with the attribute list in the first MFT record indicating which MFT record tracks which range of extents
  4. If the file is huge, the file attribute list can be made non-resident, allowing arbitrarily large numbers of MFT records
- **Free Space Map**
  NTFS stores all metadata in a dozen files with low file numbers. For example, file number 5 is the root, file number 6 is the free space bitmap, file number 8 contains a list of the volume's bad blocks.
- **Locality Heuristics** NFTS uses best fit, where the system tries to place a newly allocated file in the smallest free region large enough to hold it. NTFS also has a defragmentation utility

### Copy on Write (COW)

When updating an existing file, COW file systems do not overwrite existing data or metadata, instead, they write new versions to new locations.

- **Motivations**
  - Small writes are expensive. Large sequential writes is much better than small random writes, and the gap will continue to grow since bandwidth grows faster than seek time/rotational latency
  - Small writes are expensive on raid since we need to read old data, read old parity, write new data, and write new parity. The number of times we read/write to parity bits grows linearly with the number of reads/write but not the size of the reads/writes
  - Large DRAM caches can handle essentially all file reads, thus the cost of writes dominate performance so optimizing write performance is the number one priority.
  - Flash storage works better with copy on write: there is no need to clear the erasure block and it writes data to a different location instead of overwriting the current data wearing out the flash drive evenly
  - Since old data isn't overwritten, we can support versioning
- **Implementation:** We store inodes in a file rather than in the inode array. All the file system's contents are stored in a tree rooted in the root inode, when we update a block, we write the block and all of the blocks on the path from it to the root to new locations.

### ZFS file system

- **uberblock** the root of the ZFS storage system, ZFS has an array of 256 uberblocks and rotates successive versions
- **dnode** similar to inode, a file represented by a variable depth tree whose root is a dnode and leaves are data blocks
- **Free space map** per block group space maps. ZFS maintains a space map for each block group, has a tree of extents, and log structured updates.
- **Locality Heuristics** On writes, ZFS ensures sequential writes and batched updates.

## File and Directory Access Walkthrough

Goal: Read the file /foo/bar/baz
Steps:

1. Read the root directory `/` to determine `/foo`'s inumber
2. Open and read file 2's inode. Since FFS stores pieces of the inode array at fixed locations on disk, given a file's inumber it is easy to read the file's inode
3. From the root directory's inode, extract the direct and indirect block pointers to determine which block stores the contents of the root directory.(block 48912)
4. Read that block of data to get the list of name to inumber mappings in the root directory to find `/foo` has inumber 231
5. Since we know `/foo`'s inumber, we read inode 231 to find where `/foo`'s data blocks are stored-block 1094 in this example.
6. Read block 1094 to get the list of name to inumber mappings in the `/foo` directory to find that the directory file `/foo/bar` has inumber 731.
7. Follow similar steps to read `/foo/bar`'s inode and data block 30991 to find `/foo/bar/baz`'s inumber is 402
8. Read `/foo/bar/inode` to get the data blocks 89310, 14919, 23301
9. Usually, data blocks are cached so we don't need to repeat this process.

## Questions

1. What does mmap() do?

# Input/Output

# Motivation

Without I/O, computers are useless. However, there are thousands of devices, each one slightly different. How do we standardize the interfaces? Devices can be unreliable(media failures/transmission errors/ packet loss), how do we make them predictable, fast, and reliable?

## Bus

- Common set of wires for communication among hardware devices + associated protocols to transfer data
- Allows for read, write
- Connects n devices over a single set of wires, connections, protocols O(n^2) relationships with 1 set of wires
- Cons: Only one transaction at a time

## How does the Processor Talk to the Device?

- The CPU contains a set of registers that can be read and written
- **Port Mapped I/O:** in/out instructions
- **Memory Mapped I/O:** load/store instructions

### Optimal Parameters for I/O

- **Data granularity:** Some devices provide single bytes at a time(keyboards), while other provide whole blocks (disks, networks)
- **Access Pattern:** Some devices must be accessed sequentially(tape), others can be accessed randomly (disk, cd). Some devices require continual monitoring while other generate interrupts when they need service

## Transferring Data To/From Controller

- **Programmed I/O:** Each byte transferred via processor in/out or load/store
  - Pros: Simple hardware and software
  - Cons: Consumes processor cycles proportional to data size
- **Direct Memory Access:** Give controller access to memory bus and asks it to transfer data blocks to/from memory directly
- **Blocking Interface:** When system reads data, sleep the process until the data is ready. When the system writes data, sleep the process until device is ready for data.
- **Non-blocking Interface:** Return quickly from read/write with count of bytes transferred. It may return nothing and write may write nothing.
- **Async Interface:** When reading data, take pointer to user buffer, return immediately since the kernel will fill the user buffer and notify user. When writing data, get the pointer to user buffer, return immediately. The kernel will read the data and notify the user.

## Memory Management

## Zero Copy I/O

**Problem:** Operating systems commonly stream data between user level programs and physical devices. However, it takes a ton of time to do so since we move data from hardware to kernel then userspace or vice versa.

**Potential Solutions that don't work:** We could move the application into the kernel directly so the data doesn't need to be copied from kernel to userspace. (This is not secure) We could also modify syscalls to allow applications to modify the data stored in the kernel buffer. (This isn't flexible, it doesn't allows for modifications)

**Solution 1: Use the process' page table** The application aligns the user level buffer to a page boundary which is then provided to the kernel. Thus, a page-to-age copy from user to kernel space can be simulated by changing page table pointers instead of physically copying memory.

## Virtual Machines

- A way for a host OS to run a guest OS as an application process.
- They are widely used on client machines to run non-native applications, are used in data centers to allow a machine to be shared by multiple machines

## Virtual Machine Page Tables

- There are two sets of page tables, guest and host
- The host OS provides a set of page tables to constrain the execution of the guest OS which is thinking it is running on a real OS but it is actually running on virtual memory.
- The host translates the guest virtual addresses into physical addresses
- The guest OS manages page tables for the guest processes, exactly the same as if it was on real hardware. This is because the guest OS thinks it is on real hardware.

When the host OS transfers control to the guest OS, everything works as expected.

When the guest OS transfer control to the guest process, this is a privileged instruction and the hardware processor will first trap back to the host. However, we can't use the guest OS page table, since it is using virtual addresses instead of physical addresses. Nor could we use the host OS page table since the guest OS should not have permission to modify that content. Thus, we will need to use **Shadow Page Tables**.

### Shadow Page Table

A composite page table that represents both the guest and host page tables.

- To keep the shadow page table up to date, the host needs to check if any shadow page tables need to be updated before it changes a page table entry.
- To keep the shadow page table up to date, the host also sets the memory of the guest page tables as read only. Thus, the guest OS traps the host every time it makes a change to a page table entry. When trapping, the host updates both the guest page table and the corresponding shadow page table.

### Transparent Memory Compression

- **Problem:** Multiplexing multiplexors is hard. For example, a data center usually has a single physical machine running many virtual machines at the same time(ie a computer hosting multiple web sites)
- When a guest OS is running two copies of the same program or process, the host kernel has no way to share pages between the two instances.
- **Solution:** Commercial Operating systems run a scavenger in the background that looks for common pages that can be shared among multiple virtual machines. Once identified, the host kernel provides an illusion that each guest has its own copy.

- **Case 1: Multiple copies of the same page**
  Multiple virtual machines might share a zeroed page, it would be wasteful to create a new zeroed page for each virtual machine that needs one. The host OS instead allocates a _read-only_ zero page, which traps the host kernel when modified and allocates a new zeroed physical page for the guest OS.

They scavenger runs in the background looking for zeroed pages

- **Case 2: Compression of unused pages**
  Pages can differ by small amounts, this lets the host kernel add a layer in the memory hierarchy to save space. Instead of evicting a relatively unused page, we can save its delta with regards to the original page, and the full copy the delta is from stays read only.

If the original page is modified, the delta can be quickly recompressed or evicted as necessary.

## Fault Tolerance

All systems break, but how can we manage memory to recover application data structures after a failure, not just user file data?

## Checkpoint and Restart

- **Naive Solution:** Periodically save the program's internal data to a file.
- **General Solution:** The OS uses the virtual memory system to provide application recovery as a service
  **Implementation:**

1. Suspend each thread executing in the process and save its state. (Suspending threads is to avoid race conditions)
2. Store a copy of the contents of the application memory on disk (this is the checkpoint)
3. After a failure, resume execution by restoring the contents of memory from the checkpoint and resume each of the threads.

Use address translation to minimize the amount of time the system is stalled during a checkpoint. We can mark the application pages as copy on write to do so.

- **Process Migration:** The ability to take a running program on one system, stop the execution, then resume it on a different machine.

## Recoverable Virtual Memory

Unfortunately, checkpoint + restart is an expensive operation that we should use sparingly. Is it possible to provide the application the illusion of persistent memory, so that the contents are restored to a point not long before failure?

**Implementation:**

1. Take a checkpoint so that some consistent version of the application data is on disk
2. Record a log of every update that the application makes to memory.
3. Once things crash, read the checkpoint and apply the correct changes from the log.

**Pros:** This is how text editors save their backups.
**Cons:** Information that is not on the current version of the file can be leaked!

**Incremental Checkpoint:** Created by stopping the program and saving a copy of any pages that have been modified since the previous checkpoint.

How much work is lost during a crash is a function of how quickly we can write an incremental checkpoint to disk.

## Deterministic Debugging

- Debugging a concurrent program is hard, we don't control the scheduler, thus depending on how threads are scheduled program runs may vary.
- Debugging an OS is even harder, the OS uses widespread concurrency but also makes it difficult to tell input from output.
- **Implementation:** Use a virtual machine abstraction to allow for debugging. The host kernel mediates the initial state, the input data from I/O devices, and precise timing of interrupts.
- For multicore systems, we also need to track the order in which threads go through critical sections in addition to the initial state, the input data from I/O devices, and precise timing of interrupts

## Security

The operating system needs to protect itself and other applications from malicious or buggy implementations.

**Virtual Machine Honeypot:** Run the suspect application in a virtual machine constructed for the purpose of running suspect code. We can delete the virtual machine if the code is suspect.

## User Level Memory Management

Operating Systems now provide users the ability to:

1. Decide where to get missing pages. (local disk, non-volatile memory, remote memory, etc.)
2. Which pages can be accessed
3. Which pages should be evicted.

## Storage Devices

Persistent Storage devices like SSDs or Magnetic Disks can hold much more memory but are much slower than DRAM.

## Magnetic Disk

**Description:** Each disk drive holds one or more platters, which are the CD looking things that hold up the magnetic material. When the disk drive powers up, the platters are constantly spinning on the pole. The disk head reads/writes data by sensing or introducing a magnetic field on a surface. Since the platter is spinning very fast, it generates a layer of rapidly spinning air, which lets the disk head float close to the magnetic material without touching it. If the disk head does touch the material, this is known as a _disk crash_. This can happen when you drop a running magnetic disk.

The data itself is stored in 512 byte sectors, the disk drive must read and write the entire sector even if it only wants to change a single byte in the sector.

- **Track Buffering:** Improves performance by storing sectors that have been read by disk head but not yet requested by the OS.
- **Write acceleration:** Stores data to be written to disk in the disk's buffer memory and acknowledges the writes to the OS before the data is written to the platter. The data is actually written later. Of course, if power is lost before the disk's buffer memory has been stored, then data might be lost.
- **Tagged command queueing:** Issue multiple concurrent requests to disk and the disk processes the requests out of order in an optimal manner.

### Access and Performance

To go from a request to data, the disk must first seek the right track, wait for the desired sector to rotate to the head, then transfer the blocks. The equation for this is
`disk access time = seek time + rotation time + transfer time`

- **Seek Time:** Move the arm over the desired track(each track is kind of like a different lane on a track in track and field). The **minimum seek time** is the time it takes for the head to move from one track to an adjacent one. The **maximum seek time** is the time it takes for a disk to move from the innermost track to the outermost track. The **average seek time** is the average seek time across all possible pairs of tracks on a disk, this is usually approximated to the time it takes to seek a third of the way across a disk

- **Rotation Time:** After the disk head and found the correct track, it must wait for the target sector to rotate under it. The waiting time is known as the **rotational latency**. However, most disks start reading the entire track's sectors into their buffer memory.

- **Transfer time:** After the disk head is on the target sector, the disk must transfer the data from the target sector to the buffer memory(for reads) or from the buffer memory to the target sector(for writes). Then, it transfers the buffer memory to main memory for reads or vice versa for writes. The **surface transfer time** is the time to transfer one or more sequential sectors from a surface once the disk head begins reading. Thus, the **surface transfer time** is usually faster for outer tracks than inner tracks since outer tracks have a higher linear velocity.

## Flash Storage

Flash storage is now used for laptops, machine room servers, phones, cameras, thumb drives, and everything in between. Flash storage is a type of **solid state storage**, it has no moving parts and stores data using electrical circuits. It has much better random I/O, use less power, and is less vulnerable to physical damage.

Each flash storage element is a floating gate transistor. Since the floating gate is surrounded by an insulator, it can hold an electrical charge for months or years without requiring power. Even though it is not connected to anything, it can be charged or discharged via electron tunneling by running a sufficiently high-voltage current near it. Thus, the floating gate state can be detected by applying an intermediate voltage to the transistor control gate that will only be sufficient to activate the transistor if the floating gate is changed.

### Access and Performance

There are three operations we can perform on flash storage.

1. **Erase erasure block:** Before flash memory can be written, it must be erased by setting each cell to a logical one. Flash memory can only be erased in large units known as erasure blocks. This is a slow operation, taking several milliseconds.
2. **Write Page:** Once erased, NAND flash memory can be written on a page-by-page basis, where each page is typically 2048-4096 bytes. This takes tens of microseconds.
3. **Read Page:** NAND flash memory can be read on a page by page basis. Reading usually takes tens of microseconds.

## Calculations

Without remapping  
`Time per write = (sizeof erasure block / page size) * (page read time + page write time) + block erase time`
With remapping
`Time per write = block erase time / (sizeof erasure block / page size) + page write time`
Remapping amortizes the cost of flashing an erasure block.

### Durability

Over time, the high current loads from flashing and writing memory causes the circuits to degrade. Eventually after a few thousand to few million erasures, a given cell will wear out and no longer reliably store a bit.

Thus, flash devices use:

- **Error correcting codes:** Each page has some extra bytes to protect against bit errors in the page
- **Bad page and bad erasure block management:** If a page or erasure block has a manufacturing detect or wears out, the firmware marks it as bad and stops storing data on it.
- **Wear leveling:** Rather than overwriting a page in flash, the flash translation layer remaps the logical page to a new physical page that has already been replaced. Thus, we wear out the flash device evenly
- **Spare pages and erasure blocks:** Manufacturers may include spare pages and spare erasure blocks to provide extra space for wear leveling and to allow for bad page and bad erasure block management without shrinking the capacity of the storage device.

## The future for memory devices

- **Spinning disk vs flash storage:** Textbook is outdated, flash storage is almost always better
- **Phase change memory:** Use a current to alter the state of glass between amorphous and crystalline forms which represents the data bit(0 or 1). Apparently it has better write performance and endurance than flash, though right now storage density isn't that great.
- **Memristor** A circuit element whose resistance depends on the amounts and directors of currents that have flowed through it in the past. The goal is for each core on a 32 core processor chip to have a few gigabytes of stacked memristor memory

## Multiprocessor Scheduling

## Motivation

Most computers are multiprocessors. How do we use multiple cores to run sequential tasks?
How do we adapt scheduling algorithms for parallel applications?

## Sequential Scheduling - Naive Approach

Create a single multi-level feedback queue(MFQ) with a lock so that only one processor can read/write from it at once. Each idle processor takes the next task off the MLFQ and runs it.

**Potential Problems**:

- **Contention for MFQ lock:** Many Processes are fighting over the MFQ lock
- **Cache Coherence Overhead:** Processes need to fetch the current state of the MFQ from the cache of previous processors to hold the lock. The cache for the current processor is likely not going to be of much use since most of it will need to be updated. Also, you are fetching/updating the cache data while you are holding the MFQ lock, causes further delays.
- **Limited Cache Reuse** Since threads run on the first available processor, that processor probably doesn't have the necessary data in its cache.

## Commercial Operating System Scheduler

Use a separate copy of the MFQ for each processor to maximize cache reuse.

**Affinity Scheduling:** Once a thread has been scheduled on a processor, return it to the same processor when it is re-scheduled to maximize cache reuse. Each processor has its own copy of the queue to run threads from, only rebalance the queue length differences are large enough to compensate for the cache reload for the threads we move to different queues. Unfortunately locks are still necessary since per processor data structures must be protected by locks.

**Definition-Preempting a thread/process:** A thread is pre-empted when a higher priority thread is placed on the ready queue and the higher priority thread runs instead of the current thread.

## Oblivious Scheduling

Each thread is time sliced to available processors with no attempt to ensure threads from the same process run at the same time.

**Definition-Critical Path:** The minimum sequence of steps for the application to compute the result.  
**Definition-Spin-then-wait:** If a lock is busy, the waiting thread spin-waits briefly for it to be released, and if it still busy, it blocks and looks for other work.  
Pros: Great if the lock is held for short periods of time.

**Potential Problems:**

- **Critical Path Delay-** Preempting a thread on the critical path will slow down the end result.
- **Bulk Synchronous Delay-** A common design pattern is to split work into equal sized chunks, do the work on the chunks, then communicate the work done to the next stage of computation(Think Google mapreduce). However, at each time step the work done is limited to the slowest processor, since all the chunks need to be finished before moving onto the next stage computation.
- **Producer-consumer Delay-** Another common design pattern is the producer-consumer. The results of one thread are fed to the next thread, of which the results are fed to the next thread, etc. Preempting a thread in the middle of a producer-consumer chain will stall all of the processors in the chain.
- **Preemption of lock holder-** In spin-then-wait strategies for locks, the lock holder can be preempted-other tasks will spin-then-wait until the lock holder is rescheduled.
- **I/O-** If a read/write happens in the kernel, the thread blocks. To use the processor when the thread is waiting, the application needs to have more threads and processors so the extra threads can run while one is waiting. However, if the thread does not block(ie a read when the file is in memory), then we cannot do time slicing since we have more threads than processes.

## Gang Scheduling

Approach: Schedule all of the tasks of a program together. The application groups some work into a number of threads, and the threads run together or not at all. If the OS needs to schedule a different application or there aren't enough idle resources, it preempts all of the processors of an application to make room

Implementation in Commercial Operating Systems: The application pins each thread to a specific processor and mark it to run with high priority. The system uses the rest of the processors to run other applications without interfering with the primary application.

**Problems**: As the number of processors increase, performance grows slower since some applications scale linearly with the number of processors, other achieve diminishing returns.

Thus, it is usually more efficient to run two parallel programs with half the number of processors vs time slicing the two programs and gang scheduling.

**Definition-Space Sharing:** Allocating different processors to different tasks. Efficient since it minimizes processor context switches. The problem occurs when the not all tasks start and stop at the same time, how do we know how many processors to use if the number changes over time?

## Scheduler Activations

Make the assignment and re-assignment of processors to applications visible to applications. Each application has an execution context(aka scheduler activation). Each application is explicitly informed when a processor is added to the allocation or taken away via an upcall.

This leaves the question of how many processors should each process be assigned? This is an open research problem, there is a fundamental tradeoff between average response time and fair allocation of resources among different applications. In multiprocessors, average response time can be improved by giving extra resources to parallel interactive tasks as long as it does not cause long-running compute intensive parallel tasks to starve for resources.

## Remaining Questions

1. What is the difference between MLFQ and MFQ? Are they the same thing just abbreviated differently?
2. Even if one of the waiting processors picks up the pre-empted task, a single preemption can delay the entire computation by a factor of two, and possibly even more with cache effects. Why is this the case with Bulk synchronous delay for oblivious scheduling?
3. Preempting a thread in the middle of a producer-consumer chain will stall all of the processors in the chain. Shouldn't this be stall all of the processors after this thread in the chain?
4. Performance as a function of the number of processors, for some typical parallel applications. Some applications scale linearly with the number of processors; others achieve diminishing returns. Why is this the case?

# Queueing Theory

Response time depends non-linearly on the rate that tasks arrive at a system. For all intents and purposes, we assume that the system is work-conserving(all tasks that arrive are eventually serviced). Secondly, we are using FIFO scheduling.

## Definitions

- **Server:** A server is anything that performs tasks. ie web server, processor in a client machine, cashier in supermarket
- **Queueing delay(W):** The total time a task must wait to be scheduled. aka wait time
- **Number of tasks queued(Q):** number of tasks on the queue
- **Service time(S):** The time to complete a task assuming no waiting. aka execution time.
- **Response time(R):** Queueing delay(W) + Service time(S)
- **Arrival rate(lambda):** Average rate at which new tasks arrive
- **Arrival process:** When tasks arrive and whether arrivals are bursty or spread evenly over time.
- **Service Rate(u):** Number of tasks the server can complete per unit time when there is work to do.
- **Utilization(U):** Fraction of time the server is busy. `U = lambda / u` if lambda < u, else `U = 1` if lambda >= u
- **Throughput(X):** Number of tasks processed by the system per unit of time. `X = U * u` We can rewrite this as `X = lambda if U < 1, else X = u if U=1`
- **Number of tasks in the system(N):** The number queued + number receiving service. `N = Q + U`

## Little's Law

Conditions: Applies to any stable system where the arrival rate matches the departure rate. It relates the average throughput, response time, and number of tasks in the system.
`N = X * R`

## Response Time vs Utilization

- Higher Utilization normally implies higher queueing delay and higher response times.
- Higher Utilization also increases the risk of overload(where requests grow faster than they can be processed, thus queues and wait times grow without bound)
- Can predict average response time from the arrival process and service time but it is difficult
- Goal: make tradeoffs between higher utilization and better response time. In the fast, higher utilization was favored since computers were expensive, now, it is better to have a better response time since computers are cheaper than humans.
- **Best case(Evenly spaced arrivals):** Suppose we have a set of fixed sized tasks that arrive equally spaced from one another. As long as the rate at which tasks arrive is less than the rate at which the server completes the tasks, there is no queueing since the server finishes the previous request just in time for the next arrival.
- Suppose arrivals come in every 1 ms and the server can process each task in 1ms. If at some point a single extra request arrives, every other request will need to wait 1ms for that request.
- **Worst case(Bursty arrivals):** Suppose a group of tasks arrive exactly at the same time. The average wait time increases linearly as more tasks arrive together.
- **Exponential arrivals:** A useful model for understanding queueing behavior is the exponential distribution. This can describe the time between tasks arriving and the time it takes to service each task.
  An exponential distribution of a continuous random variable with a mean of 1 / lambda has the probability density function `f(x) = lambda _ e ^ -(lambda _ x)
  The exponential distribution is _memoryless_, ie no matter how long we have already waited for the event, the likelihood of that event occurring remains the same.

## What if Questions

## What if we changed the scheduling policy from FIFO to something else?

- If the distribution of arrivals or service times is less bursty than an exponential(ie evenly spaced or Gaussian), FIFO will deliver nearly optimal response times. Round robin will be worse than FIFO
- If task service times are exponentially distributed but individual task times are unpredictable, the average response time is exactly the same for Round Robin as for FIFO.
- If task lengths can be predicted and there is variability in service times, Shortest Job first can improve average response times, particularly if arrivals are bursty
- Real world systems usually have more bursty arrival times than the exponential distribution, their distributions are known as _heavy tailed distributions_ since it has more long tasks. SJF is better than Round Robin which is better than FIFO for _heavy tailed distributions_
- SJF can improve average response times but increases response times for long tasks since long tasks will only complete immediately before an idle period. As utilization increases, the idle periods become increasingly rare.

## What if workloads vary with the queueing delay?

- Arrival rates and service times are not always independent of queueing delay. For example, a online store becomes overloaded and slow during the holiday shopping season. Rather than continuing to browse, some customers may get fed up and leave, reducing the arrival rate of requests for individual web pages.

## What if we had multiple servers?

- Banks have multiple tellers, grocery checkouts have multiple cashiers. Clearly, there are often efficiency gains from having separate queues.
- On the other hand, a single queue is always better than separate queues when focusing on average response time because of variations in how long each task takes to service. One queue could be idle while another queue has multiple tasks

## If a processor is 90% busy serving web requests and we add another processor to reduce the load, how much will that improve average response time?

- Unfortunately, there is not enough info to say since each web request needs processing time and disk I/O and network bandwidth.
- If arrival times and service times are exponentially distributed and independent of the system response time, the overall response time is the sum of the response times of the components `response time = sum for all arrival and service times(S / (1 - U))

## What are some conclusions we can draw from real world systems?

- Almost all real world systems exhibit some randomness in their arrival process or service times
- Response time increases with increased load
- System performance is predictable across a range of load factors if we can estimate the average service time per request.
- Burstiness increases average response time.

# Real Time Scheduling

## Motivation

The software that runs the brakes in your car or autopilot on an airplane must work when called for, you can't hit the brakes to stop after you have crashed your car. The change in value of a specific task over time is shown in the below graph.

## Three techniques to increase likelihood that threads run before their deadline

- **Over provisioning:** Make sure real-time tasks only use a fraction of the system's total processing power. This is like taking 2 units when you know you can handle 16 units.
- **Earliest Deadline First:** Choose the task that has the earliest deadline first. However, this won't always work with the given tasks due to it requiring both I/O and computation. Thus, always break tasks down into shorter units and use the deadlines of the shorter units.
- **Priority Donation:** The problem of _priority inversion_ occurs when a higher priority thread must wait for a lower priority thread to complete its work. This can be solved via Priority Donation, where a higher priority thread that is waiting for a lower priority thread donates its priority to the lower priority thread, allowing it to run and finish using that resource.

## Questions

1. For priority donation, do you actually change priority values by subtracting from the priority of the higher priority thread and adding to the priority of the lower priority thread or do you just set the lower priority thread's priority to the priority of the higher priority thread?

# Reliable Storage

# Why do we need reliable storage?

- Physical devices may fail, they may wear out, they may become damaged. Large organizations see annual disk failure rates of 2-4%, which means that over 10 years 30% of the data in the disk would be gone
- Two threats to reliability
  - **Operation Interruption:** A crash or power failure in the middle of a series of related updates leaves the data in an inconsistent state. Ie crashing when moving a file so we now have two copies
    - **Solution** Create transactions for atomic updates.
  - **Loss of stored data:** Failure of non-volatile storage media can cause stored data to become corrupted or disappear
    - **Solution** Create redundancy for failures, ie RAID or using multiple layers of checksums

## Reliability, Availability,

- **Reliable:** Can the system perform the intended function? A system can be available but not working properly. Can increase reliability by testing system.
- **Available:** Can the system respond to a request? Also known as the percentage of time when the system is up. Can achieve higher availability with deploying apps across multiple geographically distant servers using load balancing to reroute requests to healthy servers
- **Durability:** Is the data still in the system? A system is durable even if you can't access the data(ie bury a usb underground). Can achieve higher durability by making backups, storing in different geographical area, performing checksums.
- A data block replicated across 100 machines is highly reliable and highly available for reads, but not highly available for writes.

## Transactions + Atomic Updates

- **Problem:** When the system crashes in the middle of several updates, the data in the system might not make sense or leaves it in a vulnerable. For example, bob is transferring 100 dollars to alice and the bank database crashes, it could have crashed when the bank pulled 100 dollars out of bob's account but before it sent the 100 dollars to alice's account. Now, bob just lost 100 dollars
- **Naive Solutions:** The unix fast file system would control the order of updates such that if a crash occurred, it could scan the disk during recovery and identify and repair inconsistent data structures. Now if this sounds kind of sketchy, it is and there were three main problems with it.
  - **Extremely slow recovery:** When the machine reboots, it has to scan all of its disks for inconsistent metadata structures.
  - **Slow Updates** To ensure the unix fast file system updates were analyzable, they had to be ordered correctly, making it hard to parallelize the stream of requests to storage devices
  - **Complex reasoning** We need to consider all possible failure scenarios to make sure that it is always possible to recover the system to a consistent state.
- **Application Level Approaches** The POSIX api first writes updates to a temporary file, then renames the temporary file to atomically replace the previously stored file.

### The Transaction Abstraction

- **Problem** You are updating a website and you want to update files stored in `/prod` with files stored in `/testing`. However, you don't want users to see the intermediate step where some docs in `/prod` are old and some are new.
- **Solution** The transaction first does all of its changes, then commits the transaction. If at any time the transaction fails before the commit, we roll back everything done in the transaction. The benefit here is that transactions are ACID.(Yes this sounds like a dumb acronym they teach you in 6th grade to memorize the dinosaurs but this is actually an extremely well thought out acronym. ACID = transaction)
  - **Atomicity** All or nothing, either a transaction commits or the system is not changed.
  - **Consistency** The transaction moves from one legal state to another. All invariants must hold at the start of the transaction and during the commit.
  - **Isolation** The updates within a transaction cannot interleave with other transactions. Even if multiple transactions concurrently execute, the results will be that T is executed entirely before T` or vice versa
  - **Durability** Once a transaction is committed, the only way to change state is with another transaction. Must survive crashes!
  - Transactions are just critical sections with durability.

## Transaction Implementations

Problem: We want to group related writes in a single, atomic transaction but for disks and other hardware, the largest atomic operation is a single write(sector or page). Thus, we need a way to combine multiple writes into a single atomic operation.

**Idempotence:** The property where an operation can be applied multiple times without changing the result beyond the first application.

Idea: A transactional system can store all of the transaction's intentions, and the updates will only be made after all of the transaction's intentions are written down. If we store all updates as idempotent operations, we can also start off where we finished during a crash

### Redo Logging

- An implementation of transactions using four phases

  1. **Prepare** Append all planned updates to the log.
  2. **Commit** Append a commit to the log, indicating the transaction has been committed
  3. **Write-back** Write values in log to disk
  4. **Garbage Collect** Remove transaction records from log

- If the system crashes in the middle of the transaction, we scan sequentially through the log and depending on the type of record we do different things. If the record is an

  1. **Record Type = Update** add this record to a list of updates for the specified transaction
  2. **Record Type = Commit** Write back all of the transaction's logged updates to disk
  3. **Record Type = Roll-back** Discard list of updates for specified transaction
  4. **Record Type = End** Discard all update records for transactions without commit records.

- Implementation Details

  1. Multiple transactions might be executing at once, meaning that we need an identifier to show which transaction the log record belongs to
  2. The write-back phase can be asynchronous, meaning we can minimize latency in the commit phase and write-back multiple things at once. However, this makes crashes take longer and the log needs to take more space in the disk.
  3. Since log records are idempotent, we can reapply an update from a redo log multiple times.
  4. Restarting in recovery is ok, we just continue from where we last left off.
  5. Once write-back completes, the space in the log can be reclaimed
  6. All transaction updates must be on the log before the commit, all commits must be on disk before the write backs, all write backs must be on disk before the transaction log records can be garbage collected.

- Performance: redo logging can have performance better than update in place, especially for small writes. Four factors allow efficient redo logging:
  - Log updates are sequential, so writing to spinning disks can be very fsat.
  - Write back is asynchronous, so transactions using redo logs can have good response time and throughput
  - The only barriers are when updates are logged and before the commit is logged, and after the commit is logged and before write-backs begin
  - Group commits combine a set of transaction commits into one log write to amortize the cost of initiating the write

## Isolation and Concurrency

To fulfill ACID properties, we need to ensure all of its properties. Redo logging only ensures durability, we also need isolation if there are multiple transactions executing at once.

Naive Solution: Two phase locking divides a transaction into two phases, the expanding phase where locks can only be acquired and the contraction phase where locks can be released but not acquired.

- Two phase locking results in serializability, meaning the execution is equivalent to processing the transactions one at a time.
- If we encounter deadlocks, we can roll back a transaction and restart at some later time.

## Transactions in File Systems

If the file system crashes when updating data, it could be left in an inconsistent state. Thus, all file system updates should be done in transactions.

- **Logging:** Include all updates to disk in transactions
- **Journaling:** Apply metadata updates via transactions. Apply file updates normally. Ths is because file updates are idempotent
- **Copy-on-write:** Does not overwrite data in place, but rather writes data then replaces pointer.
  - Copy on write systems also usually have batch updates, where you batch multiple updates into one atomic group
  - This also has an intent log, a redo log that tracks updates between large batch updates

# Error Detection and Correction

Problem: All data storage devices will fail
Solution: Use layered error detection and correction: checksums for devices, error correcting codes, RAID architectures, end to end correctness checks.

## Types of Storage Device Failures

### Sector and Page Failures

Problem: Data in the sector of a disk is lost. Flash page failures are the same thing but on SSDs.
Cause: The magnetic coating on spinning disks get scratched off, machine oil coats a sector of the spinning disk, the disk head gets too far from the surface. In flash storage, a page can get worn out from using it too many times.
Mitigation:

- Error correcting codes: Encode data with redundant metadata that tells you if a sector becomes damaged
- Remapping: remap failed sectors to good ones
  Pitfalls:
- Assuming that non-recoverable read error rates are negligible. A bit error rate of one sector per 10\*\*14 bits means in a 2TB read there is a 10% chance of error
- Assuming non-recoverable read error rates are constant. Specific workloads can wear out some parts more than others
- Assuming failures are independent. Errors are usually correlated by time and space
- Assuming uniform error rates. Some models are just worse than others

### Device Failures

Problem: Device stops being able to read or write to any disk sector.
Cause:

- For disks: Disk heads become damaged, power surge damages electronics.
- For flash devices: Wear out and device runs out of space for remapping
  Mitigation: Notify the device reading the disk that there is an error
  Pitfalls:
- Trusting advertised failure rates. Big tech lies
- Failures are usually correlated
- A mean time to failure is not the same as its useful life
- Failure rates are not constant, there are devices with disk infant mortality, and devices that die after wearing out
- Ignoring warning signs
- Devices don't behave identically, even if manufactured identically.

## RAID Multi disk redundancy for error correction

- Idea: Spread data redundantly across multiple disks in order to tolerate individual disk failures.
- Mirroring: The system write each block of data to two disks and can read data from either disk. If one fails, read from the other disk
- Rotating Parity: store several blocks of data on several disks and protect those blocks with one redundant block stored on yet another disk.
  - Each disk has 1 / # data blocks of its space dedicated to parity and is responsible for storing (# data blocks - 1) / # data blocks.
- Striping data: a strip of several sequential blocks is placed on one disk before shifting to another disk for the next strip. Thus, requests larger than a block but smaller than a strip require just one disk to seek then read/write.
- Recovery: If a disk suffers a sector failure, it reports it when there is an attempt to read the sector then remaps the damaged sector to a spare and reconstructs the lost sector from the other disks and rewrites it to the original disk.
- Ways RAID can fail
  - Both disks fail
  - One disk fails and the other disk's sector fails
  - Both disk's sectors fail

### Improving RAID Reliability

- There are three main things we can do to improve reliability: increase redundancy, reduce non recoverable read error rates, reduce mean time to repair
- To increase redundancy, duplicate it to 3 disks instead of 2.
- To reduce non-recoverable read error with scrubbing, read the entire contents of disk, find sectors with unrecoverable read errors, and reconstruct lost data from remaining RAID disks
- To reduce non recoverable read error rates with more reliable disks, use higher quality disks
- To reduce mean time to repair, use hot spares: disks that are idle but plugged into a server so that if one disk fails, the hot spare can be automatically activated
- Reduce mean time to repair with declustering: split the reconstruction of a failed disk accross multiple disks.
- Pitfalls: assuming uncorrelated failures, ignoring risk from latent errors, not implementing scrubbing, not having a backup

# Uniprocessor Scheduling

## Motivation

Given multiple tasks, how do we decide which one to do first?

## Uniprocessor

Everything discussed below is work conserving - ie never leaves processor idle

## Fifo/FCFS

Perform each task in the order it arrives. Run each task until it finishes

**Pros:** No overhead, fair since every task waits their turn, simple  
**Cons:** If a long task is scheduled before a short task, convoy effect dramatically raises average wait time. Average response time depends on task size distribution

## Shortest Job First (SJF) / Shortest Remaining Time First (SRTF)

Perform the shortest job first. Run each task till it finishes

**Pros:** Optimal strategy  
**Cons:** Impossible, Increases the variance of average response time, Starvation can occur due to too many short jobs coming in which blocks off the longer jobs on the queue

This shows that there is a tradeoff between reducing average response time and the variance of average response time

## Round Robin

Tasks take turns on the processor, if the task is not completed in the given time quanta, it goes to the end of the queue  
**Short time quanta:** Too much time spent context switching, no work will be done
**Long time quanta:** Tasks spend a long time waiting for a turn
**Infinite time quanta:** Same as FIFO
**Zero time quanta:** SJF

Simultaneous multithreading(SMT)/Hyperthreading: Switch between two processors on a cycle by cycle basis

**Pros:** No starvation  
**Cons:** Context switching takes time, need to evict cache in addition to registers of processor

## Max min fairness

Iteratively maximize the minimum allocation given to a particular process until all resources are assigned

If all processes are compute bound, this turns into round robin.

If some processes don't use entire share of processor becasue they are short running or IO bound, give those processes their entire request and redistribute the unused portion to remaining processes. Recursively distribute unused portions until nothing left. When no remaining requests can be fully satisfied, divide the remainder equally among all remaining processes.

**Pros:** Good for IO bound tasks, no starvation  
**Cons:** Computationally expensive

## Multi Level Feedback

Extension of Round Robin but uses multiple round robin queues, each with a different priority level and time quantum. Higher priority level queues go before lower priority queues, higher priority levels have shorter time quanta than lower levels.

Meant to favor short tasks over long ones. New tasks enter at top, drops to lower queue when it uses up its time quanta. If task yields the processor bc it is waiting for IO, it stays on the same level, and is removed when task completes.

Increases Priority when process receives less than its fair shares, reduces priority when process has more than fair share

Goals: Run short tasks quickly, Low Overhead, no starvation, fair, defers maintenance tasks

# Virtual Memory

## General Address Translation

### Motivation

We want to store addresses in a different place than the computer expects. We can then use address translation to move between our what the computer thinks is storing stuff and the actual stuff.

## Useful things we can do with Address Translation

1. **Process Isolation:** Protect the operating system kernel and other applications against buggy or malicious code. This is done by limiting memory references by applications so that some applications cannot access that memory. Also, address translation can be used by applications to construct sandboxes for third party extensions.
2. **Interprocess Communication:**

## Segmentation

- Translation happens on every instruction fetch, load, or store.
- Each virtual address space has holes, leading to efficient segmentation for sparse address spaces
- When is it OK to address outside valid range? This is how the stack grows
- What if not all segments fit in memory? One solution is swapping, where we save the segment of the current process to disk then switch to the next process. A better way is to only keep parts of memory we need.

## Problems with Segmentation

1. Must fit variable sized chunks into physical memory, there could be multiple gaps but no segment is small enough to fit in any of the gaps.
2. Segmentation may move processes multiple times to fit everything.
3. Limited options for swapping to disk

## Solutions to Fragmentation

Paging - Allocate physical memory in fixed size chunks where every chunk of physical memory is equivalent. The pages should be smaller than segments

**Implementation:** Use a page table per process, which resides in physical memory and contains a physical page and permission for each virtual page(valid bits, read, write, etc.)
The offset from virtual address is copied to physical address, the virtual page number is all the remaining bits.

**Usage:** Page sharing is used in the kernel region of every process which has the same page table entries. Also used when different processes run the same binaries, user level system libraries, and shared memory segments between different processes. (Can actually share objects between processes as long as the page is mapped into the same place in the address space)

## Page Table

**Page Table Context Switch:** What need to be switched on a context switch? Page table pointer and limit.  
Pros: Simple memory allocation, easy to share
Cons: Poor when address space is sparse, table is really big

**Implementation:** Page tables are maps from Virtual Addresses to Physical Addresses. A simple implementation is a very large lookup table, but can also be implemented via Trees or Hash tables

**What is in a Page Table Entry(PTE):**

- Pointer to the next level page table or actual page
- Permission bits: valid bits, read-only, write-only, read-write
- Invalid PTE can imply region of address space is actually invalid or page/directory is just somewhere else other than memory

**Using the Page Table Entry(PTE):**

1. Used in demand paging: keeps only active pages in memory, place others on disk and mark PTEs as invalid
2. Used in Copy on Write: Unix fork gives copy of parent address to child. Address spaces disconnected after child created. Implementation - make copy of parent's page tables, mark entries in both sets of page tables as read-only, page fault on write creates two copies
3. Used in Zero Fill on Demand: New data pages stay zeroed, mark PTEs as invalid, page fault gets zeroed page.

## Multi-Level Translation

**Pros:**

- Only need to allocate

## Paging - Caching and TLBs, Demand Paging

Problem: A single level page table can only have so many addresses, how do we get more space?

Solution: Use a multilevel page table, where the first page table points to the second page table which points to the third page table

Pros: Only need to allocate as many page table entries as we need for applications.(Sparse address spaces are easy)
Easy memory allocations, Easy Sharing at segment or page level(needs additional reference counting)

Cons: one pointer per page, page tables need to be contiguous, two or more if more than 2 levels lookups per reference(very expensive!)

Problem: Given a virtual address, how to map to physical address?
Solution: Use a hash table aka Inverted Page Table whose size is independent of the virtual address space. The size is directly related to the amount of physical memory.

Cons: The complexity of looking things up in hardware in a hash table makes things hard. There will also be cache issues since each thing in the hash table is far from everything else. Total size of page table = number of pages used by program in physical memory

## Address Translation Comparison

| Technique            | Advantages                                                            | Disadvantages                                                |
| -------------------- | --------------------------------------------------------------------- | ------------------------------------------------------------ |
| Simple Segmentation  | Fast context switching since the segment map is maintained by the CPU | External Fragmentation                                       |
| Paging(Single-Level) | No external fragmentation, fast + easy allocation                     | Large table size, internal fragmentation                     |
| Paged Segmentation   | Table size ~ # of pages in virtual memory, fast + easy allocation     | Multiple memory references per page access                   |
| Multi-Level Paging   | Table size ~ # of pages in virtual memory, fast + easy allocation     | Multiple memory references per page access                   |
| Inverted Page Table  | Table size ~ # of pages in physical memory                            | Complex Hash function required, no page table cache locality |

## How is Translation Accomplished? - The Memory Management Unit (MMU)

The MMU must translate virtual addresses to physical addresses for every instruction fetch, every load, and every store.

Implementation:

1. In the first level of the page table, Read PTE from memory, check that its valid, merge address
2. In the second level page table, read and check the first level, then read and check and update the PTE
3. Repeat for the n levels in the page table

Problems: This is really slow, since a single load/store will require n reads, n being the number of levels in the page table.

Solution: Caching - take advantage of the principle of locality to present the illusion of memory as the cheapest technology.

**Temporal Locality:** Keep recently accessed data items closer to processor.
**Spatial Locality:** Move contiguous blocks to the upper levels.

### How to make Address Translation Fast?

Cache the results of recent translation(the results of the MMU). Cache the PTE extra info, physical frame number, using the virtual page number as the key

**Translation Look Aside Buffer (TLAB):**
The name of the cache we use for translations. The TLAB records recent Virtual Page Number to Physical Frame number translation.

- If the Virtual Page Number is present, return the Physical Address.
- If not, the use the MMU to find and save the result of the physical address.
  - Instruction accesses spend a lot of time on the same page since their accesses are sequential
  - Stack accesses have definite locality of reference,
- We can have a multilevel TLAB with different size/speeds

### Sources of Cache Misses

**Compulsory:** Cold start or context switch - the first access to a block. Not much we can do here.
**Capacity:** Cache cannot contain all blocks accessed by the program. Solution - increase cache size.
**Conflict/Collision:** Multiple memory locations mapped to the same cache location. Solution - increase cache size or increase associativity.
**Coherence(Invalidation):** Other processes eg I/O updates main memory, while the copy of main memory in the cache is not updated since it was some other process that modified main memory. THIS IS VERY COMMON IN MULTI-PROCESS PROGRAMS!
**False Sharing:** Common cause of coherence misses.

## How is a Block found in a cache?

**Block:** Minimum quantum of caching. Data Select field is used to select data within blocks. Many caching applications don't have data select fields.
**Index:** Used to Lookup Candidate in a Cache.
**Tag:** Used to identify actual copy.

**Direct Mapped Cache:**
Cons: Conflict Misses. Since data is stored 1:1 it is likely there is already something in that slot and we will need to evict that old piece of data

**N-way set associative:** N entries per cache index.
**Fully associative cache:** Every block can hold any line, address does not include a cache index. Compare cache tags of all cache entries in parallel.

### Replacing blocks on a cache miss

For Direct Mapping, there is only one possibility.
For Set associative or Fully associative, do it randomly or evict the least recently used block

### What happens on a cache write?

**Write through:** The information is written to both the block in the cache and to the block in the lower level memory.
Pros: read misses cannot result in writes
Cons: The processor is held up on writes unless writes are buffered.
**Write back:** The info is written only to the block in the cache. The modified cache block is written to main memory only when it is replaced.
Pros: repeat writes not sent to DRAM processor not held up on writes
Cons: More complex, read miss may require writeback of dirty data

## Physically Indexed vs Virtually Indexed Caches

- Physically Indexed Caches (More common)
  - Address handed to cache after translation, the page table holds physical addresses
  - Pros: Every piece of data has a single place in the cache and the cache stays unchanged, thus a cache remains unchanged through a context switch
  - Cons: The TLB is in the critical path of the lookup
- Virtually Indexed Caches(Less common)
  - The address is handed to the cache before translation, the page table holds virtual addresses
  - Pros: The TLB is not in the critical path of lookup
  - Cons: The same data could be mapped in multiple places of cache and may need to flush cache on context switch.

## What TLB Organization Makes Sense?

Requirements: The TLB needs to be really fast since it is in the critical path of memory access. Also, there needs to be very few conflicts, otherwise miss times will be extremely high because we need to traverse the page table.

TLB lookup is serial with cache lookup, thus. the speed of the TLB can impact the speed of access to the cache

## Paging and Address Translation - Conceptual Questions

If the physical memory size (in bytes) is doubled, how does the number of bits in each entry of the page table change?
It doesn't?
Answer: It increases by one bit since now there are twice as many physical pages so the physical page number needs to expand by 1 bit.

If the physical memory size (in bytes) is doubled, how does the number of entries in the page table change?
Doubled
Answer: No change, the number of entries in the page table is determined by the size of the virtual address and the size of a page, it isn't affected by the size of physical memory.

If the virtual memory size (in bytes) is doubled, how does the number of bits in each entry of the page table change?
It doesn't
Answer: No change, the size of each entry of the page table has nothing to do with the virtual memory
If the virtual memory size (in bytes) is doubled, how does the number of entries in the page map change?
It is multiplied by a factor of n, n being the number of levels?

If the page size (in bytes) is doubled, how does the number of bits in each entry of the page table change?
Doubled

If the page size (in bytes) is doubled, how does the number of entries in the page table change?
It is multiplied by a factor of n, n being the number of levels?

## Page Allocation

Suppose that you have a system with 8-bit virtual memory addresses, 8 pages of virtual memory, and 4 pages of physical memory.

How large is each page? Assume memory is byte addressed.
8 + 8 + 4

What will the page table look like if the program runs the following function? Page out the least recently used page of memory if a page needs to be allocated when physical memory is full. Assume that the stack will never exceed one page of memory?

What happens when the system runs out of physical memory? What if the program tries to access
an address that isn’t in physical memory? Describe what happens in the user program, the operating system, and the hardware in these situations.

## Address Translation

Consider a machine with a physical memory of 8 GB, a page size of 8 KB, and a page table entry size of 4 bytes. How many levels of page tables would be required to map a 46-bit virtual address space if every page table fits into a single page?

List the fields of a Page Table Entry (PTE) in your scheme.

Without a cache or TLB, how many memory operations are required to read or write a single 32-bit word?

With a TLB, how many memory operations can this be reduced to? Best-case scenario? Worst-case scenario?

## Demand Paging

## Questions to ask

1. What is `0x12345678` `123456` is what? `78` is the offset in the page table?
2. Caching in industry? What is it used for?
3. How do we know that the main memory is different from the cached memory when detecting coherence misses?
4. Why isn't LRU implemented using a heap

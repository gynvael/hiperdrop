# About HiperDrop #

![http://gynvael.vexillium.org/ext/hd_help_wiki.png](http://gynvael.vexillium.org/ext/hd_help_wiki.png)

HiperDrop is a simple Windows console application that can be used to acquire a full memory dump and a memory map of a process. In current version HiperDrop can attach to a process in two different ways (OpenProcess or via the debugger API), download the memory using two searching "algorithms" (VirtualQueryEx or just "brute force" page-by-page) and write the output in three different ways (file per region, one file or one big file). For details on each method please check below.

P.S. Current version supports only 32-bit processes. 64-bit support is on the ToDo list.

# Attachment methods #
Currently there are two supported attachment methods:

1. **OpenProcess**, Option -o (DEFAULT)

Attachment to a process is done using OpenProcess with two flags:
  * PROCESS\_VM\_READ - for reading memory
  * PROCESS\_QUERY\_INFORMATION - for queries about the memory
In most case this method should be sufficient.

2. **Debugger API**, Option -d

Attachment to a process is done using the debugger API, or to be exact, the DebugActiveProcess function. When HyperDrop finishes downloading memory it tries to detach from the process using DebugActiveProcessStop.

Since DebugActiveProcessStop is available starting from Windows XP, **HiperDrop will not work on Windows 2000 or earlier**. In case the detachment fails, HyperDump will let the user decide when to exit (and kill the attached process).

_TODO: Make HiperDrop work on Windows 2000, just without detaching._

_TODO: 3. and 4. kernel mode and sending data from the inside of the process via pipes._

# Read methods #
Currently there are two read algorithms supported:

1. **VirtualQueryEx**, Option -q (DEFAULT)

The function VirtualQueryEx is used to query blocks of memory and quickly skip unused parts, and find the size of allocated blocks. This method is very fast, and so, it's also default.

2. **Page by page**, Option -b

A brute-force style method, that works by calling ReadProcessMemory on each page without prior queries to check if the page even exists.

This is much slower than the previous method but in some cases (e.g. when VirtualQueryEx is concealing some area due to unknown reasons) it may come in handy. To tell you the truth I never did see any such situation, but, as they say, better safe than sorry :)

# Output methods #
There are three (well, actually four) supported output methods:

1. **Multiple files**, Option -m (DEFAULT)

For every block of memory a single file is created. Such a file may be from 1 page long (4096 bytes), to multiple page long. This is the default, since it's handy in most (not all) cases.

2. **Single file**, Option -1

All the blocks are written to a single file. The free pages are skipped, so the blocks are not really on any predifined offsets in the file. In the map file there is information on which offset what memory area was written to.

This method is quite good if you are searching for something in the memory, but you totally don't care about where it was placed (or you can afford to parse the map file).

3. **Single huge file (sparse)**, Option -1f

Like the above method, except that free pages are written to as zero-filled to the file, meaning that memory address and file offsets are exactly the same, but the file will be 2 to 4 GB large.

![http://gynvael.vexillium.org/ext/hd_sparse_wiki.png](http://gynvael.vexillium.org/ext/hd_sparse_wiki.png)

By default on NTFS partition this option tries to use the [sparse file](http://en.wikipedia.org/wiki/Sparse_file) feature. This works by telling the NTFS which areas of the file are "empty" or "zero-filled", and such areas are not physically stored on the disk - instead, NTFS keeps record of these areas and in case anything wants to read from them, it just returns zeroes. Thanks to this method, even if the file is 2-4 GB large, it uses way less disk space (e.g. a dump of calc.exe uses 60MB on the disk).


4. **Single huge file (non-sparse)**, Option -1f -n

Same as above, but instead of using sparse files, the free pages are actually "physically" written as zeroes to the disk. This should be used as fallback in case you cannot use the sparse files for some reason (non-NTFS or remote partition, etc).

# ToDo list #
Grep the source file for TODO phrase, and also see the begining of the file :)
# The Filesystem is Awesome (Video Summary)

Video by Ryan Baker (26:35). Core question: what does it mean for a file to "live somewhere"?

## The Big Idea — Three Layers, One Direction

A file isn't one thing at one address. It's three separable pieces, connected by pointers that only run one way:

```
 DIRECTORY (a table)        INODE STORE                    DISK
┌────────────────────┐     ┌───────────────────────┐     ┌───────────────┐
│ name  →  number     │ →  │ #501: type, size,      │ →  │  raw bytes    │
│ "zsh" →   501        │    │ owner, mtime,          │    │  (content —   │
│                     │    │ *pointers to data*     │    │  maybe not    │
└────────────────────┘     └───────────────────────┘    │  contiguous)  │
   NAME lives here            METADATA lives here          └───────────────┘
                                                            CONTENT lives here
```

- A name maps to a number; a number (via the inode) maps to metadata + data — but nothing points back.
- An inode has no idea what it's named. The data has no idea an inode points at it.
- This one-way, decoupled structure is the entire reason hard links, symlinks, and cheap renames are possible.

## A Path Is a Route, Not an Address

`/bin/zsh` looks like a static coordinate but is actually a set of instructions executed left to right: start at root, step into bin, find zsh.

- **Proof it's a route:** inserting `.` (this directory) and `..` (parent) mid-path still resolves correctly (e.g. `/bin/../bin/zsh`) — a static address wouldn't tolerate that.
- This step-by-step resolution is called a **walk**. Each name, including the dots, is looked up in order from wherever the last step landed.
- If any intermediate name is invalid, the whole path breaks — even one that would "cancel out" algebraically — proof that resolution is sequential, not simplified first.
- **Absolute path**: starts at root, first character is `/`. **Relative path**: starts from wherever you currently are. Same underlying walk, only the starting point differs.

## What Are We Walking Through?

A disk has no built-in names or folders — just numbered blocks. Two tempting models fail before landing on the real one:

| Model | Idea | Why it fails |
|---|---|---|
| Giant flat table | full path → byte location | Rename one folder → every file under it stores its full path → hundreds of thousands of rewrites |
| Disk carved as a tree | give root the whole disk, subdivide per directory | Must size each region before knowing what goes in it; moving a file = physical byte-copy |
| Chained small tables (the real model) | each directory is its own `name → number` table; directories reference each other, forming a tree | Rename a folder → exactly one row changes, in one table. Nothing underneath ever recorded its own full path. |

"A directory doesn't just have a table — it **is** a table." This is also why the walk must be sequential: no single table can resolve a whole path in one shot, because that information doesn't live in one place.

## Inspecting Directories — ls, Hidden Files, Naming Rules

`ls` isn't "listing contents" — it's inspecting the directory-as-table. By default it prints only the name column, and hides two real rows every directory has: `.` and `..`.

```
$ ls -a          (-a = "all")
.   ..   notes.txt   photo.png
```

Hidden files are a pure display convention — `ls` hides anything starting with a dot, nothing more. A directory is itself a file, so trying to `cat` a directory fails with a specific, non-confused error:

```
$ cat .
cat: .: Is a directory
```

- Long listing (`ls -la`) reveals file type via the leading character: `-` regular file, `d` directory, `l` symlink.
- A filename can contain almost anything — spaces, newlines, emojis. Exactly two bytes are forbidden: `/` (path separator) and the null byte (marks where a name ends).
- File extensions aren't real: the `.` in `notes.txt` is just another character. `.txt` means nothing to the filesystem itself — all files are just bytes.

## Inodes — Where Metadata Actually Lives

```
$ ls -li
16777230  -rw-r--r--  1 ryan staff  27  notes.txt
   ↑ inode number
```

An **inode** holds everything the system knows about a file: size, type, owner, mtime, and pointers to where its content sits on disk. `stat` reveals the metadata but never the raw pointers — those stay internal to the filesystem implementation.

**Recap:** name (in a directory) → number → inode (metadata) → data. Every arrow points one way only.

## Hard Links — One File, Multiple Names

```
$ echo "hello" > a
$ ln a b                 ← new row: name="b", same inode as "a"

$ ls -li
16777231  a
16777231  b              ← identical inode numbers = same file, two names

$ rm a                   ← rm is really "unlink" — removes a name, not the data
$ cat b
hello                     ← "b" is completely intact; data never knew "a" existed
```

Every inode has a **link count** tracking how many names point at it. A file is only actually freed once that count hits zero — proof a file can, in a real sense, live in two places at once.

## Working Directory — Why cd Can't Be a Normal Program

- Every ordinary program the shell runs is a **child process**. A child can change its own working directory, but not its parent's.
- If `cd` were a normal program, it would `chdir()` inside itself, then exit — taking the change with it. The shell's own working directory would never move.
- So `cd` must be executed inside the shell process itself, not spawned as a child.
- `pwd -P` ("physical") sometimes differs from plain `pwd` — a hint the current directory was reached through a symlink.

## Symbolic Links — A File That's Just a Path (as Text)

| | Hard link | Symbolic link |
|---|---|---|
| What it is | Another row pointing at the same inode | A separate file with its own inode |
| Content | N/A — shares the original's data | The target path, stored as text |
| Resolution | None needed — same data | Re-read + re-walked from scratch every time |

- `ls -l` shows a leading `l` for a symlink's file type.
- A symlink's size equals the length of its target path string — e.g. 8 bytes for a link pointing at `/bin/zsh`.
- Because the target is re-resolved on every walk, two symlinks can point at each other and loop: `cat` fails with "too many levels of symbolic links" once the system gives up.

## The Answer: A File Doesn't Live in One Place

Its **name** (possibly several names) is stored across one or more directories. Its **metadata** lives in an inode. Its **content** is stored on disk — and not even necessarily in one contiguous block.

## Command Reference

| Command | What it actually does |
|---|---|
| `ls` / `-a` / `-l` / `-i` | Reads the directory's name column; `-a` shows dotfiles; `-l` shows type/permissions; `-i` shows the inode number |
| `cat` | Reads file content; fails with "Is a directory" on directories |
| `stat` | Inspects an inode's metadata — not its raw data pointers |
| `ln A B` | Hard link — new name `B` reusing `A`'s inode number |
| `ln -s A B` | Symbolic link — new file `B` whose content is the path `A` |
| `rm` | Actually `unlink` — removes a name; data survives while link count > 0 |
| `pwd` / `pwd -P` | Working directory; `-P` resolves through symlinks to the physical path |
| `cd` | Built into the shell itself — a subprocess couldn't change the shell's own working directory |

## Connections

- Same "avoid touching everything on a single change" reasoning as DDIA's storage-engine trade-offs — chained small directory tables localize the cost of a rename the same way B-trees/LSM structures localize the cost of an update.
- Docker's overlay/union filesystem builds container layers out of this exact name→inode→data separation — a copy-on-write layer gives a modified file a new inode while the directory entry above it is simply repointed, the same one-directional-pointer trick as a hard link.

## Left Open (per the video)

- Exactly how file content maps onto possibly non-contiguous disk blocks.
- Deeper shell internals — companion video "How the Shell Works" (same channel).

## Addendum: Why `..` Isn't Redundant with `.`

`..` stores a row mapping the name `".."` to the **inode number of the parent directory** — but this is deliberate redundancy, not a design flaw.

Each directory only knows its own contents — nothing stores a map of the whole tree. Without a `..` entry, finding a directory's parent while sitting inside it would require either a manually-tracked stack of every directory passed through, or scanning the entire filesystem for a name pointing at the current inode. `..` is a cached backward pointer purely so "go up" is an O(1) table lookup — the same denormalization trade-off as duplicating a column in a database to avoid a join.

The values can coincide (root's own `.` and a child directory's `..` both resolve to root's inode) but they're stored in two different files for two different purposes — self-reference vs. parent-reference — and one can't be derived from the other without already being in the other directory.

Since `.` and `..` are just ordinary rows reusing an inode number, they are literally hard links. A directory's link count is built from: +1 for its entry in its parent's table, +1 for its own `.`, and +1 for each subdirectory's `..` pointing back at it. So a directory with 3 subdirectories has a link count of 5 (1 + 1 + 3), and an empty directory always sits at exactly 2 — the classic Unix trick for estimating subdirectory count from `stat` alone (`link_count - 2`), though some modern filesystems (e.g. ext4 with `dir_nlink`) cap the reported count once very large. Root's own `..` points to itself, since root has no parent.

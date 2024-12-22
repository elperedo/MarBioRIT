# Unix Tutorial

## Goals
By the end of this introduction, you should be able to:
- Navigate and understand the directory structure.
- Copy, create, and edit files and directories.
- Compress and uncompress files.
- Copy files between your computer and your home directory.

*Ask for help if any of these things aren't working by the end!*

---

## The RIT Computer System
[RIT Research Computing](https://www.rit.edu/researchcomputing/)

---

## Basic Overview
When you connect to a server using `ssh`, you are running a program called a **shell** on the server. The shell interprets what you enter on the command line and tells the server's operating system what to do. There are many different shells; we will be using one called **bash** (the "Bourne Again Shell").

The bash shell starts by default; you don't need to do anything. The basic commands we are using will work in any shell. If you'd rather use another shell, feel free.

> **Note:** Your mouse and pointer do not work in the terminal window. You will need to use the arrow keys or keyboard shortcuts (described below).

### Important Obscure Keys
Some keys are used frequently in UNIX commands but can be difficult to find on many keyboards. Find these symbols and note their common Unix name(s):

| Symbol  | Name                |
|---------|---------------------|
| `~`     | Tilde               |
| `/`     | Forward Slash       |
| `\`     | Backslash           |
| `|`     | Pipe                |
| `#`     | Hash or Number Sign |
| `$`     | Dollar Sign         |
| `*`     | Asterisk            |
| `` ` `` | Backtick (not the same as `'` single quote) |

---

## Basic Syntax
Unix commands follow the general format:
Sure! Here's the text formatted in Markdown for easy export. You can copy and paste this directly into HackMD or any Markdown editor.

```markdown
# Unix Tutorial

## Goals
By the end of this introduction, you should be able to:
- Navigate and understand the directory structure.
- Copy, create, and edit files and directories.
- Compress and uncompress files.
- Copy files between your computer and your home directory.

*Ask for help if any of these things aren't working by the end!*

---

## The RIT Computer System
[RIT Research Computing](https://www.rit.edu/researchcomputing/)

---

## Basic Overview
When you connect to a server using `ssh`, you are running a program called a **shell** on the server. The shell interprets what you enter on the command line and tells the server's operating system what to do. There are many different shells; we will be using one called **bash** (the "Bourne Again Shell").

The bash shell starts by default; you don't need to do anything. The basic commands we are using will work in any shell. If you'd rather use another shell, feel free.

> **Note:** Your mouse and pointer do not work in the terminal window. You will need to use the arrow keys or keyboard shortcuts (described below).

### Important Obscure Keys
Some keys are used frequently in UNIX commands but can be difficult to find on many keyboards. Find these symbols and note their common Unix name(s):

| Symbol  | Name                |
|---------|---------------------|
| `~`     | Tilde               |
| `/`     | Forward Slash       |
| `\`     | Backslash           |
| `|`     | Pipe                |
| `#`     | Hash or Number Sign |
| `$`     | Dollar Sign         |
| `*`     | Asterisk            |
| `` ` `` | Backtick (not the same as `'` single quote) |

---

## Basic Syntax
Unix commands follow the general format:

```
command -options target

- Not all commands need options (also called **flags**, generally preceded by a single or double hyphen `-` or `--`), but others require them.
- Some options are followed by a parameter value.
- **Important**: Always place a space between the command and the hyphen of an option.

For example:
- `cd /class/marbio-shared` uses the command `cd` (change directory) and the target `/class/marbio-shared` to move into the directory called `marbio-shared`.
- `ls -l /class/marbio-shared` uses the command `ls` (list), the option `-l` for long-list, and the target `/class/marbio-shared` to list the contents in the "long list" format.

To get help on any command, type:
```
[command] --help

```
And, of course, Google can provide many answers (though double-check them!).

---

## Notes on Syntax for Directory Structure
You cannot point and click from the command line. Here are some tricks for navigating:

- **`.`** (single dot) indicates the present working directory.  
  For example: `cd .` will keep you where you are.
- **`..`** (two dots) indicates the parent directory of the present working directory.  
  For example: `cd ..` will move you up one directory, and `cd ../../` moves you up two.
- **`/`** (forward slash) by itself or at the start of a path refers to the root of the file system.
- **`~`** (tilde) refers to your home directory. On the class machines, your home directory is `/class/[your username]`.

### Suggestions Concerning File and Folder Names
- **Never use spaces** in filenames. Use underscores (`_`), dots (`.`), or hyphens (`-`), or use CamelCase.
  - If a file contains spaces, surround the name in double quotes:
    ```
    mv "bad filename" good_filename
    ```
- Avoid non-alphanumeric characters like `#@!*&^`, especially `?`, `*`, `\`, or `/` in filenames, as these have reserved functions.
- **Dots (`.`)** are good separators in filenames, and you can use as many as you want (though avoid using multiple in a row).
- The suffix of a filename does **not** determine the type of file it is. The suffixes are just conventions.

---

## Quotes and Slashes
- Double quotes (`"`) and single quotes (`'`) are **not interchangeable** with backticks (`` ` ``). Each has different functions.
- Unix uses the forward slash (`/`) for directory structure, while the backslash (`\`) is used in MS-DOS and Windows for the same purpose.



# Wildcards (Globbing)

You can't use your mouse to select multiple files in a terminal window. The asterisk (`*`) can be used to match "anything of any length" and `?` can be used to match "any single character."

```bash
ls *txt
```
Lists everything ending in `txt`, and

```bash
ls files.??
```
Lists all files with exactly two characters after the dot.

And of course, you can get more fancy:

```bash
ls MyProject_*R1*fast?.gz
```
Would list all the forward reads of your sequencing project, be they `fasta` or `fastq`.

---

## Compressed and Archived Files

Next-gen sequencing analysis generates huge files, and sometimes lots of files. Your life will be much simpler if you are comfortable compressing and uncompressing files and archiving sets of files.

### Compressing Files

Common compression algorithms such as `gzip`, `zip`, and `bzip2` can drastically reduce the size of text files such as `fastq` and `fasta` files.

```bash
gzip my.fastq
```
Will compress `my.fastq` to `my.fastq.gz`.

```bash
gzip -d my.fastq.gz
```
Will decompress `my.fastq.gz` to `my.fastq`. A gzipped `fastq` file takes 30% or less of the space of the regular file. With some types of files, compression ratios of 1:10 are common.

---

### Archiving Files

Multiple files can be archived into a single file using the `tar` command (originally "tape archive"). This helps organize your projects, makes them easier to share with colleagues, and has a number of other advantages that will make your sysadmin like you if you keep your rarely used data in tar files.

The command is `tar`. You want to compress files into a new file using flags and, where necessary, parameters:

```bash
tar -c -v -f myproject.tar *fa *fastq *log README
```

This will make a new file called `myproject.tar` which contains all files in the current directory ending in `fa`, `fastq`, and `log`, as well as the file called `README`. Because we added the `-v` flag (verbose), it will display the name of each file on the screen as it is processed. **It does not delete the files**, you have to do that yourself when you're done.

**Pro Tip**: If you want to sound like a Unix geek (and that's why you're here, right?), you can call tar files "tarballs."

You can see the contents of a tar file using `-t` for list (since `-l` was already taken) or the more obvious `--list`:

```bash
tar --list -f myproject.tar
```

You can extract the contents of a tar file, which does not delete the tar file:

```bash
tar -x -f myprojects.tar
```

Here's a formatted version for HackMed:

---

**Understanding `tar`**

The `tar` command is a classic tool that's been around for a long time, which means it has some quirks that aren't necessarily intuitive. Due to its long history, `tar` supports a range of flags and options in a way that's different from most modern programs. Specifically, `tar` allows flags to be combined and ordered in a non-standard way, without requiring hyphens. For example, you might see:

```bash
tar cfv myproject.tar *fa *fastq *log README
```

or

```bash
tar xfv myproject.tar
```

While this may seem confusing at first, this convention is widespread and you’ll need to get used to it.

**Combining Compression and Archiving**

You can archive and compress files simultaneously using `tar` with gzip compression. Here’s how you can do it:

```bash
tar czfv myproject.tar.gz *fa *fastq *log README
```

In this command:
- `c` stands for create a new archive.
- `z` specifies gzip compression.
- `f` indicates the filename of the archive.
- `v` enables verbose mode to show the progress (this is optional but helpful for large files).

**File Naming Conventions**

It’s worth noting that while `.tar.gz` is a common extension for compressed tar files, it's just a convention. Files can have different extensions, but sticking to this convention makes it easier to identify them.

**Pop Quiz**

What command can you use to uncompress and extract `myproject.tar.gz`? 


---

## Be Lazy

Typing is hard, and mistakes are inevitable. Since there's no auto-correct, here are some tricks to make life easier:

- **Auto Complete**: Type the first few letters of a command or filename and press the `TAB` key. The shell will autocomplete the word as much as it can. If there's only one match, it will complete the word. If there are multiple possibilities, it will complete as much as possible and list all matching files if you press `TAB` twice.

- **History**: The shell remembers your previous commands. Use the up and down arrow keys to scroll through them. This makes it easier to rerun or modify commands with fewer typos.

- **Shortcuts**: Use the `CTRL` key (or `ALT` on a Mac) with the appropriate letter for these shortcuts:
  - `CTRL+a`: Move the cursor to the beginning of the line.
  - `CTRL+e`: Move the cursor to the end of the line.
  - `ALT+b`: Move the cursor back one word.
  - `ALT+f`: Move the cursor forward one word.

## Don't Panic

You can't destroy the system or mess up other people's files easily. If something goes wrong, `CTRL+C` (^C) will stop the current process and return you to the prompt. If that doesn't work, you can always close the Terminal and start over.

## Step 4: Intro-to-Unix Tutorial

### Moving Around

1. **Log in to your server** and start by entering:
   ```bash
   pwd
   ```
   This command prints your current directory (print working directory). "Print" here means the output is shown on your screen, not on paper. A directory is the same as a folder.

2. **Home Directory**: This is your personal directory, named after your username. You can modify its contents but not its name.

3. **Navigate to `marbio-shared` Directory**:
   Move to the `marbio-shared` directory in the `users` directory:
   ```bash
   cd ../marbio-shared
   ```
   or from the top of the directory structure:
   ```bash
   cd /users/marbio-shared
   ```

4. **List Files**:
   ```bash
   ls
   ```

5. **Move into `fastqfiles` Directory**: (Hover here for a hint)

6. **Get a Detailed Report**:
   ```bash
   ls -l
   ```

7. **Sort Files by Time in Reverse Order**:
   ```bash
   ls -l -t -r
   ```
   This combination shows new files at the bottom. You can also use the combined option:
   ```bash
   ls -ltr
   ```

   There are about 80 options for `ls`! Use `ls --help` to see them all.

### Example Output of `ls -ltr`

```
-rw-rw-r-- 1 dmwelch marbioadmin  23398592 Aug  4  2016 TTAGGC_NNNNACGCA_4_R1.fastq.gz
-rw-rw-r-- 1 dmwelch marbioadmin  12890405 Aug  4  2016 TTAGGC_NNNNCGCTC_4_R1.fastq.gz
-rw-rw-r-- 1 dmwelch marbioadmin  21853345 Aug  4  2016 TTAGGC_NNNNACGCA_4_R2.fastq.gz
-rw-rw-r-- 1 dmwelch marbioadmin  11835130 Aug  4  2016 TTAGGC_NNNNCGCTC_4_R2.fastq.gz
-rw-rw-r-- 1 dmwelch marbioadmin 108944229 Aug  4  2016 TTAGGC_NNNNCTAGC_4_R1.fastq.gz
-rw-rw-r-- 1 dmwelch marbioadmin    191767 Aug  4  2016 TTAGGC_NNNNGACTC_4_R2.fastq.gz
-rw-rw-r-- 1 dmwelch marbioadmin    208245 Aug  4  2016 TTAGGC_NNNNGACTC_4_R1.fastq.gz
-rw-rw-r-- 1 dmwelch marbioadmin 102467172 Aug  4  2016 TTAGGC_NNNNCTAGC_4_R2.fastq.gz
-rw-rw-r-- 1 dmwelch marbioadmin  49171429 Aug  4  2016 TTAGGC_NNNNGAGAC_4_R1.fastq.gz
-rw-rw-r-- 1 dmwelch marbioadmin  44569654 Aug  4  2016 TTAGGC_NNNNGAGAC_4_R2.fastq.gz
-rw-rw-r-- 1 dmwelch marbioadmin  58594821 Aug  4  2016 TTAGGC_NNNNGCTAC_4_R1.fastq.gz
-rw-rw-r-- 1 dmwelch marbioadmin  53255083 Aug  4  2016 TTAGGC_NNNNGCTAC_4_R2.fastq.gz
-rw-rw-r-- 1 dmwelch marbioadmin  84920631 Aug  4  2016 TTAGGC_NNNNGTATC_4_R1.fastq.gz
-rw-rw-r-- 1 dmwelch marbioadmin  79443476 Aug  4  2016 TTAGGC_NNNNGTATC_4_R2.fastq.gz
-rw-rw-r-- 1 dmwelch marbioadmin  35476557 Aug  4  2016 TTAGGC_NNNNTCAGC_4_R1.fastq.gz
-rw-rw-r-- 1 dmwelch marbioadmin  32561632 Aug  4  2016 TTAGGC_NNNNTCAGC_4_R2.fastq.gz
```

This output displays file type, permissions, owner, group, file size, creation time, and filename. While permissions are beyond this tutorial, they may come up later.

---


## It’s Hard to Read the File Size

When file sizes have many digits, it can be hard to read. Try using the `-h` flag for human-readable sizes:

```bash
ls -ltrh
```

The `-h` denotes human-readable format.

## Fastq File Name Conventions

Fastq files processed by Illumina's CASAVA program and in-house scripts are named as follows:

- The first six characters represent the index sequenced during the indexing read step.
- The 10 characters after the underscore designate the internal bar code, the first 10 bases sequenced in the forward read step.
- Each cluster is read in forward (R1) and reverse (R2) directions, determined by the Illumina adapters.

For every sequence in an R1 file, there is a corresponding sequence in the R2 file.

## Verbal Conventions

In Unix commands, there is always a space between a program name and flags, and between flags and parameters. However, people often don’t specify these spaces when speaking. For example, when someone says "type el es dash el," they expect you to type `ls -l`, not `ls-l`. This might trip you up.

## Moving to Your Home Directory

You can move back to your home directory using one of the following methods:

```bash
cd ../[username]
cd /users/[username]
cd ~
cd
```

The `cd` command without arguments takes you to your home directory.

## Copying Files

To copy files, use the syntax:

```bash
cp [SOURCE] [DESTINATION]
```

You don't need to be in the SOURCE or DESTINATION directory to copy files.

For example:

```bash
cp /users/marbio-shared/fastqfiles/TTAGGC_NNNNACGCA_4_R1.fastq.gz TTAGGC_NNNNACGCA_4_R1.fastq.gz
cp /users/marbio-shared/fastqfiles/TTAGGC_NNNNACGCA_4_R2.fastq.gz TTAGGC_NNNNACGCA_4_R2.fastq.gz
```

You can rename the destination file:

```bash
cp /users/marbio-shared/fastqfiles/TTAGGC_NNNNACGCA_4_R1.fastq.gz my_R1.fastq.gz
cp /users/marbio-shared/fastqfiles/TTAGGC_NNNNACGCA_4_R1.fastq.gz my_R2.fastq.gz
```

To copy files to the current directory:

```bash
cp /users/marbio-shared/fastqfiles/TTAGGC_NNNNACGCA_4_R1.fastq.gz .
cp /users/marbio-shared/fastqfiles/TTAGGC_NNNNACGCA_4_R2.fastq.gz .
```

Use file globbing to copy multiple files:

```bash
cp ../marbio-shared/fastqfiles/TTAGGC_NNNNACGCA_4_R?.fastq.gz .
```

If you're in `marbio-shared/fastqfiles`, you can use:

```bash
cp TTAGGC_NNNNACGCA_4_R?.fastq.gz /users/[username]/myunixdemo
```

Or:

```bash
cp TTAGGC_NNNNACGCA_4_R?.fastq.gz ~/myunixdemo
```

Copy two `fastq.gz` files (forward "R1" and reverse "R2") into your `myunixdemo` directory.

## Compressing and Archiving

List all files with sizes:

```bash
ls -lh
```

Unzip all gzip files:

```bash
gzip -d *gz
```

Create a compressed tarball:

```bash
tar czfv myfiles.tar.gz *
```

Verify the tar file is created:

```bash
ls -l
```

Delete the original `fastq` files:

```bash
rm *fastq
```

**Note:** There is no recycle bin in the shell. Be cautious with `rm` and wildcards.

Extract the tarball:

```bash
tar xzfv myfiles.tar.gz
```

Unlike unzipping, extracting a tarball does not remove it, and creating a tarball does not remove the original files.

## Looking at File Contents

Use `more` to view a file:

```bash
more TTAGGC_NNNNACGCA_4_R1.fastq
```

Use `less` for a more interactive viewing:

```bash
less TTAGGC_NNNNACGCA_4_R2.fastq
```

Other useful commands:

- `head` — Shows the first 10 lines of a file.
- `tail` — Shows the last 10 lines of a file.

To view a portion of a file, such as the first 1000 sequences, use:

```bash
head -n 4000 TTAGGC_NNNNACGCA_4_R1.fastq > first1000.fastq
```

## Manipulating STDOUT

Redirect standard output (STDOUT) to a file:

```bash
history > HistoryFile
```

Append STDOUT to an existing file:

```bash
history >> AllMyHistoryEver
```

To extract sequences 11-20, use:

```bash
head -n 80 TTAGGC_NNNNACGCA_4_R1.fastq | tail -n 40 > 11-20.fastq
```

Pipes (`|`) combine commands, sending the output of one command as input to another.

## Editing Files

Create a file using `cat`:

```bash
cat > newfile.txt
Hello world!
This is a simple text file
```

Exit input mode with `Ctrl-C`.

To edit a file with Emacs:

```bash
emacs newfile
```

To save and exit Emacs, type `Ctrl-x Ctrl-c`. For additional commands, see the Emacs manual or online resources.

## File Transfer Clients

For GUI-based file transfers, use FileZilla:

1. Download and install FileZilla.
2. Open FileZilla and connect to your server with your credentials.

Alternatively, use the FireFTP add-on for Firefox.

For command-line transfers, `scp` can be faster, especially with wildcards.

## Extra Credit

### Useful Links

- [UNIX/Linux command line cheat-sheet](http://fosswire.com/post/2007/08/unixlinux-command-cheat-sheet/)
- [MS-DOS full command list](https://en.wikipedia.org/wiki/Comparison_of_command_line_interpreters#MS-DOS_commands)
- [A UNIX shell tutorial from Software Carpentry](https://software-carpentry.org/)

### Useful Less Commands

| Command | Description                       |
|---------|-----------------------------------|
| spacebar | Display next page                 |
| return   | Display next line                 |
| `n f`    | Move forward n lines              |
| `b`      | Move backward one page            |
| `n b`    | Move backward n lines             |
| `/ word` | Search forward for word           |
| `? word` | Search backward for word          |
| `h`      | Help                              |
| `q`      | Quit                              |

### Random Online Resources

- [Unix for Beginners](https://www.unix.com/unix-for-dummies-questions-and-answers/)
- [Introduction to Unix](https://www.tutorialspoint.com/unix/index.htm)

---


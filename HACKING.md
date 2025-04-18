This document describes each step of the setup process in more
detail, and provides information about changing the source code and
uploading/downloading files from CTSS.

This repo contains two components: CTSS and ELIZA.

## CTSS structure

Under `ctss/` you will find:

* `dist/`: upstream tar files for the s709 simulator and associated utilities
* `src/`: the build area for the above
* `bin/`: the binaries produced from the above
* `scripts/`: scripts using the above binaries to perform tasks like
setting up CTSS
* `cmd/`: files read by s709 at run time to determine configuration
* `tape/`: virtual tape files read by s709 for tasks like setting up
the OS

## ELIZA structure

Under `eliza/` you will find

* `bin`: utilities for the host (your PC) to deal with source code
  files and upload them to the guest (CTSS)
* `src/SLIP`: the CTSS source code for the SLIP library used by ELIZA,
  which is in a mix of FAP (IBM 7090 assembly language) and MAD (a
  ALGOL-58 inspired high level language)
* `src/ELIZA`: the source for ELIZA itself, which is written in MAD
  and depends on the SLIP library, along with some script files that
  determines ELIZA's personality at run time.

## Building the source

The next few sections describe the steps taken in the Quickstart in
more detail.

## `env.sh`

This file should be read in to your shell to set up the path and
resource directories correctly before running any commands.

```
$ source env.sh
```

## Make the binaries

```
$ source env.sh
$ make-binaries
```

This will unpack and make the s709 and utility binaries found under
[`ctss/dist`](ctss/dist) and copy the completed items to
[`ctss/bin`](ctss/bin).

If this worked, you will see `All binaries compiled successfully`
printed at the end.

See [here](etc/make-binaries.txt) for a transcript of how this looks, but
it may be slightly different on your computer.

## CTSS setup

After you have created the binaries in the above section, run the
following commands.

```
$ source env.sh
$ make-disks
$ format-disks
$ install-disk-loader
$ installctss
```

Some commands will ask you to press Enter a few times and then type q
and Enter to finish. This represents the emulated machine stopping for
operator input - in our virtual environment there is nothing to do
apart from pressing Enter to resume.

See [here](etc/ctss-setup.txt) for a transcript of how this looks.

After this is done, you will have a base line CTSS environment
installed on the disks and drum files in the `dasd` directory.

## ELIZA set up

```
$ source env.sh
$ add-eliza-users
$ upload-all
```

This will make CTSS users for ELIZA and SLIP, and copy all the source
code into the virtual disk files.

See [here](etc/install-eliza.txt) for a transcript of how this looks.

## Starting and logging into CTSS

```
$ source env.sh
$ runctss
```

This will start the emulator and show who is logged in.

In another window, open a telnet client and connect to localhost port
7094, eg by typing `telnet 0 7094`.

Type `login` and then the user name `eliza`. It will prompt for a
password which is the same as the user name,

We use separate users for ELIZA and SLIP as there are some overlaps
between files from both collections, and this allows us to select
which ones to use at run time. Both accounts are under the system
problem number `M1416` for simplicity of use.

Up to 30 users can log in at the same time (via separate telnet
connections to the emulator) but each account can only be logged in on
a single session at a time.

When you logout, the telnet session will close, but you can
immediately re-establish it if needed.

See [here](etc/starting-ctss.txt) for a transcript of how this looks.

## Using CTSS

Some quick points to get you orientated

* There is no prompt, but you will know CTSS is ready for input when
  it prints `R` followed by 2 numbers (which represent the CPU time
  and swapping time for the last command run).
* At the command level, CTSS only understands upper case, but if you
  type lower case it will be translated by the emulator.
* Delete will delete the last character and Control-U will kill a
  whole line. This is an affordance of the emulator; on a a real
  teletype you'd type `#` to delete and `@` to kill; these keys still
  work as such on the emulator.
* Control-C will interrupt a program and Control-\ will quit it.
* Commands are in the familiar format of the program name followed by
  parameters. Optional arguments are enclosed in brackets.
* When you enter a command, CTSS will first type something like `W
  1724.3`. The `W` means you are waiting for the command to load and
  the numbers are the time of day the command started (using tenths of
  minutes after the decimal point).
* CTSS has separate directories for each user, plus some shared
  directories, but it is not possible to create subdirectories.
* File names have two parts, called `name1` and `name2`, separated by
  spaces. The `part2` is by convention the file type, eg `MAD` for MAD
  program files.
* To get a directory listing, type `LISTF`. To narrow the search,
  provide a `name1 name2` parameter. You can use `*` for a wildcard.
  So for example, to see all the MAD source files do

```
LISTF * MAD
```

* To compile a single file `ELIZA MAD` you would run the below. 

```
MAD ELIZA
```

* Note that the MAD compiler knows that the `name2` should be MAD so this
  can be omitted. Had you stored your program code in `ELIZA M` you
  would run `MAD ELIZA M` instead.

* To specify the option to create a listing file, add the `LIST`
  parameter.

```
MAD ELIZA (LIST)
```

* If you do a directory listing after the above, you will see

```
ELIZA MAD
ELIZA BSS
ELIZA BCD
```

* The `BSS` file is the object code from the compilation.
* The `BCD` file is a text file of the listing. Type `P ELIZA BCD` to
  display it.
* To compile the complete set of source code, we will use a runcom
  script - see the section below.
* Type `LOGOUT` when you are done.

The [CTSS Programmer's
Guide](http://www.bitsavers.org/pdf/mit/ctss/CTSS_ProgrammersGuide_Dec69.pdf)
has a great deal more information on using CTSS. Start with section AA
2 for a primer.

I am writing a series of blog posts on CTSS that you may also find
useful at [timereshared.com](https://timereshared.com/ctss/).

The [ctss-kit README](ctss/ctss-kit-readme.txt) provides more details
on how the emulator works with CTSS.

See [here](etc/running-ctss.txt) for a transcript of how this looks.

## Compiling all the source

We use a `runcom` file, which is a type of batch script. You will need
to run these scripts under 3 separate users.

### 1. Compile the huge loader

Ensure CTSS is booted and login as user `SYSDEV`, password `SYSTEM`.

The file we want is called `MKHUGE RUNCOM`. Execute it by running:

```
RUNCOM MKHUGE
```

### 2. Compile the SLIP library

Login as user `SLIP`, password `SLIP`. The file we will use is in
`MAKE RUNCOM`. To execute it:

```
RUNCOM MAKE
```

This will compile all the FAP assembly and MAD source files, producing
object files and listings for each module. At the end, it will run
tests and combine the object files into a library.

The printout from Weizenbaum contained duplicate implementations of
some functions in MAD and FAP. We have chosen the MAD versions as the
standard; if you want to try the FAP versions run `RUNCOM ALT` instead.

### 3. Compile the ELIZA program

Next, login as `ELIZA` (password `ELIZA`) and run the same 
`RUNCOM MAKE` command. (Note this is a different file as we are now in
the ELIZA directory). This will compile all the ELIZA MAD source code.
It will then load ELIZA and SLIP into memory and save this as an
executable called `ELIZA SAVED`.

See [here](etc/compiling.txt) for a transcript of how this looks.

## Running ELIZA

Still logged in as `ELIZA`, issue the following command:

```
R ELIZA
```

ELIZA will prompt you for a script to use. These define the rules it
will follow and are stored in files of name `.TAPE. xxx` where `xxx`
is a three digit number.

We have two examples, `100` for the script found in the printout of
the ELIZA source code, and `200` which is from the CACM article
written by Weizenbaum. Try both! Script 200 has more features, but
also triggers some bugs in the source code. See
[`etc/running-eliza.txt`](etc/running-eliza.txt) for a comparison.

ELIZA will then print a welcome message and prompt you for input. Type
your response, keeping each line below 78 characters, and press Enter
twice to submit. ELIZA will then reply.

If you enter a blank line, ELIZA will offer to save its rules to a
file. Press Enter to print them to the console or enter a number to
create a `.TAPE. xxx` file. Suggest using a new number here to avoid
overwriting an existing script. After this, ELIZA will quit.

Otherwise, when finished, press Control-\ to interrupt back to the
CTSS command line.

See [here](etc/running-eliza.txt) for a transcript of how this looks.

## Shutting down CTSS

It's important to shut down CTSS cleanly when finished to avoid
corrupting its disks.

You will need to switch back to the *main emulator window* you started
earlier - this cannot be done from the telnet session.

Execute the following, pressing Enter after each non Control-C line.

* Press Control-C
* Type `ek 40017`
* Type `st`
* Press Control-C
* Type `ek 0`
* Type `st`
* Press Control-C
* Type `ek 40032`
* Type `st`
* Type `q` and Enter to exit.

What is going on here? On the real IBM 7094 the operator would
initiate shutdown by toggling in a value on the front panel (`ek`, for
enter keys, in the emulator)

See [here](etc/shutting-down-ctss.txt) for a transcript of how this
looks.

## Dealing with disk errors

If you forget to do the above, or your machine crashes, or if there
are any problems starting CTSS again, all is not lost. Run this
command to do the equivalent of a `fsck` or `chkdsk`.

```
$ source env.sh
$ salvagectss
```

You will need to type `st` a couple of times to start the machine.
When the output says `QUIT`, type `q`.

## Changing the source code

This and the below sections are only of interest if you are changing
the ELIZA or SLIP source code.

You can make changes online using the `EDC` editor, but for anything
other than temporary experiments I recommend editing on your host
system and uploading the changed files to CTSS. You can also download
files from CTSS, but I would recommend this only for listing files,
not source files, as whitespace and sequence numbers may change which
will play havoc with source control. I describe each method below.

ELIZA source is in [`eliza/src/ELIZA`](eliza/src/ELIZA) and SLIP source is in
[`eliza/src/SLIP`](eliza/src/SLIP).

Very important - upload and download can only take place when CTSS is
shut down. You will soon be able to do the shutdown sequence from
memory.

### Editing online

`EDC` is a line orientated text editor for card image files similar to
`ed` or `edlin`. For more details, see my blog post at
[timereshared.com](https://timereshared.com/ctss-ed-and-friends/); it
is also described in sections AH.3.12 and AH.3.02 of the [CTSS
Programmer's
Guide](http://www.bitsavers.org/pdf/mit/ctss/CTSS_ProgrammersGuide_Dec69.pdf).
`EDC` has the same commands as the former `ED`.

### Uploading files

Use

```
$ upload user file
```

where `user` is either `ELIZA` or `SLIP` and `file` is the name of a
file on your local system.

As a convenience, you can type `upload-slip` or `upload-eliza` (with
no parameters) to upload all files intended for each account, or
`upload-all` to upload everything.

### Adding new files

If you create a new MAD, FAP, script tape or runcom file and it is in
one of the existing directories, it will be uploaded automatically by
the relevant `upload-` commands. If you want it to be automatically
compiled, also edit the relevant runcom file.

Files should follow the same column layout as existing ones. You don't
need to provide sequence numbers at the end of each line though.

### Downloading files via `extractctss`

`extractctss` is a utility that can be run on the host to download one
or more files. Usage:

```
$ extractctss file-type name1 name2 prob prog local-file
```

`file-type` should be `t` for card image files and `l` for listings.

A sample run to get the main MAD source file for ELIZA

```
$ extractctss t eliza mad m1416 eliza eliza.mad
```

This will produce `eliza.mad` in your current directory.

For the compiler output `ELIZA BCD`:

```
$ extractctss l eliza bcd m1416 eliza eliza.bcd
```

`extractctss` can also download several files in batch mode; see the
[ctss-kit README](ctss/ctss-kit-readme.txt) for more details

### Downloading files via `RQUEST`

On the real 7094, printing and card punching was not done online due
to the CPU load it would place on the system. Instead, users could
request files to be printed and punched. Several times a day, the CTSS
operators would collect the files on to a tape and bring this to a
smaller system for printing and punching, and then distribute the
results to users.

We can use this facility with the emulator as well.

1. While online, type `RQUEST PRINT name1 name2` to request printing
   of file `name1 name2` or `RQUEST DPUNCH name1 name2` to request
   punching. Wildcards are OK, so you could type `RQUEST PRINT * BCD`
   to produce a print out of all compile listings.
2. Shut down CTSS
3. Run `dskedtctss` on your host
4. You will need to press Enter a few times and then `q` when done.
5. Printer output will be in `output/sysprint.txt`. Each punch request
   will yield a separate file in `output/`.

## Development tips

* If you change any files, look through the runcom output to spot any
  errors when compiling. I've added comments in the runcom file to
  show what is expected.
* If a compile fails, it will *not* print `LENGTH xxx`. Look in the
  associated listing file (eg `ELIZA BCD`) for more details.
* FAP errors are cryptic 1 or 2 letter codes. See the [FAP
  manual](http://www.bitsavers.org/pdf/ibm/7090/C28-6235-2_7090_FAP.pdf)
  chapter 3 for an explanation. `TD` errors are warnings only and can
  usually be ignored (they represent partial instructions that will be
  completed at run time).
* If you get an error `s709: commopen: bind failed: Address already in
  use` when restarting CTSS, wait a few seconds and try again.

## Questions, bugs etc

Please raise an issue/pull request if you see any problems, or email
me at rupert@timereshared.com

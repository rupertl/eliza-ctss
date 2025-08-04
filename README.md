Compile and run Joseph Weizenbaum's original 1965 code for
[ELIZA](https://sites.google.com/view/elizaarchaeology/home) on
[CTSS](https://en.wikipedia.org/wiki/Compatible_Time-Sharing_System),
using the s709 IBM 7094 emulator.

* [elizagen.org blog
  post](https://sites.google.com/view/elizagen-org/blog/eliza-reanimated)
* [Video demo](https://youtu.be/j5Tw-XVcsRE)
* [Paper](http://arxiv.org/abs/2501.06707)

See the Prerequisites and Quickstart section below to begin.

Alternative ways to get ELIZA:

* [Automated setup](docker/DOCKER-README.md) using Docker.
* Add to an existing CTSS installation via a [virtual tape file](https://timereshared.com/files/ctss/eliza/).

Updating from an older version? See [`UPDATING.md`](UPDATING.md). Note
there is a new step in the Quickstart where you need to login as `sysdev`.

## Prerequisites

You will need a Unix like environment that can compile C code and run
Python and shell scripts. This has been tested on

* Arch Linux
* Debian
* MacOS

but may work on other Unix/Linux flavours.

You will also need the `telnet` command line program or a GUI telnet
client. The command line program can be installed by `sudo apt install
telnet` on Debian/Ubuntu or `brew install telnet` on MacOS.

## Quickstart

If you just want to try ELIZA, follow the below steps to bring up CTSS
and get ELIZA compiled and running. If you'd like to find out more
about each step, see [`HACKING.md`](HACKING.md).

1. Fetch this repo and set up the environment.

```
$ git clone https://github.com/rupertl/eliza-ctss.git
$ cd eliza-ctss
$ source env.sh
$ make-binaries
$ make-disks
```

2. The next two commands will prompt you to press Enter a number of
times and then `q` to quit.

```
$ format-disks
$ install-disk-loader
```

3. Continue installing CTSS and the ELIZA source.

```
$ installctss
$ add-eliza-users
$ upload-all
```

4. Start CTSS by typing `runctss`
5. Use another terminal window to start your telnet client and connect to localhost port 7094. For
the command line client you can type `telnet 0 7094`.
6. In the telnet session, type `login sysdev` and give the password `system`
7. The screen should now look something like

```
$ telnet 0 7094
Trying 0.0.0.0...
Connected to 0.
Escape character is '^]'.
s709 2.4.3 COMM tty0 (KSR-37) from 127.0.0.1

MIT8C0: 1 USER AT 02/01/25  921.5, MAX = 30
READY.

login sysdev
W 921.6
Password
 M1416     6 LOGGED IN  02/01/25  921.6 FROM 700000
 HOME FILE DIRECTORY IS M1416 SYSDEV

THIS IS A RECONSTRUCTED CTSS SYSTEM.
VERSION: 1.0.8
BUILT: 01/25/25 1053.0

 CTSS BEING USED IS: MIT8C0
R .033+.000

```
8. Type `runcom mkhuge` and wait for it to print `MKHUGE HAS BEEN RUN`.
9. Type `login slip` and give the password `slip`
10. Type `runcom make` and wait for it to print `MAKE HAS BEEN RUN`.
11. Type `login eliza` and give the password `eliza`
12. Type `runcom make` again and wait for `MAKE HAS BEEN RUN`.
13. Type `r eliza`
14. Give the answer `100` to the prompt of which [script](SCRIPTS.md) to use
15. ELIZA will print a greeting and you can now interact with her.
    Keep your responses below 72 characters and press Enter twice to
    submit. This is how your screen could look after a first response.

```
r eliza
W 1916.6
EXECUTION.
WHICH SCRIPT DO YOU WISH TO PLAY
100
HOW DO YOU DO . I AM THE DOCTOR . PLEASE SIT DOWN AT THE TYPEWRITER AND TELL ME
YOUR PROBLEM .
INPUT
Men are all alike.

DID YOU THINK THEY MIGHT NOT BE ALL ALIKE
INPUT
```

16. When you are finished, press `Control-\ ` (backslash) to interrupt and then
    type `logout`.
17. Shut down CTSS cleanly. You will need to switch back to the
    *main emulator window* you started earlier - this cannot be done 
    from the telnet session.

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

18. If you want to run ELIZA again, do `runctss` and then telnet in as
user `eliza` and `r eliza` like before. (You do not need to recompile it).

## Where to go next

If you would like to try different personality scripts for ELIZA, read
[`SCRIPTS.md`](SCRIPTS.md).

To learn more about compiling ELIZA, and how to make your own changes
to the soruce code, see [`HACKING.md`](HACKING.md).

If you would like to read more about the process of reconstructing the
code from the printouts, see
[`RECONSTRUCTION-CARD.md`](RECONSTRUCTION-CARD.md).

To find out more about CTSS, see my blog post series at
[timereshared.com](https://timereshared.com/ctss/).

## Known bugs and issues

The goal of this repository is to get ELIZA running in its original
form with as few modifications as possible. There are missing features
and bugs from the original code: see
[`KNOWN-ISSUES.md`](KNOWN-ISSUES.md) for details.

## Sources and licences

The ELIZA and SLIP library code under `eliza/` is from the source code
[printout](https://sites.google.com/view/elizagen-org/original-eliza)
which was found among Weizenbaum's papers at MIT by Jeff Shrager and
the MIT archivists, and
[transcribed](https://github.com/jeffshrager/elizagen.org/tree/master/1965_Weizenbaum_MAD-SLIP)
by Anthony Hay and Arthur Schwarz respectively. Weizenbaum's estate
has kindly agreed for this to be open sourced under a [Creative
Commons CC0 public domain
license](https://creativecommons.org/publicdomain/zero/1.0/). About 4%
of the SLIP library were missing in the original printout; these were
re-implemented by Anthony, Arthur and Rupert Lane and are found in
`eliza/src/SLIP/SLIP-reconstructed` with tests in `SLIP-tests`. This
newer work is also licenses as CC0.

The s709 emulator, ctss-kit and utilities under `ctss/` are from Dave
Pitts' [IBM 7090/7094 page](https://cozx.com/dpitts/ibm7090.html),
based on previous work on s709 by Paul Pierce, with minor
modifications by Rupert. This uses the MIT license, see
[LICENSE.txt](ctss/LICENSE.txt).

CTSS environment set up, upload and runcom scripts are by Rupert,
licensed as CC0.

The ELIZA script
[`eliza/src/ELIZA/tape.200`](eliza/src/ELIZA/tape.200) is adapted from
Weizenbaum's article for the _Communications of the ACM_ in 1966.

The [Docker automation](docker/DOCKER-README.md) was contributed by
[@MarquisdeGeek](https://github.com/MarquisdeGeek)

## Acknowledgements

Thanks to everyone on [Team
ELIZA](https://sites.google.com/view/elizaarchaeology/team?authuser=0)
for their help and advice,

## Questions, bugs etc

Please raise an issue/pull request if you see any problems, or email
me at rupert@timereshared.com

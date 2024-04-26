# ios23mole-logging-script

This is the Bash script for the first IOS project that logs the user's file modifications from 2023.

## 📚 Introduction

- `mole` – wrapper for efficient use of text editor with options
automatic selection of the most frequently or most recently modified file.

## 📜 USE
-   `mole -h`
-   `mole [-g GROUP] FILE`
-   `mole [-m] [FILTERS] [DIRECTORY]`
-   `mole list [FILTERS] [DIRECTORY]`
-   `mole secret-log [-b DATE] [-a DATE] [DIRECTORY1 [DIRECTORY2 [...]]]`

---

- `-h` – Prints help for using the script (option `secret-log` by
     it should not have been listed in the help; we don't want to alert the mole that we are collecting
     information).
- `mole [-g GROUP] FILE` – The specified file will be opened.
     - If the `-g` switch was specified, the given file opening will be
     also assigned to a group named `GROUP`. `GROUP` can be
     the name of both the existing and the new group.
- `mole [-m] [FILTERS] [DIRECTORY]` - If `DIRECTORY` matches
     existing directory, the script selects a file from that directory that
     should be opened.
   - If no directory is specified, the current directory is assumed.
   - If several files were edited by the script in the given directory,
       the file that was opened (edited) using the script is selected
       last**.
  - If the `-m` argument was specified, the script will select the file that
      was opened (edited) **most often** using the script.
        - If multiple files are found when using `-m` switch
            with the same maximum number of openings, `mole` can choose
            any of them.
   - File selection can be further influenced by specified `FILTERS` filters.
   - If none has yet been opened (edited) in the given directory
       the file, or no file matches the specified filters, acts
       with an error.
- `mole list [FILTERS] [DIRECTORY]` – The script displays a list of files,
   which were opened (edited) using a script in the given directory.
   - If no directory is specified, the current directory is assumed.
   - The list of files can be filtered using `FILTERS`.
   - The list of files will be sorted lexicographically and each file will be
       listed on a separate line.
   - Each line will have a format
       `FILENAME:<INDENT>GROUP_1,GROUP_2,...` where `FILENAME` is the name
       file (including its possible extensions), `<INDENT>` is the number
       spaces needed for alignment and `GROUP_*` are group names, u
       which the file is registered.

       - The list of groups will be sorted lexicographically.
       - If the groups are specified using the `-g` switch (see
           FILTERS section), consider only when listing files and groups
           records belonging to these groups.
       - If the file does not belong to any group, it will be in the list instead
           groups, only the `-` character is written.
       - The minimum number of spaces used for alignment (`INDENT`) is
           one. Each line will be aligned to list groups
           started in the same position. So for example:

           <!-- -->
  
              FILE1:  grp1,grp2
              FILE10: grp1,grp3
              FILE:   -

- `mole secret-log [-b DATE] [-a DATE] [DIRECTORY1 [DIRECTORY2 [...]]]`
   – In order to catch the mole, the script creates a secret compressed log with
   information about files opened (edited) through the script
   ``pier''.
   - If directories were specified, the secret log will contain records of
       opened (edited) files only from these directories.
       Non-existent directories or directories with no entries will
       ignored.
   - If no directory was specified, the secret log will contain
       records from all registered directories.
   - Opened (edited) files to be in the secret log
       recorded, it is possible to limit further using the `-a` and `-b` filters (see
       below).

## 📑 Filters

`FILTERS` can be a combination of the following filters (each can be specified
maximum once):

- `[-g GROUP1[,GROUP2[,...]]]` - Group specification. The file will be
     considered (for purposes of opening or listing) only if his
     startup falls into at least one of these groups.
- `[-a DATE]` - Records of opened (edited) files before
     they will not be considered by this date.
- `[-b DATE]` - Records of opened (edited) files after this
     date will not be considered.
- The `DATE` argument is in `YYYY-MM-DD` format.

## 🔧 Settings and configuration

- The script remembers information about its execution in a file that is
 given by the variable `MOLE_RC`. The file format is not specified.
     - If the variable is not set, it is an error.
     - If the file does not exist on the path given by the `MOLE_RC` variable,
         the file will be created including the path to the given file (if the
         does not exist).
- The script starts the editor, which is set in the `EDITOR` variable. If
     variable `EDITOR` is not set, respect variable `VISUAL`.
     If neither is set, the `vi` command is used.

## 💊 Secret log format

- The secret log generated using the `secret-log` command will be stored in
   the `.mole` directory located in the home directory (i.e. e.g.
   `/home/$USER/.mole/`). The file name will be in the format
   `log_USER_DATETIME.bz2`, where `USER` corresponds to the name of the current one
   user and `DATETIME` corresponds to the date and time the secret log was created.
   - The secret log will contain records of all known manipulations
       (i.e. opening through the `mole` script) with the selected files,
       possibly further limited to a given time period using switches
       `-a`, `-b`, or their combination.
   - The format of the log entries will be `FILEPATH;DATETIME_1;DATETIME_2;...`,
       where
       - `FILEPATH` is the real path to the file,
       - `DATETIME_N` is the date and time of the `N`th acquaintance chronologically
           opening a file either across all known history or in
           given time period.
   - Entries in the secret log will be sorted lexicographically by
       `FILEPATH` values.
- The format of `DATETIME` and `DATETIME_N` values is `YYYY-MM-DD_HH-mm-ss`.
- Compress the secret log using the `bzip2` utility.

## 📨 The return value

- The script returns success if the operation succeeds or if it succeeds
 editing. If the editor returns an error, the script returns the same error
 return code. An internal script error will be accompanied by an error message
 reporting.

## 📃 Examples of use

The following examples assume that the `mole` script is available in one
from the paths in the `PATH` variable.

1. Editing various files:

``` sourceCode
$ export MOLE_RC=$HOME/.config/molerc
$ date
Thu Feb 16 01:37:14 PM CET 2023
$ mole ~/.ssh/config
$ mole -g bash ~/.bashrc
$ mole ~/.local/bin/mole
$ mole -g bash ~/.bashrc                         # (D)
$ mole ~/.indent.pro
$ mole ~/.viminfo

$ date
Mon Feb 20 07:21:09 PM CET 2023
$ mole -g bash ~/.bash_history
$ mole -g git ~/.gitconfig
$ mole -g bash ~/.bash_profile                   # (C)
$ mole -g git ~/proj1/.git/info/exclude
$ mole ~/.ssh/known_hosts                        # (A)
$ mole -g git ~/proj1/.git/config
$ mole -g git ~/proj1/.git/COMMIT_EDITMSG
$ mole ~/proj1/.git/COMMIT_EDITMSG
$ mole -g git ~/proj1/.git/config                # (F)
$ mole -g project ~/proj1/main.c
$ mole -g project ~/proj1/struct.c
$ mole -g project ~/proj1/struct.h
$ mole -g project_readme ~/proj1/README.md

$ date
Fri Feb 24 03:52:34 PM CET 2023
$ mole -g git2 ~/.gitconfig
$ mole ~/proj1/main.c
$ mole ~/.bashrc                                 # (E)
$ mole ~/.indent.pro
$ mole ~/.vimrc                                  # (B)
```

1. Re-editor:

``` source Code
$ cd ~/.ssh
$ mole
... # spustí se editace souboru ~/.ssh/known_hosts (odpovídá řádku A)
$ mole ~
... # spustí se editace souboru ~/.vimrc (odpovídá řádku B)
$ mole -g bash ~
... # spustí se editace souboru ~/.bash_profile (odpovídá řádku C)
$ mole -g bash -b 2023-02-20 ~
... # spustí se editace souboru ~/.bashrc (odpovídá řádku D)
$ cd
$ mole -m
... # spustí se editace souboru ~/.bashrc (odpovídá řádku E)
$ mole -m -g git ~/proj1/.git
... # spustí se editace souboru ~/proj1/.git/config (odpovídá řádku F; ve skupině git byl daný soubor editován jako jediný dvakrát, zbytek souborů jednou)
$ mole -m -g tst
... # ! chyba, nebyl nalezen žádný soubor k otevření
$ mole -a 2023-02-16 -b 2023-02-20
... # ! chyba, nebyl nalezen žádný soubor k otevření
```

2. Display a list of edited files

``` sourceCode
$ mole list $HOME
.bash_history: bash
.bash_profile: bash
.bashrc:       bash
.gitconfig:    git,git2
.indent.pro:   -
.viminfo:      -
.vimrc:        -
$ mole list -g bash $HOME
.bash_history: bash
.bash_profile: bash
.bashrc:       bash
$ mole list -g project,project_readme ~/proj1
main.c:    project
README.md: project_readme
struct.c:  project
struct.h:  project
$ mole list -b 2023-02-20 $HOME
.bashrc:     bash
.indent.pro: -
.viminfo:    -
$ mole list -a 2023-02-23 $HOME
.bashrc:     -
.gitconfig:  git2
.indent.pro: -
.vimrc:      -
$ mole list -a 2023-02-16 -b 2023-02-24 -g bash $HOME
.bash_history: bash
.bash_profile: bash
$ mole list -a 2023-02-20 -b 2023-02-24 $HOME
$ mole list -g grp1,grp2 $HOME
```

3. Creating a secret log:

``` sourceCode
$ date
Fri Feb 24 04:13:58 PM CET 2023
$ mole secret-log
$ bunzip2 -k --stdout /home/trusty/.mole/log_trusty_2023-02-24_16-14-01.bz2
/home/trusty/.bash_history;2023-02-20_19-21-13
/home/trusty/.bash_profile;2023-02-20_19-21-38
/home/trusty/.bashrc;2023-02-16_13-37-31;2023-02-16_13-38-02;2023-02-24_15-53-05
/home/trusty/.gitconfig;2023-02-20_19-21-22;2023-02-24_15-52-39
/home/trusty/.indent.pro;2023-02-16_13-38-34;2023-02-24_15-53-18
/home/trusty/.local/bin/mole;2023-02-16_13-37-46
/home/trusty/proj1/.git/COMMIT_EDITMSG;2023-02-20_19-25-12;2023-02-20_19-25-18
/home/trusty/proj1/.git/config;2023-02-20_19-24-59;2023-02-20_19-25-27
/home/trusty/proj1/.git/info/exclude;2023-02-20_19-22-04
/home/trusty/proj1/main.c;2023-02-20_19-25-51;2023-02-24_15-52-48
/home/trusty/proj1/README.md;2023-02-20_19-26-36
/home/trusty/proj1/struct.c;2023-02-20_19-26-03
/home/trusty/proj1/struct.h;2023-02-20_19-26-18
/home/trusty/.ssh/config;2023-02-16_13-37-19
/home/trusty/.ssh/known_hosts;2023-02-20_19-22-48
/home/trusty/.viminfo;2023-02-16_13-38-53
/home/trusty/.vimrc;2023-02-24_15-54-04
$ date
Fri Feb 24 04:15:13 PM CET 2023
$ mole secret-log -b 2023-02-22 ~/proj1 ~/.ssh
$ bunzip2 -k --stdout /home/trusty/.mole/log_trusty_2023-02-24_16-15-22.bz2
/home/trusty/proj1/main.c;2023-02-20_19-25-51
/home/trusty/proj1/README.md;2023-02-20_19-26-36
/home/trusty/proj1/struct.c;2023-02-20_19-26-03
/home/trusty/proj1/struct.h;2023-02-20_19-26-18
/home/trusty/.ssh/config;2023-02-16_13-37-19
/home/trusty/.ssh/known_hosts;2023-02-20_19-22-48
```

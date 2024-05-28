# Data Git

Data Git (``dgit``) is a light weight bash script for linux, aiming at automatically persisting output data and saving running environment for reproducibility.

## Description

The primary usage of ``dgit`` is through ``dgit log``:

```bash
dgit log <log_name> <bash_file> [<description>]
```

On running this command, the script ``<bash_file>`` will be started as if running ``bash <bash_file>``. The difference is that ``dgit`` will run in background and saves the following things:
- the command in ``<bash_file>``
- the standard output and standard error
- all the output files produced by the command
- the current working directory

These data can be referenced later by ``<log_name>``. Even if you modify some codes or overwrite some output files later, all records can be recovered with ease.

**Saving Output.** The basic idea of ``dgit`` is to monitor the directory with ``inotifywait`` to see what files are modified when the process is running. It summarizes all the files within the current directory that are written or modified by the command, and copies them for future reference. It also greps the contents of the standard output and standard error, and saves them in a file.

**Saving Working Directory.** ``dgit`` is designed to work with ``git``. It creates a temporary commit to save the working directory and resets back so that the commit is not visible to the user. If you are repeatedly trying small changes in the code, it can be troublesome to commit each time you run the code. ``dgit`` can do it for you! However, **remember that ``dgit`` can't recover files in ``.gitignore``.**

**Saving Commnad.** The reason why ``dgit`` only accept a bash file as input is that it encourages to save all the commands (including command line arguments) in a bash file. This file will be copied by ``dgit`` and can be safely added to ``.gitignore`` if it's not part of the project.

## Installation

```bash
git clone https://github.com/tonycaisy/dgit.git
cd dgit
./install
```
Then restart the terminal so that the ``dgit`` command is available.

## Usage

```bash
dgit init
```

This command initializes the current directory as a ``dgit`` repository. It creates a hidden directory ``.dgit`` to store all the logs. You must first run ``git init`` to initialize the directory as a git repository.

```bash
dgit log [options] <log_name> <bash_file> [<description>]
Options:
  -d <dir>: where to trace the outputs, default is the whole directory
```

This command runs the bash script ``<bash_file>`` and creates a log entry named ``<log_name>`` for future reference. When ``-d`` is specified, only the files within the directory will be traced. The ``<description>`` is an optional string describing the log entry.

```bash
dgit ls
```

This command lists all the log entries.

```bash
dgit stdout <log_name>
```

This command prints the standard output and standard error of the log entry.

```bash
dgit fout [-f] <log_name>
```

This command copies all the files produced by the log entry back to the current directory. If ``-f`` is set, it will overwrite the existing files without asking.

```bash
dgit command <log_name>
```

This command prints the command that was run in the log entry.

```bash
dgit checkout [-f] <log_name>
```

This command resets the current working directory to the state when the log entry was created. It will call ``git checkout`` to reset the working directory so you should save all the changes before running this command. It will also copy the bash file back. If ``-f`` is set, it will overwrite the existing bash file without asking.

```bash
dgit rm <log_name>
```

This command removes a log entry. It is **not** reversible.

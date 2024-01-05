# scp-open

Open and edit a file on a server connecting by ssh

## Purpose

The purpose of this script is to open a file located on a server
which can be reached by ssh and allow to edit the file, if necessary,
copying the edited file back to the original position.

It is useful in the cases where mounting a remote location by ssh
is not possible or allowed.

## Installation

Put the scp-open python script in Path and make sure it is
executable. Python 3 is needed (I used Python 3.10 to develop
the script).

The following Python libraries are required:
- loguru
- sh

The following command line tools are required:
- scp
- lsof
- xdg-open (as it is the default program which opens the file)

## Usage

To open a file, the basic syntax is just:
```
scp-open <remotefile>
```
where ``<remotefile>`` is the remote location as it would be specified
for scp, e.g. comprehensive of the server location, username, etc
and the path of the file.

To simplify things, consider creating an alias for the server connection
parameters in .ssh/config and use key authentication.

### Setting the program opening the file

By default, the file is opened using ``xdg-open``, i.e. it ultimately uses the
XDG settings in the local system to find out which program is used.

In order to use a specific program, the program name (or path to its executable)
can be stated after the file name, as additional parameter:
```
scp-open <remotefile> <program>
```

### Copying back the file after editing

If the local copy was modified, when the editor program is closed, the scp-open
tool will ask the user if to copy the file back to the original location.

If the ``--copy-back`` (short ``-c``) option is used, the file is copied back
to the server, without asking.

In order to increase safety, it is possible to create a copy of the original
file on the server, whenever the local copy is copied there, using the
option ``--backup`` (short ``-b``).

## Internals

The remote file is copied by scp to a local temporary file.
A MD5 checksum of the original file is computed.

The file is opeded using by xdg-open or a program specified as
optional argument. The processes of this program and its child processes
which use the temporary file are monitored using lsof.

When all processes which keep the file open are done,
the MD5 checksum is computed again. If it changed, then
the local temporary file is copied back to the original position
on the server, using scp.

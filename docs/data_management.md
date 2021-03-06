---
layout: page
title: "Data Management: File Operations"
tagline:
---

Data is managed using the `files` service of the Agave CLI. With many parallels in
Linux file commands (`ls, mv, cp, rm,` etc.), Agave `files-*` commands can be used to 
list files, upload and download data, remotely manage and organize data, and import
external data from the web.

Agave `files-*` commands act on Agave STORAGE `systems`. See the [systems basics](systems_basics.md)
documentation for more details.

This page is broken down into the following sections:

1. [Listing files and navigating](#listing-files-and-navigating)
2. [Uploading and downloading via the CLI](#uploading-and-downloading-via-the-cli)
3. [Other file operations](#other-file-operations)
4. [File or folder history](#file-or-folder-history)
5. [Further help](#further-help)

<br>
#### Listing files and navigating

To list the files available to you on a STORAGE system, use:
```
% files-list -S data-tacc-work-username -L /
drwx------  username  4096  13:07  .
```

The `-L` flag provides a long listing (similar to `ls -l` in Linux), and the
optional forward slash (`/`) at the end of the line indicates the root path -
your `$WORK` directory. From the listing above, we have no files or folders 
available to us yet.

To make a new folder, then list the contents of that folder, do:
```
% files-mkdir -S data-tacc-work-username -N sd2e-data /
Successfully created folder sd2e-data

% files-list -S data-tacc-work-username -L /
drwx------  username  4096  15:49  .
drwx------  username  4096  15:49  sd2e-data

% files-list -S data-tacc-work-username -L sd2e-data/
drwx------  username  4096  15:49  .
```

To remove a folder, use the `files-delete` command:
```
% files-delete -S data-tacc-work-username sd2e-data/ 
Successfully deleted sd2e-data/
```

WARNING: The `files-delete` command will delete folders with or without content.

*Tip: if you set `data-tacc-work-username` as your default system, then you 
do not need to provide it on the command line. More info [here](systems_basics.md).*

<br>
#### Uploading and downloading via the CLI

Files can be transferred from your local machine to the remote STORAGE system
using the `files-upload` command, and from the remote STORAGE system to your
local machine using the `files-get` command.

First, find or create a local file and upload it to the STORAGE system (recreate
the sd2e-data folder if you deleted it in the previous example):
```
% touch my_file.txt
% echo 'Hello, world!' > my_file.txt
% files-upload -S data-tacc-work-username -F my_file.txt sd2e-data/
Uploading my_file.txt...
######################################################################## 100.0%

% files-list -S data-tacc-work-username -L sd2e-data/
drwx------  username  4096  15:53  .
-rw-------  username  0     15:53  my_file.txt
```

Use the `files-copy` command to make a copy of the file on the remote STORAGE system:
```
% files-copy -S data-tacc-work-username -D sd2e-data/my_copy.txt sd2e-data/my_file.txt
Successfully copied sd2e-data/my_file.txt to sd2e-data/my_copy.txt

% files-list -S data-tacc-work-username -L sd2e-data/
drwx------  username  4096  15:57  .
-rw-------  username  0     15:57  my_copy.txt
-rw-------  username  0     15:53  my_file.txt
```

The `-D` flag specifies the destination of the copied file, including path and
new file name. Experienced Linux command line users may find the order of options 
given to be counterintuitive - first the name of the target is given, then the
name of the original file is given. This is a common convention for many of the
Agave CLI file commands. 

Then, download the result:
```
% files-get -S data-tacc-work-username sd2e-data/my_copy.txt
######################################################################## 100.0%

% ls
my_copy.txt  my_file.txt
```

<br>
#### Other file operations

Using the Agave CLI, files and folders can also be renamed, moved, and deleted remotely on
the STORAGE system. The syntax for these operations is very similar to the 
`files-copy` command syntax. Here are some common examples:
```
# Rename a file or folder in place
% files-rename -S data-tacc-work-username -N sd2e-data/renamed.txt sd2e-data/my_file.txt

# Make a subfolder in the sd2e-data/ folder
% files-mkdir -S data-tacc-work-username -N sd2e-data/subfolder sd2e-data/f

# Move a file into that subfolder
$ files-move -S data-tacc-work-username -D sd2e-data/subfolder/renamed.txt sd2e-data/renamed.txt

# Delete a file or a folder
% files-delete -S data-tacc-work-username sd2e-data/subfolder/renamed.txt
% files-delete -S data-tacc-work-username sd2e-data/subfolder/
```

Be cautious with `files-move` and `files-delete` commands. Just like a Linux
filesystem, files inadvertently deleted or written over are probably unrecoverable.


<br>
#### File or folder history

You can list the history of events for a specific file or folder. This will give
more descriptive information (when applicable) related to number of retries, permission
grants and revocations, reasons for failure, and hiccups that may have occurred in 
the transfer process.
```
% files-history -S data-tacc-work-username sd2e-data/my_copy.txt                  
File item copied from https://agave.designsafe-ci.org/files/v2/media/system/data-tacc-work-username//sd2e-data/my_file.txt
```

<br>
#### Further help

Reminder: at any time, you can issue an Agave command with the `-h` flag to
find more information on the function and usage of the command. Extensive Agave
documentation can be found [here](http://developer.agaveapi.co/).

---
Return to the [API Documentation Overview](../index.md)

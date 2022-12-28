# General Things To Know About CVS
**Author:** Pantelis Roditis
**Created:** 2006/08/14 05:08

# Introduction
Here is a basic tutorial for CVS for new users. If you are new to CVS these are some basic commands to help you.

# Basic Commands
  * **`cvs add`**: Add a new file or directory to the repository. After the
  add, a cvs commit must be performed. The files will be added in the
  repository the next time you run a "cvs commit".
  * **`cvs commit`**: Use  this command when you wish to `publish` your
  changes to other users.
  * **`cvs update`**: This command is executed when you wish to update your
  copies of source files from changes that other users have made to the source
  in the repository.
  * **`cvs checkout`**: Creates the private copy of the source. You can work
  with this copy without interfearing with the other users.
  * **`cvs import`**: Incorporates a set of updates from off-site into the
  repository.
  * **`cvs remove`**: remove files from source repository, pending a cvs commit
  on the same files.
  * **`cvs diff`**: show changes between files in working directory and source
  repository, or between 2 revisions in same repository.
  * **`cvs release`**: cancel a cvs checkout
  * **`cvs status`**: show the current status of files

# How to add a new user

In order to add a new user you need to modify two files, the **passwd** and the
**readers/writers** file. Both files can be found under the CVSROOT directory.

The first step is to add the user into the passwd file. The file looks something like:
```
user1:-p1MW3dBhIj3w:cvs
user2-pgh0UZHzz4ho::cvs
user3:-pRzNIIV2J0L6:cvs
```

Where, **user1** is the username of each user, **p1MW3dBhIj3w** is the
encrypted password of the user and **cvs** is the optional system username.

In order to add a new user simply copy a line and change the username and the
password in its encrypted form. In order to encrypt the password type:

```
encrypt -s SA -p
Enter string: type here the desired password
```

After adding the user to the passwd file, you also need to add him/her into the
readers or writers file (see more details about readers and writers file in the
sections that follow).

The readers/writers files look something like:

```
user1
user2
user3
```

So just simply type the user you desired to add in a new line.

# How to give write access to a user

In order to give write access to a user, simply add his/her name to the
**writers** file which can be found under CVSROOT directory. The writers file
looks something like:

```
user1
user2
user3
```

To add a new user into the writers file, simply add that user in a new line.
Save and exit.

# How to give read only access to a user

Similarly to above, but this time the name of the user is added in the file
**readers** which can be found under the CVSROOT directory. The readers file
looks something like:

```
user1
user2
user3
```

In order to add a new user to the readers file, simply add him/her under a new
line. Save and exit.

# Start a new project
In order to start a new project follow the following steps:


* Go to the top of the project tree:
```
cd /home/myproj
```

* Import the new project to cvs. start = releasetag
```
cvs import -m "Initial Import into CVS" myproj echothrust start
```

Where:
- `myproj` is the name under which you will check out the project,
- `echothrust` is the vendor tag that we will be using

> NOTE: `cvs` will ignore certain files based on some rules:
> * Empty directories will not be imported (you can fix this by simply creating a dummy file like \
>   `find . -type d -empty -exec touch {}/.dummy \;`
> * Certain filename and directory patters will not be imported. This can be fixed by adding `-I!`
>   after the import, so the command will look like \
>   `cvs import -I! -m "mycomment" myproj echothrust start`


# How to create new file/directory

* In order to create a new file into the cvs, firstly you need to create that file.
```
vi new_file
```

* In order to add that file into the repository, simply type
```
cvs add new_file
```

* The last step is to publish your file to the others. In order to commit the file type:
```
cvs commit new_file
```

* Now you are done. Another user can see the new_file when s/he will update the CVS.


The added files will be placed in the repository after a cvs commit will be
performed. If a file was removed and a cvs commit has not taken place yet, you
can retrieve that file using cvs add.


Similarly, in order to create a new **directory** in the repository firstly you
need to:

* Make a new directory
```
mkdir new_directory
```

* Then you need to add that directory in the repository, using
```
cvs add new_directory
```

**NOTE**: You cannot add an empty directory into the repository. You need to
create a file into the directory and then you can do:
```
cvs add new_directory
cvs add new_file
```

* In order to publish your new_directory to the other users, commit your work by using:
```
cvs commit
```

* The other users will be able to see the new directory once they update their CVS.
```
cvs update
```
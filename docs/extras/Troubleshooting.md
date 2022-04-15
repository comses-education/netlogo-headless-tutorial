# Docker `build` Troubleshooting Guide

This document contains a compilation of commonly obtained errors when running a `docker build` command.

<h3> Your <code>build</code> command is not correct</h3>
  
Error message:

`"docker build" requires exactly 1 argument.`

It means that your `build` command has some typing error. The most likely ones are that it has spaces at any side of the `=` signs or that it is lacking the last `.`.

<h3>Your image name contains upper case letters.</h3>

Error message:

`invalid argument "WolfSheepIm" for "-t, --tag" flag: invalid reference format: repository name must be lowercase`

Use only lower case letters to name the image.

<h3> Your Dockerfile is not in plain text format</h3>

Error message:

`Error response from daemon: Dockerfile parse error line 1: unknown instruction: {\RTF1\ANSI\ANSICPG1252\COCOARTF1671\COCOASUBRTF600`

You will need to convert the format of your file to plain text. If you are using TextEdit, open your Dockerfile, press <kbd>shift</kbd> + <kbd>command</kbd> + <kbd>T</kbd> and save it. Use Finder to make sure that the name of the file does not include any extensions.

<h3> The Dockerfile was saved under a different name</h3>

Error message:

`unable to prepare context: unable to evaluate symlinks in Dockerfile path: lstat ~/MyDocker/Dockerfile: no such file or directory`

Use Finder or File Explorer to check that the name of the file was correctly spelled (`Dockerfile`), and it does not end with any extensions. If your file's name looks like `Dockerfile.txt`, you have to edit its name to erase the `.txt` termination. If your file's name ends with a `.rtf`, go to the previous error to see how to correct it.

If you are not able to erase the extension from your `Dockerfile`'s name, you can also add a `-f` flag to your `build` command. With this flag you can tell Docker the name of your Dockerfile. In our example, and assuming we have a file named `Dockerfile.txt`, our build command would be `docker build --build-arg MODEL_NAME=WolfSheep.nlogo --build-arg NETLOGO_VERSION=6.1.1 -f Dockerfile.txt -t wolfsheepim .`. Note that it is necessary to have a Dockerfile in plain text format to be able to build a Docker image.

<h3> The name of your .nlogo file was mispelled</h3>

Error message:

```
Step 9/10 : RUN rmdir /home/netlogoim/results  && mv /home/src/$MODEL_NAME /home/src/NLModel.nlogo
 ---> Running in (12-digit number)
mv: cannot stat '/home/src/WolfSheeppWrongSpelling.nlogo': No such file or directory
The command '/bin/sh -c rmdir /home/netlogoim/results  && mv /home/src/$MODEL_NAME /home/src/NLModel.nlogo' returned a non-zero code: 1
```

This means that the name of the model you provided in the `build` command, i.e. the name after `--build-arg MODEL_NAME=`, is not identical to the name of your .nlogo file. Check the spelling and that it includes the `.nlogo` extension.

<h3> The name of your .nlogo file contains spaces</h3>

Error message:

```
Step 9/11 : RUN mv /home/src/$MODEL_NAME /home/src/NLModel.nlogo
 ---> Running in (12-digit number)
mv: target '/home/src/NLModel.nlogo' is not a directory
The command '/bin/sh -c mv /home/src/$MODEL_NAME /home/src/NLModel.nlogo' returned a non-zero code: 1
```

Even if you use quotes around the file name, sometimes Docker is unable to find the correct path. If you find this error, try deleting the spaces in your .nlogo file (or replacing them with dashes) and use this new name in the `pathtoNLfile` argument.

<h3> The name of the BehaviorSpace experiment is wrong</h3>
When trying to run a container from your image, you get an error message from NetLogo:

`Exception in thread "main" java.lang.IllegalArgumentException: Invalid run, specify experiment name or setup file`

Verify that the name of the experiment you are asking the container to run is correctly typed and that it exists in the .nlogo file that you used to build the image.

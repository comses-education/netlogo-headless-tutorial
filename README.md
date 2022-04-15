# Containerizing a NetLogo model

Welcome! This tutorial will walk you through the process of containerizing a NetLogo model. To learn more about the importance of containerizing models and the basic principles behind this tool, we invite you to read [our article](https://doi.org/10.18174/sesmo.18074). The objective of this tutorial is to provide you with a hands-on experience of the steps involved, without the need to download or run anything locally in your machine. Once completed, we expect you to have a basic understanding of what you need to do to containerize and run your NetLogo model using Docker. With this knowledge and following the instructions given in [`docs/Next Steps`](docs/Next%20Steps), you should be able to containerize your own mode in your local machine.

We have structured this repository following the [suggested project organization](https://swcarpentry.github.io/good-enough-practices-in-scientific-computing/#project-organization) by Software Carpentry. Feel free to implement this organization within your own project. In this tutorial you will containerize the "Wolf Sheep Predation" model from NetLogo's model library. We have modified it to create a mock BehaviorSpace experiment (named "experiment1") and included it in the `src` folder of this repository. Feel free to explore the contents of this repository before coming back to this page. However, we suggest to refrain from modifying them until instructed in the steps below. Keep in mind that the tutorial will only allow you to containerize the "Wolf Sheep Predation" model already included in the `src` folder and the containerization will fail if this file is modified.

### A note on the tutorial workflow

We broke down the process of building and running a Docker container into three steps. You will be able to complete these steps using GitHub's browser interface, but you can also choose to clone this repository to your local machine and push your changes from there. We recommend using GitHub's browser interface and opening this repository in a second tab; this way you will be able to revisit the instructions while you create or edit files. Every time you commit (or push, if working from your local machine) your changes, the repository will automatically check your progress on each of the three steps. For now, the steps are marked as incomplete. By making the changes indicated in the instructions below, you will be able to sequentially pass each of the three steps. You can make as many attempts as needed to successfully complete them. To check your progress, right-click on the `Containerization Steps` badge below, open the link in a new tab, and then click on the name of your latest commit. 

[![Tutorial](../../actions/workflows/steps.yml/badge.svg)](../../actions/workflows/steps.yml)

At this point, the badge is in a red, `failing`, state. It will stay this way until all three steps of the tutorial are successfully completed, at which point it will change to a green, `passing` state. However, keep in mind that, even when your submission has passed the three tests, it takes some minutes for the badge status to update to `passing`. Therefore, it is advisable to click on the badge and look for the results of the latest commit, instead of waiting until the badge status is updated. The badge will appear at the end of each step so it will make it easier to check your progress.

Once you successfully containerize this model and run the mock experiment, you will have completed this tutorial. We expect that this exercise will equip you with the basic tools needed to containerize any NetLogo model. After completing this tutorial, we invite you to follow the instructions given in [`docs/Next Steps`](docs/Next%20Steps) to download this repository and modify the steps described here to containerize your own model. We have included a list of common Docker error messages and how to solve them at [`docs/extras/Troubleshooting.md`](docs/extras/Troubleshooting.md). Let us know if you find any roadblocks or any other problem by creating an issue [here](https://github.com/comses-education/NetLogo-headless/issues)!

***

## Step 1: Creating a Dockerfile

The Dockerfile is the file that describes how to create the image of your model and what steps to execute when you run a container based on that image. Follow these instructions (preferably in a separate tab, which will allow you to revisit the instructions while working on your file) to create the Dockerfile for this model:

1. Copy the following code (see [`docs/extras/Dockerfile Sections.md`](docs/extras/Dockerfile%20Sections.md) for more details about the commands used in this Dockerfile):

```Dockerfile
FROM openjdk:8-jdk

ARG MODEL_PATH
ARG NETLOGO_VERSION
ARG NETLOGO_NAME=NetLogo-$NETLOGO_VERSION
ARG NETLOGO_URL=https://ccl.northwestern.edu/netlogo/$NETLOGO_VERSION/$NETLOGO_NAME-64.tgz

ENV LC_ALL=C.UTF-8 \
    LANG=C.UTF-8
    
RUN mkdir /home/netlogo \
 && wget $NETLOGO_URL \
 && tar xzf $NETLOGO_NAME-64.tgz -C /home/netlogo --strip-components=1 \
 && rm $NETLOGO_NAME-64.tgz
    
COPY . /home/

RUN mv /home/$MODEL_PATH /home/src/NLModel.nlogo

ENTRYPOINT ["/home/netlogo/netlogo-headless.sh", \
 "--model", "/home/src/NLModel.nlogo", \
 "--table", "/home/results/table-output.csv", \
 "--spreadsheet", "home/results/spreadsheet-output.csv"]
```

2. (Recommended) If you prefer to keep a window or tab with these instructions open, scroll to the top of the page, where you will find the title: `comses-education/containerizing-netlogo-(your username)`. Right-click on `containerizing-netlogo-(your username)` and open the link in a new window/tab. This will take you to this same landing page, which will allow you to revisit the instructions while creating the Dockerfile.

![Step1-2,3](docs/extras/images/S1-23.png)

3. Click on `Add file`, then on `Create new file`.
4. Paste the code you copied in step 1 into the text box.
5. Above this text box, you will see a small text field where you can name your file. Write the word "Dockerfile" (with a capital "D"). You should now be able to read the path `containerizing-netlogo-(your GitHub username)/Dockerfile`. Note that "Dockerfile" is not followed by any format extension (no ".txt" or ".yml", for example).

![Step1-5](docs/extras/images/S1-5.png)

6. Scroll to the bottom of the page and click on `Commit new file`.

![Step1-6](docs/extras/images/S1-6.png)

#### Did it work?

Right-click on the `Containerization Steps` badge below and open the link in a new tab. You should see a list of your commits, the bottom two being "GitHub Classroom Autograding Workflow" and "Update GitHub Classroom Autograding", which you can ignore. The first item of the list should be "Create Dockerfile", or your custom commit message if you decided to modify the default. Click on it. In the top panel, you will see some information about the automated testing and its status. If it says "In progress", wait until finishes and shows a "Failure" status.  Under `steps.yml` you will see the three steps. At this point, `Step 1` should have a green checkmark to its left, `Step 2` a red cross, and `Step 3` a gray slash. If this is the case, you have successfully completed Step 1 and can close that browser tab and continue to the next step. If you still see a red cross next to `Step 1`, scroll to the `Annotations` panel below the three steps to read the error messages and correct your work accordingly.

[![Tutorial](../../actions/workflows/steps.yml/badge.svg)](../../actions/workflows/steps.yml)

## Step 2: Building the image

Now we are going to create the virtual entity that will download NetLogo, contain your model, and be ready to execute any experiment that you choose (from the list of BehaviorSpace experiment(s) that you have saved in your NetLogo file, which in our example corresponds to the only option of "experiment1"). 

The `docker build` command asks Docker to find a `Dockerfile` in the current folder and follow the instructions found there to build a Docker image. For this exercise, we will include the `docker build` command within a file named `build_image.sh`. However, when you are building the image with your own model, you will be able to type the command in the terminal window of your machine (see the instructions in `Next Steps` for more details).

Follow these instructions (again, it will be more comfortable if you have a second tab with these contents open; see Step 1-2):

1. Copy the following line:

<code>docker build --build-arg MODEL_PATH=<ins>pathtoNLfile</ins> --build-arg NETLOGO_VERSION=<ins>NLversion</ins> -t <ins>imagename</ins> .</code>

2. Click on the file `build_image.sh` and then, at the top-right corner of the text box showing the contents of the file, click on the pencil ("Edit this file") button.

![Step2-2](docs/extras/images/S2-2.png)

3. Paste the command and replace the <code><ins>underlined</ins></code> arguments as directed:
* Replace <code><ins>pathtoNLfile</ins></code> with the _relative_ path to the NetLogo model file, including the .nlogo extension. Relative to the location of the Dockerfile, the NetLogo model file is located at the following path: `src/Wolf-Sheep-Predation.nlogo`. Therefore, for this exercise, this is the path you should use after `MODEL_PATH=`. Note that using spaces for the `.nlogo` file name can be problematic. When containerizing your own model, we recommend to edit your model file name so spaces are erased or replaced by dashes or underscores.
* Replace <code><ins>NLversion</ins></code> with the version of NetLogo that you are using to run your model (in our case, `6.2.2`). If, when containerizing your own model, you are not sure what NetLogo version you are using, you can check the name of your NetLogo application and copy the number that goes after the word 'NetLogo'.
* Replace <ins><code>imagename</ins></code> with the name you want to give to your image. It can be any name, containing only lowercase letters, digits, and separators like periods, underscores and dashes. For this exercise we suggest `wolfsheep`.
* Make sure you are _not_ leaving any spaces between the `=` sign and the file path or version number you inserted, and to include the `.` at the end of the command.
4. Scroll to the bottom of the page and commit your changes.

![Step2-4](docs/extras/images/S2-4.png)


#### Did it work?

Right-click on the `Containerization Steps` badge below, open the link in a new tab, and look for your new commit (it should be "Update build_image.sh" if you did not change the default commit message). Since building an image can take a couple of minutes, you may see an orange dot next to `Step 2` and the status of the test will be "In Progress". When the test completes, you should see green checkmarks next to `Step 1` and `Step 2`, while the only red cross remaining should correspond to `Step 3`. If you get a red cross next to `Step 2` instead, scroll to the `Annotations` panel below the three steps to read the error messages and correct your work accordingly. If you can only see a cryptic message like "Process completed with exit code 1", it means that you got an error that we did not contemplate. Click on the title of the message to read the error given by Docker. As a reminder, the [Troubleshooting](docs/extras/Troubleshooting.md) guide includes a list common Docker error messages and their causes.

[![Tutorial](../../actions/workflows/steps.yml/badge.svg)](../../actions/workflows/steps.yml)

## Step 3: Running an experiment

In this step you will run the container to open the model, run the specified BehaviorSpace experiment, obtain the output files specified in the last two lines of the `Dockerfile`, and transfer the results to a folder of your preference. This is all done through the use of the `docker run` command. Note that subsequent runs will overwrite the result files if the same folder is used twice to receive the results. 

As with the `build` command, we will run the `run` command from a file, named `run_experiment.sh`, but it can be also executed from a terminal window when building and running your own container in your machine.

To run the experiment, follow the instructions (again, you can open a new window/tab with this instructions to have them at hand):

1. Copy the following line:

<code>docker run -v <ins>path/to/your/results/folder</ins>:/home/results <ins>imagename</ins> --experiment <ins>experimentname</ins></code>

2. Click on the file `run_experiment.sh` and then, at the top-right corner of the text box showing the contents of the file, click on the pencil ("Edit this file") button.
3. Paste the command and replace the <code><ins>underlined</ins></code> arguments as directed:
* <code><ins>path/to/your/results/folder</ins></code> is replaced by the _absolute_ path to your results folder in your computer. If the folder does not exist, Docker will create it. For this exercise, the folder that receives the results is the following: `$GITHUB_WORKSPACE/results`. When running experiments in your own machine, note that the path for the receiving folder has to be _absolute_, meaning that it has to start with a `/` or a `~` (or a `C:` in Windows machines). For example, and absolute path would look like `~/Documents/MyProject/results/`.
* <code><ins>imagename</ins></code> is replaced by the name you gave to the image in Step 2 (in our case, `wolfsheep`).
* <code><ins>experimentname</ins></code> is replaced by the name of the experiment you want to run. Note that it has to match exactly with the name saved in the .nlogo file. In our example, we saved our mock experiment under the name `experiment1`.
4. Scroll to the bottom of the page and commit your changes.

#### Did it work?

Open the `Containerization Steps` badge below in a new tab and look for your new commit (it should be "Update run_experiment.sh" if you did not change the default commit message). When the tests are completed, you should see green checkmarks for the three steps. If you get a red cross instead, scroll down to read the error messages in the `Annotations` panel and correct your `run` command accordingly.

[![Tutorial](../../actions/workflows/steps.yml/badge.svg)](../../actions/workflows/steps.yml)

Once you get your three green checkmarks, you will be able to see two output files in the [`results`](results/) folder. This means that you successfully containerized the "Wolf Sheep Predation" model and ran an experiment using it. Congratulations!

## Final notes

You have completed the tutorial, meaning that you are ready to containerize your own model! Now you can follow the instructions presented at [`docs/Next Steps/`](docs/Next%20Steps). We also created a [video](https://youtu.be/ICYM-vXx_Q4) showing how the containerization process would look like in a local computer. As a summary, it all boils down to three basic steps: create a Dockerfile, build the image, and run the experiment. 

  

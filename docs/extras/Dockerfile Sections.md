# Sections of the Dockerfile
In case you are willing to delve deeper into the commands given in the Dockerfile, we have included a brief explanation. 

### `FROM`
Dockerfiles start with a `FROM`, followed by a base image. Since we are not going to create everything from scratch, we need to establish what image we are going to use to build our new image from. In this case, we are going to use an image of OpenJDK. Therefore, our Dockerfile starts with `FROM openjdk:8-jdk`.

### `ARG` and `ENV`
ARGuments are pieces of information that may be given to Docker when the `build` command is executed, but can also be set to some default values. In our case, we are leaving the first two arguments (`MODEL_PATH` and `NETLOGO_VERSION`) open for modifications in the `build` command because these are expected to be modified by each user, while the second two (`NETLOGO_NAME` and `NETLOGO_URL`) were assigned default values. You could also overwrite the second two arguments in you `build` command by adding to it `--build-arg <ARG>=<new value for this ARG>`, although there is no need to do that in our case. 

While ARGuments are only stored at the time the image is built and then discarded, ENVironment variables are kept in the image. In our case, we are using `ENV` to set the coding to a standard format, to avoid any incompatibilities.

### `RUN` and `COPY`

In this section we are telling Docker the steps it has to follow to build the image. In short, we are asking it to create some folders and to download and unzip our specified NetLogo version. Afterwards, we are asking it to copy our NetLogo model to a folder in the image. In other words, this is the step in which our model is immortalized in the image we built.

### `ENTRYPOINT`

The final step is to tell the image what to do when we use it to run a container. We use `ENTRYPOINT` to tell it to run `netlogo-headless.sh`, with the arguments `model`, `table`, and `spreadsheet` as specified. When we use the `run` command, we are specifying one extra argument, `experiment`, to be the name of one of the available BehaviorSpace experiments. This way, different experiments can be run in the same model image.

If you are not interested in one type of output from your experiments (notice this will modify the outputs of all the experiments that you run using that image), you can delete the line corresponding to it. For example, if you only wanted to get results in the table format and not the spreadsheet, then your `ENTRYPOINT` section would look like the following:

```
ENTRYPOINT ["/home/netlogoim/netlogo-headless.sh", \
 "--model", "/home/netlogoim/ModInImage.nlogo", \
 "--table", "/home/netlogoim/results/table-output.csv"]
 ```
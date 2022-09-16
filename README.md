# Getting started

This quick reference guide shows how to leveraging this repo to customize the [source-to-image](https://github.com/openshift/source-to-image) jupyter notebook builder image for OpenDatahub JupyterHub. This branch creates a fedora based s2i builder image for the jupyter container that takes the [textbook content](https://github.com/jappavoo/UndertheCovers) and generate a new image which can be deployed in ODH environment. 

>**_Note:_** This container is only a template and does not contain the textbook content.

## Anatomy of the branch

We have the base directory that includes files like requirements, and other configurations required during the build step. The layout of this branch looks like:<br/>
<br/><img src="folder_structure.png" alt="folder_structure" width="400" height="350" /><br/>

- base: 
    * To configure base image: **<base_registry>/<base_image>:<base_tag>**
    <br/>[base_registry](.base/base_registry)
    <br/>[base_image](.base/base_image)
    <br/>[base_tag](.base/base_tag)
   * To specify system and python libraries:
    <br/>[python_prereqs](.base/python_prereqs)
    <br/>[python_pkgs](.base/python_pkgs)
    <br/>[jupyter_enable_etxs](.base/jupyter_enable_etxs)

- makefile -defines the set of commands/targets to run against the base image, described in the user guide.

## Customizing this repo
This OPE repo acts like as generic template for creating customized jupyter notebook images that can be deployed in an OpenShift managed cloud environment.
<br/><br/> &nbsp;&nbsp;  To customize and author materials, one can clone the OPE/container branch and can from there create more branches based on the local repo for each<br/> environment like development, testing, production etc. 
<br/> <br/><img src='repo_lineage.png' width=550 height=350>

## Docker build process

The container template is created over two stages to achieve better organization and to reduce size. 

**Stage one:**
- Add libraries required to install/build development dependencies

**Stage two:**
- Copy the final binary folders from stage-one.
- Include artifacts(executables, configs) from local
- Enable jupyter extensions
- Foler permissions


The final image created after second stage contains only the required binaries, the process looks like :<br/>
<br/><img src="docker_build.png" alt="folder_structure" width="600" height="350" /><br/>

## User Guide

### Makefile - targets
This repo utilizes make tool to efficiently build, run, tag and publish the jupyter notebook s2i builder image. 
- build - builds the custom s2i builder image
- push - push current build to private registry
- publish - push the current private build to public registry
- root - executes the private image as root user
- ope - executes the private image with root shell
- nb - starts published version with jupyter notebook interface
- lab - starts published version with jupyter lab interface
### Adding python packages
&nbsp;S2i framework uses [micropipenv](https://github.com/thoth-station/micropipenv) a pip wrapper tool to manage the python library installation. The requirements file generated using micropipenv should be updated in the repository to include additional python libraries. 

<br/> Follow the steps to generate requirements.txt using micropipenv [here](https://github.com/AbiShanna/Ope-Documentation/tree/main/micropipenv).

### Adding system libraries
[Distribution packages](.base/distro_pkgs) listed here installs the system libraries during the build time.
### Jupyter extensions to be enabled/disabled
The list of jupyter extensions to be enabled can be updated in the [enable extensions](.base/jupyter_enable_exts) file.
</br> The list of jupyter extensions to be disabled can be updated in the [disable extensions](.base/jupyter_disable_exts) file.

### Launching in desktop
To run the published public image in your local workstation that has docker running, 
- Clone the git repository
- Checkout branch fedora/ubuntu based on the requirement
- In the parent directory execute 
     * 'make nb' to launch the classic jupyter notebook interface
     * 'make lab' to launch the jupyter lab interface



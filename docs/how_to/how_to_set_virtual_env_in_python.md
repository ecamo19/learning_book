---
title: "How to set a virtual enviroment for Python"
author: "ECM"
date: "2021"
output: 
 prettydoc::html_pretty:
    highlight: pygments
    theme: cayman   
    toc: yes
    number_sections: True
    toc_depth: 2
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE, comment = "", fig.align = 'center',
					  fig.width = 16, fig.height = 10, eval = F)
```


All the code was gather from this [video](https://www.codecademy.com/paths/computer-science/tracks/cspath-cs-101/modules/cspath-modules/articles/learn-python-pipenv)

# Check pip version

```{bash}
pip --version
```


# Install pipenv at the user level

```{bash}
pip install --user pipenv
```

__If you see a warning saying: ' bla bla bla which is not in path ' you have to edit the bash profile__

```{bash eval = FALSE}
# Open bash_profile
nano ~/.bash_profile

# Add to the profile 
export PATH="add path that appears in the error:$PATH"

restart terminal
```

# Check that pipenv is install

```{bash}
pipenv --version
```


# Set-up virtual enviroment for the first time

## Set up last python version 

```{bash eval = FALSE}
# Initialize python virtual env
# Three means the python version, if you work with python 2 change to to two  
# Run this in the folder the you are working 
# Should create a file called Pipfile

pipenv --three
```

## Set up an old python version

```{bash eval = FALSE}

# Check all the python versions available in my computer
ls /usr/bin/python*

# Install the version wanted
pipenv install --python 3.7 
```


# Daily workflow

+ __The steps from above are done to set up the pipenv for the first time.__

+ __The following steps are done every single time I work in the project__

## Open a shell (terminal) inside a virtual environment
```{bash eval = FALSE}
pipenv shell

# Exit the shell
exit
```

```{bash eval = FALSE}
# Open python inside the virtual environment
python3

# Check package version
print(numpy.__version__)
```

## Install packages in pipenv

+ __Remember to run this command inside the pipenv shell__ 

+ __Make sure that the package appears in the PIPfile__

```{bash eval = FALSE}
# This install any version of numpy aka the latest version
pipenv install _package_
```

## Install a specific version of the package in pipenv
```{bash eval = FALSE}
# This install a specific version of a package 
pipenv install _package_ == 2.18.1 
```

## Create requirement.txt file

```{bash}
pipenv lock -r > requirements.txt
```

## Removing a virtualenv
```{bash}
test_project pipenv --rm
```
 
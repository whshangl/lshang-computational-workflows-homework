# Homework

In this homework you will tie together the three separate technologies discussed during
the course:

2. Setting up a git version control repository.
3. Using Docker and the `Dockerfile` to specify an image.
1. Unit testing using Python and the `pytest` package.

with a new technology, the use of the CircleCI continuous integration
service (https://circleci.com) to continously test the software
in your git version control repository on every new commit pushed.

*NOTE: The instructions in this document are not complete. You will need to
refer to the notes in the file `README_instructor.md` in this repository
to construct the necessary commands.*

## Deliverable

By email to jack.hale@uni.lu, Subject: Computational Workflows I homework.

1. Link to the github repository on github.com.
2. Your repository should contain a modified version of this file `homework.md`
   file containing the specific commands that you used to complete the
   exercises. You do not have to include everything, just the key commands.

     ```
     # Add your commands here
     ```

## High level overview

At the end of this homework, you will have a computational workflow consisting of:

1. A git repository hosted on https://github.com.

1. A `Dockerfile` with an environment containing `python` and the Python
   module `pytest`.

2. A Docker image built from the `Dockerfile` and uploaded to the Dockerhub.

3. A simple Python function and a set of unit tests.

4. An automatic trigger setup between github.com and CircleCI; when you
   make a new commit to your github repository, the Python tests will
   run inside a Docker container using the image that we uploaded to the Dockerhub.
   The result will be reported back to github.

This basic setup is the starting point for ensuring quality sofware engineering
in the computational sciences.

## Setting up a git version control repository

1. Go to https://github.com. Click on the plus icon in the top right hand
   corner of the screen. Then click on New Repository.

2. Create a public repository called e.g.
   'jhale/computational-workflows-homework'. Make sure to check the 'Initialize
   this repository with a README' checkbox. Click the 'Create Repository'
   button.

3. Click the Clone or Download button and copy the text. In a terminal run, e.g.:

     git clone git@github.com:jhale/computational-workflows-homework.git

   substituting with the correct location of your repository.

## Dockerfile

1. Create a file `docker/Dockerfile` in the repository containing the following text.

```
FROM ubuntu:18.04

RUN apt-get update && \
    apt-get install -y python3 python3-ipython python3-pytest python3-numpy && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
```

```
# mkdir docker
# cd docker
# vim Dockerfile
```

2. `git add` and `git push` the file `docker/Dockerfile` to the repository.

```
# git add docker/Dockerfile
# git commit -m "add Dockerfile by terminal"
# git pull
# git push -u origin master
```

## Build and push Docker image

1. `docker build`, `docker login` and `docker push` the image described by the
   `Dockerfile` to the Dockerhub. Tag your Docker image
   `<yourdockerhubusername>/computational-workflows`.

```
# cd docker
# docker build .
# docker images
# docker tag 7006816dea11 whshangl/computational-workflows
# docker images
# docker push whshangl/computational-workflows
```

## Run a container, and share in files from the host.

1. `docker run` the image. Use the `-v` flag to share the root of the git
   repository to `/root/shared` inside the container. Use the `-ti` flag to get
   an interactive prompt inside the running container.

```
# docker run -ti -v $(pwd):/root/shared whshangl/computational-workflows
```

## Setup a simple Python test suite

1. In a terminal running on the host (outside the container), copy across the
   files ``python/unit_testing/wallet/wallet.py`` and
   ``python/unit_testing/test_wallet.py`` to the root of your homework repository.
   ``git add``, ``git commit`` and ``git push`` them.

```
# git status
# git add wallet.py
# git commit -m "add wallet.py"
# git push
# git status
# git add test_wallet.py
# git commit -m "add test_wallet.py"
# git push

```

2. Run the tests inside the container by going to `/root/shared` and running the
   command `py.test-3`. The tests should fail.

3. Modify ``wallet.py`` until the tests in ``test_wallet.py`` all pass.

4. ``git add``, ``git commit`` and ``git push`` the working ``wallet.py`` file.

## Setup CircleCI to run tests

1. Sign up for an account on CircleCI (https://circleci.com). It is important
   that you use the option to get an account using your existing Github login
   as this will link the two accounts.

1. First of all we will write a file ``.circleci/config.yml``. This file will
   contain all of the instructions necessary for CircleCI to test your project.
   As usual, ``git add``, ``git commit`` and ``git push`` it to your repository.
   CircleCI is almost infinitely customisable. Check out the documentation for more
   details (https://circleci.com/docs/2.0/configuration-reference/#section=configuration).

```
version: 2.1

jobs:
  build:
    docker:
      - image: jhale/computational-workflows # e.g. the primary container, where your job's commands are run
    steps:
      - checkout # check out the code from github in the project directory
      - run: py.test-3 # run py.test-3 inside the container
```

1. Go to https://circleci.com and login to your account.
2. Click on the 'Add Project` button on the left hand side.
3. Find the the Github repository e.g.
   `jhale/computational-workflows-homework`. Click 'Set Up Project'.
4. Click 'Start building' in the 'Next Steps' section.
5. CircleCI will:
     * Checkout your git repository from github.
     * Pull the Docker image from the Dockerhub.
     * Launch a container running your Docker image.
     * Run the command `py.test-3` inside the image.
     * If the commands runs successfully, it will report 
       the success back to github.com. Otherwise, it will
       report a failure.
6. Refresh your repository home page on github.com. You 
   should see a small green tick next to the last commit.
   Click the green tick, then click Details. You will
   be taken to circleci.com, where you should see a summary
   of the continuous integration tests.

## Exercises

1. Write a new test in ``test_wallet.py`` that fails. Push the
   file to github. What do you see on github.com?
   
   I saw a red cross next to the last commit on github.com. What's more, I received an email telling me that my workflow  did not succeed.

2. Fix the test. What do you see on github.com?

   I saw a small green tick next to the last commit again.

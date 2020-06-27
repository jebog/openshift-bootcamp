# Building a Customer Docker Image

In this exercise, you'll create a simple Python application image from source code built locally.

Use the resoures in the [python-image](resources/python-image) directory.

If you have cloned the entire `openshift-bootcamp` Git repo, change directory to `/home/ibmdemo/awb-bootcamp/openshift-bootcamp/Labs/Lab01-Building-a-container/resources/python-image`. If not, then visit the Git repository in the URL and copy the file contents directly into a new directory.

To build the image, run the following

```
$ docker build -t my-python .
```

The docker image should start building, and it will be available in the list of docker images

```
$ docker images | grep my-python
```
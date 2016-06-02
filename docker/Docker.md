# Anafora Docker Container Guide
This docker container runs a fully functional instance of Anafora. The image was built on an Ubuntu
16.04 base and uses Apache 2 as the web server.

## Pre-requisites
1. Install Docker (see https://docs.docker.com/engine/installation/)

## Configuration
The container only contains a sample authentication htdigest file with a single user named "anafora"
with password "anafora". In order to create your own users you will want to create an Apache
htdigest file on the host system and add in the users that you want. The following command will
create the user "myuser" in a new file named anafora.htdigest
```
htdigest -c anafora.htdigest anafora myuser
```
the htdigest program will prompt you for the password to set. You only want to use the `-c` argument
for the first user since it will wipe out an existing file.

You will then mount this file onto the
container at `/home/anafora/anafora-1.1/anafora.htdigest` when you run it.

## Operation
### Run with the sample project
The Anafora container has the sample project installed, so in order to run the container with the sample project simply run the container with
```
docker run --name anafora -p 80:80 dkincaid/anafora:1.1
```
### Run with a new project
The Anafora Docker container has the project
directory is set to /home/anafora/project_root. In order to use your own project do the following:

1. Create your .setting.xml, .schema and all the other project files needed according to the Anafora
   administrators guide somewhere on the host filesystem (e.g. maybe in
   `/home/myuser/anafora-project-root`). You'll specify this directory when you run the container below.
2. Set the owner and group of the project files on the host to uid 1000 and gid 33 to insure that
   Apache can write to them.
2. Set the permissions on the Corpus files to 0660
2. Run the Docker container using 
```
docker run --name anafora -p 80:80 -v \
   /home/myuser/anafora-project-root:/home/anafora/project_root dkincaid/anafora:1.1 \
   -v /home/myuser/anafora.htdigest:/home/anafora/anafora-1.1/anafora.htdigest
```

This will mount the host directory `/home/myuser/anafora-project-root` as
`/home/anafora/project_root` in the Anafora container which will make it the active project directory for
the Anafora web app.

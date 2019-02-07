# docker-various-examples

A repository with some interesting examples of cool docker things

# Generate a Java project, build and run it

First go to the java directory in this project:

`cd java`

Now build the docker image:

`docker build . -t maven-build`

(not sure why I am using maven-build as a name but too lazy to go change it now)

The Dockerfile uses dockers [multistage build](https://docs.docker.com/develop/develop-images/multistage-build/) feature to pull a generated project from [start.spring.io](https://start.spring.io/), unpacks it and then copies the code into anther container that has maven and java installed. The project will then build and be run.

Now you can run the container and mount a volume to edit the code on your machine.

Let's say you want to store your code in the java project of this repo under another folder called app.

From the java directory:

`mkdir -p app && cd "$_"`

On Windows: 

`mkdir app`
`cd app`

Create a named volume for the current directory (using mavendev here but feel free to call it something else):

`docker volume create -d local -o type=none -o o=bind -o device=$(pwd) mavendev`

Windows! One annoying thing is $(pwd) on Windows returns a path with Widows backslashes. You need forward slashes for volumes so on Windows you need to do this:

`docker volume create -d local -o type=none -o o=bind -o device=%CD:\=/% mavendev`

Take a second to look at the volume you just created:

```
$ docker volume ls
DRIVER              VOLUME NAME
local               mavendev
```

You can inspect it to make sure you are happy with the directory setup. Device is the directory we want to bind into our container soon. Basically we want to share a directory between our machine and the container. Make sure this directory is empty on your machine.

```
$ docker volume inspect mavendev
[
  {
    "CreatedAt": "2019-02-06T19:18:44Z",
    "Driver": "local",
    "Labels": {},
    "Mountpoint": "/var/lib/docker/volumes/mavendev/_data",
    "Name": "mavendev",
    "Options": {
      "device": "/Users/ruairi/Dev/docker-various-examples/java/app",
      "o": "bind",
      "type": "none"
  },
  "Scope": "local"
}
]
```

If you're on Windows it would look something like this:

```
[
  {
    "CreatedAt": "2019-02-07T07:27:11Z",
    "Driver": "local",
    "Labels": {},
    "Mountpoint": "/var/lib/docker/volumes/mavendev/_data",
    "Name": "mavendev",
    "Options": {
        "device": "/c/Users/ruairi/Dev/docker-various-examples/java/app",
        "o": "bind",
        "type": "none"
    },
    "Scope": "local"
  }
]
```

Now we can run the container and use the volume we created:

`docker run --rm -it -v mavendev:/usr/app -p 8080:8080 maven-build sh`

This opens a shell in the container.
You can run the app with:

`./mvnw spring-boot:run`

You should now be able to access the running application at <http://localhost:8080>

You will just see a Whitelabel Error Page since we haven't setup any routes yet but that still shows the app is running and accessible.

Notice the java code from the docker image is now in java/app.

**Next steps**

Check out [this post on using docker for local development](http://www.realgorithm.io/2018/04/simple-dev-environment-with-docker-compose/)

[Play with docker](https://labs.play-with-docker.com/)

# Generate a React project, build and run it

First go to the javascript directory in this repository:

`cd javascript`

Build the docker image:

`docker build . -t react-build`

This will generate a react app for you.

You may have to ctrl+c to exit after generation is complete.

Let's create a directory for our app code.

From the javascript directory:

`mkdir -p app && cd "$_"`

On Windows: 

`mkdir app`
`cd app`


Create a named volume for the current directory (using reactdev here but feel free to call it something else):

`docker volume create -d local -o type=none -o o=bind -o device=$(pwd) reactdev`

On Windows do it like this:

`docker volume create -d local -o type=none -o o=bind -o device=%CD:\=/% reactdev`

Take a second to look at the volume you just created:

```
$ docker volume ls
DRIVER              VOLUME NAME
local               reactdev
```

You can inspect it to make sure you are happy with the directory setup. Device is the directory we want to bind into our container soon. Basically we want to share a directory between our machine and the container. Make sure this directory is empty on your machine.

```
$ docker volume inspect reactdev
[
{
"CreatedAt": "2019-02-06T19:22:50Z",
"Driver": "local",
"Labels": {},
"Mountpoint": "/var/lib/docker/volumes/reactdev/_data",
"Name": "mavendev",
"Options": {
"device": "/Users/ruairi/Dev/docker-various-examples/react/app",
"o": "bind",
"type": "none"
},
"Scope": "local"
}
]
```

If you're on Windows it would look something like this:

```
[
  {
    "CreatedAt": "2019-02-07T07:27:11Z",
    "Driver": "local",
    "Labels": {},
    "Mountpoint": "/var/lib/docker/volumes/mavendev/_data",
    "Name": "mavendev",
    "Options": {
        "device": "/c/Users/ruairi/Dev/docker-various-examples/javascript/app",
        "o": "bind",
        "type": "none"
    },
    "Scope": "local"
  }
]
```

Now we can run the container and use the volume we created:

`docker run --rm -it -v reactdev:/usr/app/my-app -w /usr/app/my-app -p 3000:3000 react-build sh`

Now you can run `yarn start`

And access the app on <http://localhost:3000>

# Cleanup

You probably don't want to leave all the stuff we just created hanging aroound on yoour achne if you're not usign it.

Here's how to clean it all up.

### Containers

First stop and remove the containers we created.

To see them we use [docker ps](https://docs.docker.com/engine/reference/commandline/ps/):

```
docker ps -a -q
```

If you stopeed the containers you need to use `-a` to show stopped containers. Because we only need the ID we pass `-q`.

Now you can copy the IDs of the container you want to remove and run:

`docker rm <the-ids-space-seperated>`

If the container is running, you would first haev to do:

`docker stop <the-ids-space-seperated>`

If you just want to get rid of all containers you can do this:

```
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
```

### Images

You can see what images we created using [docker image ls](https://docs.docker.com/engine/reference/commandline/image_ls/) (`docker images` works too).

The command to remove and image is: `docker rmi <image-id>` 

It's fine to leave the images there but if you'd like to free up some space you can delete them.


### Volumes

List your volumes: `docker volume ls`

Delete a volume `docker volume rm <volume-name>`

Clean up all unused volumes: `docker volume prune`


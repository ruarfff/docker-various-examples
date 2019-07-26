# docker-various-examples

A repository with some interesting examples of cool docker things

## Generate a Java project, build and run it

First go to the java directory in this project:

`cd java`

Now build the docker image:

`docker build . -t java-build`

The Dockerfile uses dockers [multistage build](https://docs.docker.com/develop/develop-images/multistage-build/) feature to pull a generated project from [start.spring.io](https://start.spring.io/), unpacks it and then copies the code into another container that has java installed. The project will then build.

Now you have a docker image from which you can create a container that will have a java project built with gradle in it. 
You can use this container as a development environment if you wish. You could run the container, exec into it and work away there. You can also mount a volume from the container to a directory on your host machine. We will look at both options.

### Work with docker exec

Earlier we built a docker image and gave it the name `java-build`. We can now create a container from that image and leave it running as a process like so:

`docker run -dit -p 8080:8080 --name java-dev java-build sh`

The container is now running and has been created from the java-build image. We gave it the name `java-dev`. We configured it to run as a daemon and keep the shell process running. We need this to be able to interact with the container. We also exposed (published) port 8080. We can use this to access any web app we run in the container.

You can check that the container is running with this:

`docker ps`

Now let's exec into the container. This is kind of analogous to ssh-ing into a machine somewhere.

`docker exec -it java-dev sh`

Now you should be in the container. You can look at files and run some command e.g.

```bash
/usr/app # ls -al
total 52
drwxr-xr-x    1 root     root          4096 Jul 26 09:55 .
drwxr-xr-x    1 root     root          4096 Jul 24 17:29 ..
-rw-r--r--    1 root     root           341 Jul 26 09:54 .gitignore
drwxr-xr-x    5 root     root          4096 Jul 26 09:55 .gradle
-rw-r--r--    1 root     root           938 Jul 26 09:54 HELP.md
drwxr-xr-x    9 root     root          4096 Jul 26 09:55 build
-rw-r--r--    1 root     root           551 Jul 26 09:54 build.gradle
drwxr-xr-x    3 root     root          4096 Jul 26 09:54 gradle
-rwxr-xr-x    1 root     root          5305 Jul 26 09:54 gradlew
-rw-r--r--    1 root     root          2269 Jul 26 09:54 gradlew.bat
-rw-r--r--    1 root     root            26 Jul 26 09:54 settings.gradle
drwxr-xr-x    4 root     root          4096 Jul 26 09:54 src
````

```bash
/usr/app # ./gradlew bootRun
Starting a Gradle Daemon, 1 incompatible and 1 stopped Daemons could not be reused, use --status for details
> Task :compileJava UP-TO-DATE
> Task :processResources UP-TO-DATE
> Task :classes UP-TO-DATE

> Task :bootRun

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.2.RELEASE)

2019-07-26 10:08:11.891  INFO 92 --- [  restartedMain] com.example.demo.DemoApplication         : Starting DemoApplication on 865a1d047f0a with PID 92 (/usr/app/build/classes/java/main started by root in /usr/app)
2019-07-26 10:08:11.893  INFO 92 --- [  restartedMain] com.example.demo.DemoApplication         : No active profile set, falling back to default profiles: default
2019-07-26 10:08:12.006  INFO 92 --- [  restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : Devtools property defaults active! Set 'spring.devtools.add-properties' to 'false' to disable
2019-07-26 10:08:12.006  INFO 92 --- [  restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : For additional web related logging consider setting the 'logging.level.web' property to 'DEBUG'
2019-07-26 10:08:13.876  INFO 92 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2019-07-26 10:08:13.927  INFO 92 --- [  restartedMain] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2019-07-26 10:08:13.927  INFO 92 --- [  restartedMain] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.14]
2019-07-26 10:08:13.947  INFO 92 --- [  restartedMain] o.a.catalina.core.AprLifecycleListener   : The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path: [/opt/openjdk-12/lib/server:/opt/openjdk-12/lib:/opt/openjdk-12/../lib:/usr/java/packages/lib:/usr/lib64:/lib64:/lib:/usr/lib]
2019-07-26 10:08:14.034  INFO 92 --- [  restartedMain] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2019-07-26 10:08:14.034  INFO 92 --- [  restartedMain] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 2027 ms
2019-07-26 10:08:14.286  INFO 92 --- [  restartedMain] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2019-07-26 10:08:14.518  INFO 92 --- [  restartedMain] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2019-07-26 10:08:14.616  INFO 92 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-07-26 10:08:14.621  INFO 92 --- [  restartedMain] com.example.demo.DemoApplication         : Started DemoApplication in 3.366 seconds (JVM running for 4.207)
```

Check if the application is accessible by going to <http://localhost:8080>

You will just see a Whitelabel Error Page since we haven't setup any routes yet but that still shows the app is running and accessible.

At this point you could even update the Dockerfile to install an editor and git and manage to do some coding but that's probably not the best development experience so we will also look at using volumes so you can edit code on your own machine instead.

Now to exit the running container, hit ctrl+c if you need to stop the spring boot app and type `exit` to shut down the container.

### Work with volumes

Now you can run the container and mount a volume to edit the code on your machine.

Let's say you want to store your code in the java project of this repo under another folder called app.

From the java directory:

`mkdir -p app && cd "$_"`

On Windows:

`mkdir app`
`cd app`

Create a named volume for the current directory (using javadev here but feel free to call it something else):

`docker volume create -d local -o type=none -o o=bind -o device=$(pwd) javadev`

**Windows!** One annoying thing is $(pwd) on Windows returns a path with Widows backslashes. So you might see $(pwd) give something like `C:\Users\you\whatever` but it needs to be in the form `/c/Users/you/whatever`. I haven't found a good way to workaround this yet. One thing, if you're using regular Cmd.exe you can at least do this in the current directory: `echo %CD:\=/%` to print the path with forward slashes. You then need to replace `C:` with `/c`.

So if you're on Windows you need to update the volume command to something like:

> Of course replace my username with yours and update the path to wherever you put your code.

`docker volume create -d local -o type=none -o o=bind -o device=/c/Users/ruairi/Dev/docker-various-examples/java/app javadev`


Take a second to look at the volume you just created:

```bash
$ docker volume ls
DRIVER              VOLUME NAME
local               javadev
```

You can inspect it to make sure you are happy with the directory setup. Device is the directory we want to bind into our container soon. Basically we want to share a directory between our machine and the container. Make sure this directory is empty on your machine.

```bash
$ docker volume inspect javadev
[
  {
    "CreatedAt": "2019-02-06T19:18:44Z",
    "Driver": "local",
    "Labels": {},
    "Mountpoint": "/var/lib/docker/volumes/javadev/_data",
    "Name": "javadev",
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

```bash
[
  {
    "CreatedAt": "2019-02-07T07:27:11Z",
    "Driver": "local",
    "Labels": {},
    "Mountpoint": "/var/lib/docker/volumes/javadev/_data",
    "Name": "javadev",
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

`docker run --rm -it -v javadev:/usr/app -p 8080:8080 java-build sh`

This opens a shell in the container.
You can run the app with:

`./gradlew bootRun`

You should now be able to access the running application at <http://localhost:8080>

Notice the java code from the docker image is now in java/app. You can edit this code and your changes will be reflected in the container.

To exit, type `exit` from within the running container. Because we used the `--rm` flag, the container will be removed after exiting.

## Generate a React project, build and run it

First go to the javascript directory in this repository:

`cd javascript`

Build the docker image:

`docker build . -t react-build`

This will generate a react app for you.

You may have to ctrl+c to exit after generation is complete.

I won't repeat the steps for using docker exec of using volumes here but you can do basically the same things just updating the name of the image and container.

To see the web app running do:

`docker run -it -p 3000:3000 -w /usr/app/my-app react-build yarn start`

You should see a placeholder react page at <http://localhost:3000>

## Fun with networks

We will look at how to network docker containers together.

You can take a look at what docker networks currently exist on your machine with:

```bash
docker network ls
```

You can also inspect a network:

```bash
docker network inspect bridge
```

You can create a new network with:

```bash
docker network create my-network
```

I won't go much into networking beyond showing how to create one with docker but if you have requirements, like a particular network driver you want to use, you can set that while creating a network. For example:

```bash
docker network create --driver bridge my-network
```

This will set the driver to `bridge`. This happens to be the default driver so we left it out when creating our new network above but there are other drivers you can use if you need to. A good resource to learn more about network drivers for docker [is here](https://blog.docker.com/2016/12/understanding-docker-networking-drivers-use-cases/).

We can try running the images we made earlier and connect them to the network we just created:

```bash
docker run --rm -dit -p 3000:3000 -w /usr/app/my-app --name js-on-network --network my-network react-build yarn start
```

```bash
docker run --rm -dit -p 8080:8080 --name java-on-network --network my-network java-build ./gradlew bootRun
```

### Challenge

Can you confirm the 2 containers are attached to the same network?

Hint, docker has an [inspect command](https://docs.docker.com/engine/reference/commandline/inspect/).

Attach a shell to one of the running containers.

Can you confirm the 2 containers are on the same network from within the container?

How do they address each other?

### Extra challenge

Add a database to the network and have the spring app connect to it.

## Cleanup

You probably don't want to leave all the stuff we just created hanging around on your machine if you're not using it.

Here's how to clean it all up.

### Containers

First stop and remove the containers we created.

To see them we use [docker ps](https://docs.docker.com/engine/reference/commandline/ps/):

```bash
docker ps -a -q
```

If you stopeed the containers you need to use `-a` to show stopped containers. Because we only need the ID we pass `-q`.

Now you can copy the IDs of the container you want to remove and run:

`docker rm <the-ids-space-seperated>`

If the container is running, you would first haev to do:

`docker stop <the-ids-space-seperated>`

If you just want to get rid of all containers you can do this:

```bash
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

### Networks

Remove the network we created with `docker network rm my-network`

You can also prune all unused networks with `docker network prune`

## Next steps

Check out [this post on using docker for local development](http://www.realgorithm.io/2018/04/simple-dev-environment-with-docker-compose/)

[Play with docker](https://labs.play-with-docker.com/)

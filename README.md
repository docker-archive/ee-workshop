# Deploying Multi-OS applications with Docker EE

Docker EE 2.0 (beta) is the first Containers-as-a-Service platform to offer production-level support for the integrated management and security of both Linux and Windows Server Containers. It is also the first platform to support both Docker Swarm and Kubernetes orchestration.

In this lab we'll use a Docker EE cluster comprised of Windows and Linux nodes. We'll deploy both a Java web app on Linux and a multi-service application that includes both Windows and Linux components using Docker Swarm. Then we'll take a look at securing and scaling the application. Finally, we will then deploy the app using Kubernetes.

> **Difficulty**: Intermediate (assumes basic familiarity with Docker) If you're looking for a basic introduction to Docker, check out [https://training.play-with-docker.com](https://training.play-with-docker.com)

> **Time**: Approximately 75 minutes

> **Introduction**:
>	* [What is the Docker Platform](#intro1)
>	* [Overview of Orchestration](#intro2)
>		* [Basics of Docker Swarm mode](#intro2.1)
>		* [Basics of Kubernetes](#intro2.2)

> **Tasks**:

> * [Task 1: Configure the Docker EE Cluster](#task1)
>   * [Task 1.1: Accessing PWD](#task1.1)
>   * [Task 1.2: Install a Windows worker node](#task1.2)
>   * [Task 1.3: Create Three Repositories](#task1.3)
>   * [Task 1.3.1: Restrict access to a repository](#task1.3.1)
> * [Task 2: Deploy a Java Web App](#task2)
>   * [Task 2.1: Clone the Demo Repo](#task2.1)
>   * [Task 2.2: Build and Push the Linux Web App Image](#task2.2)
>   * [Task 2.3: Deploy the Web App using UCP](#task2.3)
> * [Task 3: Deploy the next version with a Windows node](#task3)
>   * [Task 3.1: Clone the repository](#task3.1)
>   * [Task 3.2: Build and Push Your Java Images to Docker Trusted Registry](#task3.2)
>   * [Task 3.3: Deploy the Java web app with Universal Control Plane](#task3.3)
>   * [Task 3.4: Deploy the Windows .NET App](#task3.4)
> * [Task 4: Deploy to Kubernetes](#task4)
>   * [Task 4.1: Build .NET Core app instead of .NET](#task4.1)
>   * [Task 4.2: Examine the Docker Compose File](#task4.2)
>   * [Task 4.3: Deploy to Kubernetes using the Docker Compose file](#task4.3)
>   * [Task 4.4: Verify the app](#task4.4)

## Understanding the Play With Docker Interface

![](./images/pwd_screen.png)

This workshop is only available to people in a pre-arranged workshop. That may happen through a [Docker Meetup](https://events.docker.com/chapters/), a conference workshop that is being led by someone who has made these arrangements, or special arrangements between Docker and your company. The workshop leader will provide you with the URL to a workshop environment that includes [Docker Enterprise Edition](https://www.docker.com/enterprise-edition). The environment will be based on [Play with Docker](https://labs.play-with-docker.com/).

If none of these apply to you, contact your local [Docker Meetup Chapter](https://events.docker.com/chapters/) and ask if there are any scheduled workshops. In the meantime, you may be interested in the labs available through the [Play with Docker Classroom](training.play-with-docker.com).

There are three main components to the Play With Docker (PWD) interface. 

### 1. Console Access
Play with Docker provides access to the 3 Docker EE hosts in your Cluster. These machines are:

* A Linux-based Docker EE 18.01 Manager node
* Three Linux-based Docker EE 18.01 Worker nodes
* A Windows Server 2016-based Docker EE 17.06 Worker Node

> **Important Note: beta** Please note, as of now this is a beta Docker EE 2.0 environment. Docker EE 2.0 shows off the new Kubernetes functionality which is described below.

By clicking a name on the left, the console window will be connected to that node.

### 2. Access to your Universal Control Plane (UCP) and Docker Trusted Registry (DTR) servers

Additionally, the PWD screen provides you with a one-click access to the Universal Control Plane (UCP)
web-based management interface as well as the Docker Trusted Registry (DTR) web-based management interface. Clicking on either the `UCP` or `DTR` button will bring up the respective server web interface in a new tab.

### 3. Session Information

Throughout the lab you will be asked to provide either hostnames or login credentials that are unique to your environment. These are displayed for you at the bottom of the screen.

## Document conventions

- When you encounter a phrase in between `<` and `>`  you are meant to substitute in a different value.

	For instance if you see `<dtr domain>` you would actually type something like `ip172-18-0-7-b70lttfic4qg008cvm90.direct.ee-workshop.play-with-docker.com`


- When you see the Linux penguin all the following instructions should be completed in your Linux console

	![](./images/linux75.png)

- When you see the Windows flag all the subsequent instructions should be completed in your Windows console

    ![](./images/windows75.png)

## <a name="intro1"></a>Introduction
Docker EE provides an integrated, tested and certified platform for apps running on enterprise Linux or Windows operating systems and Cloud providers. Docker EE is tightly integrated to the the underlying infrastructure to provide a native, easy to install experience and an optimized Docker environment. Docker Certified Infrastructure, Containers and Plugins are exclusively available for Docker EE with cooperative support from Docker and the Certified Technology Partner.

### <a name="intro2"></a>Overview of Orchestration
While it is easy to run an application in isolation on a single machine, orchestration allows you to coordinate multiple machines to manage an application, with features like replication, encryption, loadbalancing, service discovery and more. If you've read anything about Docker, you have probably heard of Kubernetes and Docker swarm mode. Docker EE allows you to use either Docker swarm mode or Kubernetes for orchestration. 

Both Docker swarm mode and Kubernetes are declarative: you declare your cluster's desired state, and applications you want to run and where, networks, and resources they can use. Docker EE simplifies this by taking common concepts and moving them to the a shared resource.

#### <a name="intro2.1"></a>Overview of Docker Swarm mode
A swarm is a group of machines that are running Docker and joined into a cluster. After that has happened, you continue to run the Docker commands you’re used to, but now they are executed on a cluster by a swarm manager. The machines in a swarm can be physical or virtual. After joining a swarm, they are referred to as nodes.

Swarm mode uses managers and workers to run your applications. Managers run the swarm cluster, making sure nodes can communicate with each other, allocate applications to different nodes, and handle a variety of other tasks in the cluster. Workers are there to provide extra capacity to your applications. In this workshop, you have one manager and three workers.

#### <a name="intro2.2"></a>Overview of Kubernetes

Kubernetes is available in Docker EE 2.0 (currently in beta) and included in this workshop. Kubernetes deployments tend to be more complex than Docker Swarm, and there are many component types. UCP simplifies a lot of that, relying on Docker Swarm to handle shared resources. We'll concentrate on Pods and Load Balancers in this workshop, but there's plenty more supported by UCP 2.0.

## <a name="task1"></a>Task 1: Configure the Docker EE Cluster

The Play with Docker (PWD) environment is almost completely set up, but before we can begin the labs, we need to do two more steps. First we'll add a Windows node to the cluster. We've left the node unjoined so you can see how easy it is to do. Then we'll create two repositories in Docker Trusted Registry.
(The Linux worker nodes are already added to the cluster)

### <a name="task 1.1"></a>Task 1.1: Accessing PWD

1. Navigate in your web browser to the URL the workshop organizer provided to you.

2. Fill out the form, and click `submit`. You will then be redirected to the PWD environment.

3. Click `Access`

	It may take a few minutes to provision out your PWD environment. After this step completes, you'll be ready to move on to task 1.2: Install a Windows worker node

### <a name="task1.2"></a>Task 1.2: Join a Windows worker node

Let's start by adding our 3rd node to the cluster, a Windows Server 2016 worker node. This is done using Docker Swarm.

1. From the main PWD screen click the `UCP` button on the left side of the screen

	> **Note**: Because this is a lab-based install of Docker EE we are using the default self-signed certs. Because of this your browser may display a security warning. It is safe to click through this warning.
	>
	> In a production environment you would use certs from a trusted certificate authority and would not see this screen.
	>
	> ![](./images/ssl_error.png)

2. When prompted enter your username and password (these can be found below the console window in the main PWD screen). The UCP web interface should load up in your web browser.

	> **Note**: Once the main UCP screen loads you'll notice there is a red warning bar displayed at the top of the UCP screen, this is an artifact of running in a lab environment. A UCP server configured for a production environment would not display this warning
	>
	> ![](./images/red_warning.png)


3. From the main dashboard screen, click `Add a Node` on the bottom left of the screen

	![](./images/add_a_node.png)

4. Select node type "Windows", check the box, that you followed the instructions and copy the text from the dark box shown on the `Add Node` screen. Don't select a custom listen or advertise address.

	> **Note** There is an icon in the upper right corner of the box that you can click to copy the text to your clipboard
	> ![](./images/join_text.png)


	> **Note**: You may notice that there is a UI component to select `Linux` or `Windows`on the `Add Node` screen. In a production environment where you are starting from scratch there are [a few prerequisite steps] to adding a Windows node. However, we've already done these steps in the PWD environment. So for this lab, just leave the selection on `Linux` and move on to step 2

![](./images/windows75.png)

6. Switch back to the PWD interface, and click the name of your Windows node. This will connect the web-based console to your Windows Server 2016 Docker EE host.

7. Paste the text from Step 4 at the command prompt in the Windows console. (depending on your browser, this can be tricky: try the "paste" command from the edit menu instead of right clicking or using keyboard shortcuts)

	You should see the message `This node joined a swarm as a worker.` indicating you've successfully joined the node to the cluster.

5. Switch back to the UCP server in your web browser and click the `x` in the upper right corner to close the `Add Node` window

6. You will be taken back to the UCP Dashboard. In the left menu bar, click Shared Resources, and select Nodes.

![](/images/select_nodes.png)

You should be taken to the `Nodes` screen and will see 4 worker nodes listed at the bottom of your screen.

	Initially the new worker node will be shown with status `down`. After a minute or two, refresh your web browser to ensure that your Windows worker node has come up as `healthy`

	![](./images/node_listing.png)
	> \: Update with new nodes page screenshot 

Congratulations on adding a Windows node to your UCP cluster. Now you are ready to use the worker in either Swarm or Kubernetes. Next up we'll create a couple of repositories in Docker Trusted registry.

### <a name="task1.3"></a>Task 1.3: Create Three DTR Repositories

Docker Trusted Registry is a special server designed to store and manage your Docker images. In this lab we're going to create a couple of different Docker images, and push them to DTR. But before we can do that, we need to setup repositories in which those images will reside. Often that would be enough.

However, before we create the repositories, we do want to restrict access to them. Since we have two distinct app components, a Java web app, and a .NET API, we want to restrict access to them to the team that develops them, as well as the administrators. To do that, we need to create two users and then two organizations.

1. In the PWD web interface click the `DTR` button on the left side of the screen.

	> **Note**: As with UCP before, DTR is also using self-signed certs. It's safe to click through any browser warning you might encounter.

2. From the main DTR page, click users and then the New User button.
![](./images/user_screen.png)

3. Create a new user, `java_user` and give it a password you'll remember. I used `user1234`. Be sure to save the user.
![](/images/create_java_user.png)
Then do the same for a `dotnet_user`.

4. Select the Organization button.
![](./images/organization_screen.png)

5. Press New organization button, name it java, and click save.
![](./images/java_organization_new.png)
Then do the same with dotnet and you'll have two organizations.
![](./images/two_organizations.png)

6. Now you get to add a repository! Click on the java organization, select repositories and then Add repository
![](./images/add_repository_java.png)

7. Name the repository `java_web`. 

	![](./images/create_repository.png)
> Note the repository is listed as "Public" but that means it is publicly viewable by users of DTR. It is not available to the general public.

8. Now it's time to create a team so you can restrict access to who administers the images. Select the `java` organization and the members will show up. Press Add user and start typing in java. Select the `java_user` when it comes up.
![](./images/add_java_user_to_organization.png)

9. Next select the `java` organization and press the `Team` button to create a `web` team.
![](./images/team.png)

10. Add the `java_user` user to the `web` team and click save.
![](./images/team_add_user.png)
![](./images/team_with_user.png)

11. Next select the `web` team and select the `Repositories` tab. Select `Add Existing repository` and choose the `java_web`repository. You'll see the `java` account is already selected. Then select `Read/Write` permissions so the `web` team has permissions to push images to this repository. Finally click `save`.
![](./images/add_java_web_to_team.png)

12. Now add a new repository owned by the web team and call it `database`.

12. Repeat 4-11 above to create a `dotnet` organization with the `dotnet_user` and a repository called `dotnet_api`.
You'll now see both repositories listed.
	![](./images/two_repositories.png)

Congratulations, you have created two new repositories in two new organizations, each with one user.

## <a name="task2"></a>Task 2: Deploy a Java Web App with Universal Control Plane
Now that we've completely configured our cluster, let's deploy a couple of web apps. These are simple web pages that allow you to send a tweet. One is built on Linux using NGINX and the other is build on Windows Server 2016 using IIS.  

Let's start with the Linux version.

### <a name="task2.1"></a> Task 2.1: Clone the Demo Repo

![](./images/linux75.png)

1. From PWD click on the `worker1` link on the left to connnect your web console to the UCP Linux worker node.

2. Before we do anything, let's configure an environment variable for the DTR URL. You may remember that the session information from the Play with Docker landing page. Select and copy the the URL for the DTR host.

	![](./images/session-information.png)

3. Set an environment variable $DTR. For instance, if your DTR host name was `ip172-18-0-17-bajlvkom5emg00eaner0.direct.ee-beta2.play-with-docker.com`, you would type:

```
$ DTR='ip172-18-0-17-bajlvkom5emg00eaner0.direct.ee-beta2.play-with-docker.com'
```

4. Now use git to clone the workshop repository.

	```
	$ git clone https://github.com/dockersamples/hybrid-app.git
	```

	You should see something like this as the output:

	```
	Cloning into 'hybrid-app'...
	remote: Counting objects: 389, done.
	remote: Compressing objects: 100% (17/17), done.
	remote: Total 389 (delta 4), reused 16 (delta 1), pack-reused 363
	Receiving objects: 100% (389/389), 13.74 MiB | 3.16 MiB/s, done.
	Resolving deltas: 100% (124/124), done.
	Checking connectivity... done.
	```

	You now have the necessary demo code on your worker host.

### <a name="task2.2"></a> Task 2.2: Build and Push the Linux Web App Images

![](./images/linux75.png)

1. Change into the `java-app` directory.

	`$ cd ./hybrid-app/java-app/`

2. Set the DTR_HOST environment variable. This will be useful throughout the workshop.

```
$ export DTR_HOST=<dtr hostname>
$ echo $DTR_HOST
```

2. Use `docker build` to build your Docker image.

	`$ docker build -t $DTR_HOST/java/java_web .`

	> **Note**: Be sure to substitute your DTR Hostname and your User Name - both these are listed at the top of your PWD page.

	The `-t` tags the image with a name. In our case, the name indicates which DTR server and under which organization's respository the image will live.

	> **Note**: Feel free to examine the Dockerfile in this directory if you'd like to see how the image is being built.

	There will be quite a bit of output. The Dockerfile describes a two-stage build. In the first stage, a Maven base image is used to build the Java app. But to run the app you don't need Maven or any of the JDK stuff that comes with it. So the second stage takes the output of the first stage and puts it in a much smaller Tomcat image.

3. Log into your DTR server from the command line

first use the dotnet_user, which isn't part of the java organization

	```
	$ docker login $DTR_HOST
	Username: <your username>
	Password: <your password>
	Login Succeeded
	```
	
	Use `docker push` to upload your image up to Docker Trusted Registry.

	
	```
	$ docker push $DTR_HOST/java/java_web
	```
	
	> TODO: add output of failure to push


	The access control that you established in the [Task 1.3](#task1.3) prevented you from pushing to this repository.	

4. Now try logging in using `java-user`, and then use `docker push` to upload your image up to Docker Trusted Registry.

	```
	$ docker push $DTR_HOST/java/java_web
	```

	The output should be similar to the following:

	```
	The push refers to a repository [<dtr hostname>/java/java_web]
	feecabd76a78: Pushed
	3c749ee6d1f5: Pushed
	af5bd3938f60: Pushed
	29f11c413898: Pushed
	eb78099fbf7f: Pushed
	latest: digest: sha256:9a376fd268d24007dd35bedc709b688f373f4e07af8b44dba5f1f009a7d70067 size: 1363
	```

	Success! Because you are using a user name that belongs to the right team in the right organization, you can push your image to DTR.

5. In your web browser head back to your DTR server and click `View Details` next to your `java_web` repo to see the details of the repo.

	> **Note**: If you've closed the tab with your DTR server, just click the `DTR` button from the PWD page.

6. Click on `Images` from the horizontal menu. Notice that your newly pushed image is now on your DTR.
![](./images/pushed_image.png)

7. Repeat 1,2 and 4 but build a `java/database` in the `database/` directory and push it to DTR. This is a simple MySQL database with a basic username/password and an initial table configuration.

### <a name="task2.3"></a> Task 2.3: Deploy the Web App using UCP
![](./images/linux75.png)

The next step is to run the app in Swarm. As a reminder, the application has two components, the web front-end and the database. In order to connect to the database, the application needs a password. If you were just running this in development you could easily pass the password around as a text file or an environment variable. But in production you would never do that. So instead, we're going to create an encrypted secret. That way access can be strictly controlled.

1. Go back to the first Play with Docker tab. Click on the UCP button. You'll have the same warnings regarding `https` that you have before. Click through those and log in. You'll see the Universal Control Panel dashboard.

2.  There's a lot here about managing the cluster. You can take a moment to explore around. When you're ready, click on `Swarm` and select `Secrets`.
![](./images/ucp_secret_menu.png)

3. You'll see a `Create Secret` screen. Type `MYSQL_PASSWORD` in `Name` and `password` in `Content`. Then click `Create` in the lower left. Obviously you wouldn't use this password in a real production environment. You'll see the content box allows for quite a bit of content, you can actually create structred content here that will be encrypted with the secret.

4. Next we're going to create two networks. First click on `Networks` under `Swarm` in the left panel, and select `Create Network` in the upper right. You'll see a `Create Network` screen. Name your first network `back-tier`. Leave everything else the default.
![](./ucp_network.png)

5. Repeat step 4 but with a new network `front-tier`.

6. Now we're going to use the fast way to create your application: `Stacks`. In the left panel, click `Shared Resources`, `Stacks` and then `Create Stack` in the upper right corner.

7. Name your stack `java_web` and select `Swarm Services` for your `Mode`. Below you'll see we've included a `.yml` file. Before you paste that in to the `Compose.yml` edit box, note that you'll need to make a quick change. Each of the images is defined as `<your-dtr-instance>/java/<something>`. You'll need to change the `<your-dtr-instance>` to the DTR Hostname found on the Play with Docker landing page for your session. It will look something like this:
`ip172-18-0-21-baeqqie02b4g00c9skk0.direct.ee-beta2.play-with-docker.com`
You can do that right in the edit box in `UCP` but wanted to make sure you saw that first.
![](./images/ucp_create_stack.png)

Here's the `Compose` file. Once you've copy and pasted it in, and made the changes, click `Create` in the lower right corner.
```
version: "3.3"

services:

  database:
    image: <your-dtr-instance>/java/database
    # set default mysql root password, change as needed
    environment:
      MYSQL_ROOT_PASSWORD: mysql_password
    # Expose port 3306 to host. 
    ports:
      - "3306:3306" 
    networks:
      - back-tier

  webserver:
    image: <your-dtr-instance>/java/java_web
    ports:
      - "8080:8080" 
      - "8000:8000"
    networks:
      - front-tier
      - back-tier

networks:
  back-tier:
  front-tier:

secrets:
  mysql_password:
    external: true
```
Then click `Done` in the lower right.

8. Click on `Stacks` again, and select the `java_web` stack. Click on `Inspect Resources` and then select `Services`. Select `java_web_webserver`. In the right panel, you'll see `Published Endpoints`. Select the one with `:8080` at the end. You'll see a `Apache Tomcat/7.0.84` landing page. Add `/java-web` to the end of the URL and you'll see you're app.

![](./images/java_web1.png)

## <a name="task3"></a>Task 3: Deploy the next version with a Windows node

Now that we've moved the app and updated it, we're going to add in a user sign-in API. For fun, and to show off the cross-platform capabilities of Docker EE, we are going to do it in a Windows container.

### <a name="task3.1"></a> Task 3.1: Clone the repository
![](./images/windows75.png)

1. Because this is a Windows container, we have to build it on a Windows host. Switch back to the main Play with Docker page, select the name of the Windows worker. Then clone the repository again onto this host:

	```
	PS C:\git clone https://github.com/dockersamples/hybrid-app.git
	```
2. Set an environment variable for the DTR host name. Much like you did for the Java app, this will make a few step easier. Copy the DTR host name again and create the environment variable. For instance, if your DTR host was `ip172-18-0-17-bajlvkom5emg00eaner0.direct.ee-beta2.play-with-docker.com` you would type:

	```
	PS C:\> $env:DTR="ip172-18-0-17-bajlvkom5emg00eaner0.direct.ee-beta2.play-with-docker.com"

### <a name="task3.2"></a> Task 3.2: Build and Push Windows Images to Docker Trusted Registry
![](./images/windows75.png)

1. CD into the `c:\hybrid-app\netfx-api` directory. 

	> Note you'll see a `dotnet-api` directory as well. Don't use that directory. That's a .NET Core api that runs on Linux. We'll use that later in the Kubernetes section.

	`PS C:\> cd c:\hybrid-app\netfx-api\`


2. Use `docker build` to build your Windows image.

	`$ docker build -t $env:DTR/dotnet/dotnet_api .`

	> **Note**: Feel free to examine the Dockerfile in this directory if you'd like to see how the image is being built.

	Your output should be similar to what is shown below

	```
	PS C:\hybrid-app\netfx-api> docker build -t $env:DTR/dotnet/dotnet_api .

	Sending build context to Docker daemon  415.7kB
	Step 1/8 : FROM microsoft/iis:windowsservercore-10.0.14393.1715
	 ---> 590c0c2590e4

	<output snipped>

	Removing intermediate container ab4dfee81c7e
	Successfully built d74eead7f408
	Successfully tagged <dtr hostname>/dotnet/dotnet_api:latest
	```
	> **Note**: It will take a few minutes for your image to build.

4. Log into Docker Trusted Registry

	```
	PS C:\> docker login $env:DTR
	Username: dotnet_user
	Password: user1234
	Login Succeeded
	```

5. Push your new image up to Docker Trusted Registry.

	```
	PS C:\Users\docker> docker push $env:DTR/dotnet/dotnet_api
	The push refers to a repository [<dtr hostname>/dotnet/dotnet_api]
	5d08bc106d91: Pushed
	74b0331584ac: Pushed
	e95704c2f7ac: Pushed
	669bd07a2ae7: Pushed
	d9e5b60d8a47: Pushed
	8981bfcdaa9c: Pushed
	25bdce4d7407: Pushed
	df83d4285da0: Pushed
	853ea7cd76fb: Pushed
	55cc5c7b4783: Skipped foreign layer
	f358be10862c: Skipped foreign layer
	latest: digest: sha256:e28b556b138e3d407d75122611710d5f53f3df2d2ad4a134dcf7782eb381fa3f size: 2825
	```
6. You may check your repositories in the DTR web interface to see the newly pushed image.


### <a name="task3.3"></a> Task 3.3: Deploy the Java web app
![](./images/linux75.png)

1. First we need to update the Java web app so it'll take advantage of the .NET API. Switch back to `worker1` and change directories to the `java-app-v2` directory. Repeat steps 1,2, and 4 from Task 2.2 but add a tag `:2` to your build and pushes:

	```
	$ docker build -t $env:DTR/java/java_web:2 .
	$ docker push $env:DTR/java/java_web:2
	```
This will push a different version of the app, version 2, to the same `java_web` repository.

2. Next repeat the steps 6-8 from Task 2.3, but use this `Compose` file instead:

```
version: "3.3"

services:

  database:
    image: <your-dtr-instance>/java/database
    # set default mysql root password, change as needed
    environment:
      MYSQL_ROOT_PASSWORD: mysql_password
    # Expose port 3306 to host. 
    ports:
      - "3306:3306" 
    networks:
      - back-tier

  webserver:
   image: <your-dtr-instance>/java/java_web:2
   ports:
     - "8080:8080" 
     - "8000:8000"
   networks:
     - front-tier
     - back-tier
   environment:
     BASEURI: http://dotnet-api/api/users

  dotnet-api:
    image: <your-dtr-instance>/dotnet/dotnet-api
    ports:
      - "57989:80"
    networks:
      - front-tier
      - back-tier

networks:
  back-tier:
  front-tier:

secrets:
  mysql_password:
    external: true
```


## <a name="task4"></a>Task 4: Deploy to Kubernetes

Now that we have built, deployed and scaled a multi OS application to Docker EE using Swarm mode for orchestration, let's learn how to use Docker EE with Kubernetes.

Docker EE lets you choose the orchestrator to use to deploy and manage your application, between Swarm and Kubernetes. In the previous tasks we have used Swarm for orchestration. In this section we will deploy the application to Kubernetes and see how Docker EE exposes Kubernetes concepts.

### <a name="task4.1"></a>Task 4.1: Build .NET Core app instead of .NET
![](./images/linux75.png)

For now Kubernetes does not support Windows workloads in production, so we will start by porting the .NET part of our application to a Linux container using .NET Core.

1. From the Play with Docker landing page, click on `worker1` and CD into the `hybrid-app/dotnet-api` directory. 

	`$ cd ~/hybrid-app/dotnet-api/`

2. Use `docker build` to build your Linux image.

	`$ docker build -t $DTR_HOST/dotnet/dotnet_api:core .`

	> **Note**: Feel free to examine the Dockerfile in this directory if you'd like to see how the image is being built. Also, we used the `:core` tag so that the repository has two versions, the original with a Windows base image, and this one with a Linux .NET Core base image.

	Your output should be similar to what is shown below

```
Sending build context to Docker daemon   29.7kB
Step 1/10 : FROM microsoft/aspnetcore-build:2.0.3-2.1.2 AS builder
2.0.3-2.1.2: Pulling from microsoft/aspnetcore-build
723254a2c089: Pull complete

	<output snipped>

Removing intermediate container 508751aacb5c
Step 7/10 : FROM microsoft/aspnetcore:2.0.3-stretch
2.0.3-stretch: Pulling from microsoft/aspnetcore

Successfully built fcbc49ef89bf
Successfully tagged ip172-18-0-8-baju0rgm5emg0096odmg.direct.ee-beta2.play-with-docker.com/dotnet/dotnet_api:latest
```
	> **Note**: It will take a few minutes for your image to build.

4. Log into Docker Trusted Registry

	```
	$ docker login $DTR_HOST
	Username: dotnet_user
	Password: user1234
	Login Succeeded
	```

5. Push your new image up to Docker Trusted Registry.

	```
	$ docker push $DTR_HOST/dotnet/dotnet_api:core
	The push refers to a repository [<dtr hostname>/dotnet/dotnet_api]
	5d08bc106d91: Pushed
	74b0331584ac: Pushed
	e95704c2f7ac: Pushed
	669bd07a2ae7: Pushed
	d9e5b60d8a47: Pushed
	8981bfcdaa9c: Pushed
	25bdce4d7407: Pushed
	df83d4285da0: Pushed
	853ea7cd76fb: Pushed
	55cc5c7b4783: Skipped foreign layer
	f358be10862c: Skipped foreign layer
	latest: digest: sha256:e28b556b138e3d407d75122611710d5f53f3df2d2ad4a134dcf7782eb381fa3f size: 2825
	```

6. You may check your repositories in the DTR web interface to see the newly pushed image.

### <a name="task4.2"></a>Task 4.2: Examine the Docker Compose File
![](./images/linux75.png)

Docker EE lets you deploy native Kubernetes applications using Kubernetes deployment descriptors, by pasting the yaml files in the UI, or using the `kubectl` CLI tool.

However many developers use `docker-compose` to build and test their application, and having to create Kubernetes deployment descriptors as well as maintaining them in sync with the Docker Compose file is tedious and error prone.

In order to make life easier for developers and operations, Docker EE lets you deploy an application defined with a Docker Compose file as a Kubernetes workloads. Internally Docker EE uses the official Kubernetes extension mecanism by defining a [Custom Resource Definition](https://kubernetes.io/docs/tasks/access-kubernetes-api/extend-api-custom-resource-definitions/) (CRD) defining a stack object. When you post a Docker Compose stack definition to Kubernetes in Docker EE, the CRD controller takes the stack definition and translates it to Kubernetes native resources like pods, controllers and services.

We'll use a Docker Compose file to instantiate our application, and it's the same file as before, except that we will switch the .NET Docker Windows image with the .NET Core Docker Linux image we just built.

Let's look at the Docker Compose file in `app/docker-stack.yml`.

Change the images for the dotnet-api and java-app services for the ones we just built. And remember to change `<dtr hostname>` to the long DTR hostname listed on the landing page for your Play with Docker instance.

```
version: '3.3'

services:
  database:
    deploy:
      placement:
        constraints:
        - node.platform.os == linux

    image: <dtr hostname>/java/database
    networks:
      back-tier: null
    ports:
    - mode: ingress
      published: 3306
      target: 3306
  dotnet-api:
    deploy:
      placement:
        constraints:
        - node.platform.os == linux
    image: <dtr hostname>/dotnet/dotnet_api:core
    networks:
      back-tier: null
    ports:
    - mode: ingress
      published: 57989
      target: 80
  java-web:
    deploy:
      placement:
        constraints:
        - node.platform.os == linux
    image: <dtr hostname>/java/java_web:2

    networks:
      back-tier:
      front-tier:

    ports:
    - mode: ingress
      published: 8000
      target: 8000
    - mode: ingress
      published: 8080
      target: 8080

networks:
  back-tier:
  front-tier:

secrets:
  mysql_password:
    external: true
```

### <a name="task4.3"></a>Task 4.3: Deploy to Kubernetes using the Docker Compose file
![](./images/linux75.png)

Login to UCP, go to Shared resources, Stacks.

![](./images/kube-stacks.png)

Click create Stack. Fill name: hybrid-app, mode: Kubernetes Workloads, namespace: default.

![](./images/kube-create-stack.png)

You should see the stack being created.

![](./images/kube-stack-created.png)

Click on it to see the details.

![](./images/kube-stack-details.png)

### <a name="task4.4"></a>Task 4.4: Verify the app
![](./images/linux75.png)

Go to Kubernetes / Pod. See the pods being deployed.

![](./images/kube-pods.png)

Go to Kubernetes / Controllers. See the deployments and ReplicaSets.

![](./images/kube-controllers.png)

Go to Kubernetes / Load Balancers. See the Kubernetes services that have been created.

![](./images/kube-lb.png)

Click on `java-app-published` to the the details of the public load balancer created for the Java application.

![](./images/kube-java-lb.png)

There will be a link for the public url where the service on port 8080 is exposed. Click on that link, add `/java-web/` at the end of the url. You should be led to the running application.

![](./images/kube-running-app.png)

## Conclusion

In this lab we've looked how Docker EE can help you manage both Linux and Windows workloads whether they be traditional apps you've modernized or newer cloud-native apps, leveraging Swarm or Kubernetes for orchestration.

You can find more information on Docker EE at [http://www.docker.com](http://www.docker.com/enterprise-edition) as well as continue exploring using our hosted trial at [https://dockertrial.com](https://dockertrial.com)

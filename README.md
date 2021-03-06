# [jenkins](https://jenkins.io/doc/tutorials/)

# Run Jenkins in Docker
- In this tutorial, you’ll be running Jenkins as a Docker container from the jenkinsci/blueocean Docker image.
- Run the jenkinsci/blueocean image as a container in Docker, this command automatically downloads the image if this hasn’t been done or updates the jenkinsci/blueocean Docker image, if an updated one is available.
```bash
docker run \
  -u root \
  -d \
  --rm \
  -p 8081:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v $(which docker):/usr/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --name jenkins-docker \
  jenkinsci/blueocean
```
Note:
```bash
-v /var/run/docker.sock:/var/run/docker.sock \
```
- Map the /var/jenkins_home directory in the container to the Docker volume with the name jenkins-data on the host machine.
- If this volume does not exist on the host machine, then this docker run command will automatically create the volume for you.
```bash
-v "$HOME":/home \ 
```
- Map the $HOME directory on the host (i.e. your local) machine (usually the /Users/<your-username> directory) to the /home directory in the container
```bash
docker exec -it jenkins-docker bash
```
- Access Jenkins/Blue Ocean Docker container through terminal using docker exec commands add --name flag

# Jenkinsfile Java
- Create a Jenkinsfile in the route of SCM
```
pipeline {
    agent {
        docker {
            image 'maven:3-alpine' 
            args '-v /root/.m2:/root/.m2' 
        }
    }
    stages {
        stage('Build') { 
            steps {
                sh 'mvn -B -DskipTests clean package' 
            }
        }
	stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
	stage('Deliver') {
            steps {
                sh './jenkins/scripts/deliver.sh'
            }
        }
    }
}
```
- The **image** parameter (of the agent section’s docker parameter) downloads the maven:3-apline Docker image (if it’s not already available on your machine) and runs this image as a separate container. This means that:
- You’ll have separate Jenkins and Maven containers running locally in Docker.
- The Maven container becomes the agent that Jenkins uses to run your Pipeline project. However, this container is short-lived - its lifespan is only that of the duration of your Pipeline’s execution.

- The **args** parameter creates a reciprocal mapping between the /root/.m2 (i.e. Maven repository) directories in the short-lived Maven Docker container and that of your Docker host’s filesystem. 
- This is to ensure that the artifacts necessary to build your Java application (which Maven downloads while your Pipeline is being executed) are retained in the Maven repository beyond the lifespan of the Maven container. 
- This prevents Maven from having to download the same artifacts during successive runs of your Jenkins Pipeline, which you’ll be conducting later on. 
- Be aware that unlike the Docker data volume you created for jenkins-data above, the Docker host’s filesystem is effectively cleared out each time Docker is restarted. This means you’ll lose the downloaded Maven repository artifacts each time Docker restarts.

- Defines a stage (directive) called **Build** that appears on the Jenkins UI.
- This sh step (of the steps section) runs the Maven command to cleanly build your Java application (without running any tests).

- Defines a stage (directive) called **Test** that appears on the Jenkins UI.
- This sh step (of the steps section) executes the Maven command to run the unit test on your simple Java application. 
- This command also generates a JUnit XML report, which is saved to the target/surefire-reports directory (within the /var/jenkins_home/workspace/simple-java-maven-app directory in the Jenkins container).
- This junit step (provided by the JUnit Plugin) archives the JUnit XML report (generated by the mvn test command above) and exposes the results through the Jenkins interface. 
- In Blue Ocean, the results are accessible through the Tests page of a Pipeline run. 

- The **post** section’s always condition that contains this junit step ensures that the step is always executed at the completion of the Test stage, regardless of the stage’s outcome

- Defines a new stage called **Deliver** that appears on the Jenkins UI.
- This sh step (of the steps section) runs the shell script deliver.sh located in the jenkins/scripts directory from the root of the simple-java-maven-app repository. 
- Explanations about what this script does are covered in the deliver.sh file itself. 
- As a general principle, it’s a good idea to keep your Pipeline code (i.e. the Jenkinsfile) as tidy as possible and place more complex build steps (particularly for stages consisting of 2 or more steps) into separate shell script files like the deliver.sh file. This ultimately makes maintaining your Pipeline code easier, especially if your Pipeline gains more complexity.

# Run Jenkins
http://localhost:8080/blue/organizations/jenkins/simple-java-maven-app/activity
Go back to Jenkins again, log in again if necessary and ensure you’ve accessed Jenkins’s Blue Ocean interface.

Click Run at the top left, then quickly click the OPEN link which appears briefly at the lower-right to see Jenkins running your amended Pipeline project. If you weren’t able to click the OPEN link, click the top row on the Blue Ocean interface to access this feature.

# Run Docker Inside Jenkins

## Mac
- docker.sock
```bash
docker run \
  -u root \
  -d \
  --rm \
  -p 8081:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --name jenkins-docker \
  jenkinsci/blueocean
```
<span style="color:red">ERROR:</span> http://localhost:8081/job/github-conalryan-jenkins/38/console

[Pipeline] { (Declarative: Agent Setup)
[Pipeline] isUnix
[Pipeline] readFile
[Pipeline] sh
+ docker build -t 6473f000819aec3c716ce22fcc82f5d58a9ff7e8 -f Dockerfile .
Sending build context to Docker daemon  452.6kB

...

[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (versions)
[Pipeline] sh
+ pwd
/var/jenkins_home/workspace/github-****-jenkins
[Pipeline] sh
+ mvn -v
Apache Maven 3.2.3 (33f8c3e1027c3ddde99d3cdebad2656a31e8fdf4; 2014-08-11T20:58:10+00:00)
Maven home: /opt/apache-maven-3.2.3
Java version: 1.8.0_191, vendor: Oracle Corporation
Java home: /usr/lib/jvm/java-8-openjdk-amd64/jre
Default locale: en_US, platform encoding: ANSI_X3.4-1968
OS name: "linux", version: "4.9.125-linuxkit", arch: "amd64", family: "unix"
[Pipeline] sh
+ docker -v
/var/jenkins_home/workspace/github-****-jenkins@tmp/durable-39b25248/script.sh: 1: /var/jenkins_home/workspace/github-****-jenkins@tmp/durable-39b25248/script.sh: docker: not found

- $(which docker)
```bash
which docker
# /usr/local/bin/docker
```
```bash
docker run \
  -u root \
  -d \
  --rm \
  -p 8081:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v $(which docker):/usr/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --name jenkins-docker \
  jenkinsci/blueocean
```
<span style="color:green">SUCCESS:</span> http://localhost:8081/job/github-conalryan-jenkins/1/

```bash
docker exec -it jenkins-docker bash
docker -v
Docker version 18.09.2, build 6247962
```
# machine
- docker.sock
```bash
docker run \
  -u root \
  -d \
  --rm \
  -p 8081:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --name jenkins-docker \
  jenkinsci/blueocean
```

<span style="color:red">ERROR:</span>

[Pipeline] { (Declarative: Agent Setup)
[Pipeline] isUnix
[Pipeline] readFile
[Pipeline] sh
+ docker build -t 3a36f5981b68a21e3f820761b080ffc092765809 -f Dockerfile .
Sending build context to Docker daemon    150MB

...

[Pipeline] sh
+ docker -v
/var/jenkins_home/workspace/acs-hotel-call-center@tmp/durable-8e4744af/script.sh: 1: /var/jenkins_home/workspace/acs-hotel-call-center@tmp/durable-8e4744af/script.sh: docker: not found

- which docker
```bash
which docker
/usr/bin/docker
```
```bash
docker run \
  -u root \
  -d \
  --rm \
  -p 8081:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v $(which docker):/usr/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --name jenkins-docker \
  jenkinsci/blueocean
```
<span style=color:red>ERROR:</span>

[Pipeline] { (Declarative: Agent Setup)
[Pipeline] isUnix
[Pipeline] readFile
[Pipeline] sh
+ docker build -t 30dbf6d4e5c162cb60517da6e481709d3804c4fa -f Dockerfile .
/var/jenkins_home/workspace/acs-hotel-call-center-test@tmp/durable-1afeda3b/script.sh: line 1: docker: not found

```bash
docker exec -it jenkins-docker bash
docker -v
bash: /usr/bin/docker: No such file or directory
```

- wrong user?
```bash
docker run \
  -d \
  --rm \
  -p 8081:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v $(which docker):/usr/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --name jenkins-docker \
  jenkinsci/blueocean
```
<span style=color:red>ERROR:</span>

- sudu
```bash
sudo docker run \
  -d \
  --rm \
  -p 8081:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v $(which docker):/usr/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --name jenkins-docker \
  jenkinsci/blueocean
```
<span style=color:red>ERROR:</span>Doesn't even create a container.

- dif user
sudu
```bash
sudo docker run \
  -u auser \
  -d \
  --rm \
  -p 8081:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v $(which docker):/usr/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --name jenkins-docker \
  jenkinsci/blueocean
```
<span style=color:red>ERROR:</span>docker: Error response from daemon: linux spec user: unable to find user auser: no matching entries in passwd file.

- privileged
```bash
docker run \
  --privileged \
  -d \
  --rm \
  -p 8081:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v $(which docker):/usr/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --name jenkins-docker \
  jenkinsci/blueocean
```
<span style=color:red>ERROR:</span>docker: Error response from daemon: linux spec user: unable to find user auser: no matching entries in passwd file.

- sudo privileged
```bash
docker run \
  --privileged \
  -d \
  --rm \
  -p 8081:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v $(which docker):/usr/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --name jenkins-docker \
  jenkinsci/blueocean
```
<span style=color:red>ERROR:</span>docker: Error response from daemon: linux spec user: unable to find user auser: no matching entries in passwd file.

- pass docker **GID******
```bash
docker run \
  -u $(grep docker /etc/group) \
  -d \
  --rm \
  -p 8081:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v $(which docker):/usr/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --name jenkins-docker \
  jenkinsci/blueocean
```
<span style=color:red>ERROR:</span>docker: Error response from daemon: linux spec user: unable to find user docker: no matching entries in passwd file.

- Commented # USER root in Dockerfile
```bash
docker run \
  -u root \
  -d \
  --rm \
  -p 8081:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v $(which docker):/usr/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --name jenkins-docker \
  jenkinsci/blueocean
```
<span style=color:red>ERROR:</span>docker: not found

[Pipeline] sh
+ java -version
openjdk version "1.8.0_191"
OpenJDK Runtime Environment (build 1.8.0_191-8u191-b12-2ubuntu0.18.04.1-b12)
OpenJDK 64-Bit Server VM (build 25.191-b12, mixed mode)
[Pipeline] sh
+ mvn -v
Apache Maven 3.2.3 (33f8c3e1027c3ddde99d3cdebad2656a31e8fdf4; 2014-08-11T20:58:10+00:00)
Maven home: /opt/apache-maven-3.2.3
Java version: 1.8.0_191, vendor: Oracle Corporation
Java home: /usr/lib/jvm/java-8-openjdk-amd64/jre
Default locale: en_US, platform encoding: ANSI_X3.4-1968
OS name: "linux", version: "3.10.0-514.26.2.el7.x86_64", arch: "amd64", family: "unix"
[Pipeline] sh
+ node --version
v10.15.2
[Pipeline] sh
+ npm --version
6.4.1
[Pipeline] sh
+ docker -v
/var/jenkins_home/workspace/acs-hotel-call-center-test@tmp/durable-177413d4/script.sh: 1: /var/jenkins_home/workspace/acs-hotel-call-center-test@tmp/durable-177413d4/script.sh: docker: not found

```bash
docker exec -id jenkins-docker bash
docker -v
Docker version 18.09.1-ce, build 4c52b901c6cb019f7552cd93055f9688c6538be4
```

- Add auser to Dockerfile
```bash
docker run \
  -u root \
  -d \
  --rm \
  -p 8081:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v $(which docker):/usr/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --name jenkins-docker \
  jenkinsci/blueocean
```

# AuserJenkins
```bash
docker run \
  -d \
  --rm \
  -p 8081:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v $(which docker):/usr/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --name jenkins-docker \
  dockerhub.mycompany.net:5000/myapp
```
<span style=color:red>ERROR:</span>docker: Error response from daemon: linux spec user: unable to find user auser: no matching entries in passwd file.

- -u root
```bash
docker run \
  -u root \
  --rm \
  -p 8081:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v $(which docker):/usr/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --name jenkins-docker \
  dockerhub.mycompany.com:5000/myapp
```
<span style=color:red>ERROR:</span>docker: error while loading shared libraries: libltdl.so.7: cannot open shared object file: No such file or directory

Running from: /usr/share/jenkins/jenkins.war
[Pipeline] stage
[Pipeline] { (Declarative: Agent Setup)
[Pipeline] isUnix
[Pipeline] readFile
[Pipeline] sh
+ docker build -t 71f51e796272a3359a6686cdcd673dafc69ca695 -f Dockerfile .
docker: error while loading shared libraries: libltdl.so.7: cannot open shared object file: No such file or directory

- --user $(id -u)
```bash
docker run \
  --user $(id -u) \
  --rm \
  -p 8081:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v $(which docker):/usr/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --name jenkins-docker \
  dockerhub.mycompany:5000/myapp
```
<span style=color:red>ERROR:</span>touch: cannot touch '/var/jenkins_home/copy_reference_file.log': Permission denied Can not write to /var/jenkins_home/copy_reference_file.log. Wrong volume permissions?

- --user $(id -u)
```bash
docker run \
  --user $(id -u) \
  --privileged \
  --rm \
  -p 8081:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v $(which docker):/usr/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --name jenkins-docker \
  dockerhub.mycompany.com:5000/myapp
  ```
<span style=color:red>ERROR:</span>touch: cannot touch '/var/jenkins_home/copy_reference_file.log': Permission denied
Can not write to /var/jenkins_home/copy_reference_file.log. Wrong volume permissions?

docker run \
  --rm \
  -p 8081:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v $(which docker):/usr/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --name jenkins-docker \
  jenkinsci/blueocean


mkdir $PWD/jenkins
sudo chown -R 1000:1000 $PWD/jenkins
docker run -d -p 8080:8080 -p 50000:50000 -v $PWD/jenkins:/var/jenkins_home --name jenkins jenkins

mkdir $HOME/jenkins_home
sudo chown -R 1000:1000 $HOME/jenkins_home
docker run -d -p 8080:8080 -p 50000:50000 -v $HOME/jenkins_home:/var/jenkins_home --name jenkins jenkins
docker run \
  --privileged
  --rm \
  -p 8081:8080 \
  -v $HOME/jenkins_home:/var/jenkins_home \
  -v $(which docker):/usr/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --name jenkins-docker \
  jenkinsci/blueocean
ERROR:
```bash
docker -v
bash: /usr/bin/docker: No such file or directory
```

- Privileged
```bash
docker run \
  --privileged \
  -d \
  --rm \
  -p 8081:8080 \
  -v $HOME/jenkins_home:/var/jenkins_home \
  -v $(which docker):/usr/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --name jenkins-docker \
  jenkinsci/blueocean
```
ERROR:
```bash
docker -v
bash: /usr/bin/docker: No such file or directory
```

sudo chmod -R 777 /home/swateek/tmp/jenkins/data
sudo docker run --rm --privileged -p 8080:8080 -p 50000:50000 -v /home/swateek/tmp/jenkins/data:/var/jenkins_home jenkins

mkdir /home/ronit/jenkins_home
chown 1000:1000 /home/ronit/jenkins_home
docker run --privileged --name jenkins-1 -p 8080:8080 -p 50000:50000 -v /home/ronit/jenkins_home:/var/jenkins_home jenkins

--------------------------------------------------------------------------------------------------

### /var/run/docker.sock
```bash
docker run \
  -u root \
  -d \
  --rm \
  -p 8081:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --name jenkins-docker \
  jenkinsci/blueocean
```
docker exec -it jenkins-docker bash
docker -v
<span style=color:green>SUCCESS:</span>Docker version 18.09.1-ce, build 4c52b901c6cb019f7552cd93055f9688c6538be4

$ docker run \
>   -u root \
>   -d \
>   --rm \
>   -p 8081:8080 \
>   -v jenkins-data:/var/jenkins_home \
>   -v /var/run/docker.sock:/var/run/docker.sock \
>   -v "$HOME":/home \
>   --name jenkins-docker \
>   jenkinsci/blueocean
3e36684ef4543e624c4deaa429af2b3d5a5d9d4adfa52232df67c22f6b9d969f
$ docker ps -a
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                                              NAMES
3e36684ef454        jenkinsci/blueocean   "/sbin/tini -- /us..."   3 seconds ago       Up 2 seconds        50000/tcp, 0.0.0.0:8081->8080/tcp                  jenkins-docker
fd75fdb58189        jenkins               "/bin/tini -- /usr..."   14 months ago       Up 2 months         0.0.0.0:8080->8080/tcp, 0.0.0.0:50000->50000/tcp   focused_spence
 docker exec -it jenkins-docker bash
bash-4.4# docker -v
Docker version 18.09.1-ce, build 4c52b901c6cb019f7552cd93055f9688c6538be4
bash-4.4#

<span style=color:ref>ERROR:</span>+ docker -v
/var/jenkins_home/workspace/acs-hotel-call-center-test@tmp/durable-5c3aa0ba/script.sh: 1: /var/jenkins_home/workspace/acs-hotel-call-center-test@tmp/durable-5c3aa0ba/script.sh: docker: not found

### Add Docker to Dockerfile
```bash
docker run \
  -u root \
  -d \
  --rm \
  -p 8081:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --name jenkins-docker \
  jenkinsci/blueocean
```
docker exec -it jenkins-docker bash
docker -v
<span style=color:green>SUCCESS:</span>Docker version 18.09.1-ce, build 4c52b901c6cb019f7552cd93055f9688c6538be4

 docker run \
>   -u root \
>   -d \
>   --rm \
>   -p 8081:8080 \
>   -v jenkins-data:/var/jenkins_home \
>   -v /var/run/docker.sock:/var/run/docker.sock \
>   -v "$HOME":/home \
>   --name jenkins-docker \
>   jenkinsci/blueocean
3e36684ef4543e624c4deaa429af2b3d5a5d9d4adfa52232df67c22f6b9d969f
 docker ps -a
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                                              NAMES
3e36684ef454        jenkinsci/blueocean   "/sbin/tini -- /us..."   3 seconds ago       Up 2 seconds        50000/tcp, 0.0.0.0:8081->8080/tcp                  jenkins-docker
fd75fdb58189        jenkins               "/bin/tini -- /usr..."   14 months ago       Up 2 months         0.0.0.0:8080->8080/tcp, 0.0.0.0:50000->50000/tcp   focused_spence
 docker exec -it jenkins-docker bash
bash-4.4# docker -v
Docker version 18.09.1-ce, build 4c52b901c6cb019f7552cd93055f9688c6538be4
bash-4.4#

<span style=color:green>SUCCESS:</span>
+ docker -v
Docker version 18.09.3, build 774a1f4

docker run \
  -u root \
  -d \
  --rm \
  -p 8080:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --name jenkins-docker \
  jenkinsci/blueocean

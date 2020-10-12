# python_flask
A Python web application using Flask framework, using Ngninx for reverse proxy.
For deployment of these services I used DockerCompose as an orchestrator.
#### flask service overview:

```
    flask:
        build:
            context: ./app/
            dockerfile: Dockerfile
        image: zivmit/py-docker-images:latest
        container_name: py_flask
        networks:
            our-network:
                aliases:
                    - py_flask
        ports:
            - 8080:5000
networks:
    our-network:
```
Docker-compose will build a container named py_flask and expose the service on ports 8080 (outside the container) and 5000 (inside the container).
The ngnix container will use the `our-network` for communication with py_flask container.
Docker-compose will build the image from this Dockerfile:
it will copy the application code into te container, and install requirements(flask) via pip3.

```
FROM python:3.6.1-alpine
WORKDIR /usr/src/app
COPY services/app/ /usr/src/app
RUN pip3 install -r requirements.txt
CMD ["python3","api.py"]
```

#### Nginx creation overview:
```
services:
    nginx:
        image: nginx:1.17-alpine
        container_name: nginx
        depends_on:
            - py_flask
        volumes:
            - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf
        networks:
            - our-network
        ports:
            - 80:80
````
Nginx container is depend on py_flask container which means it will up just after the py_flask container will be up.
I mounted a ngnix.conf file that configured to be a proxy for py_flask service. 

```
server {
    listen 80;
    server_name localhost;
location / {
        proxy_pass http://py_flask:5000/;
        proxy_set_header Host "localhost";
    }
}
```
I created a Jenkinsfile that will trrigerd when pushing a new code version, the job will build a DockerImage and push it to DockerHUb.
You can see the Jenkinsfile here:
https://github.com/zivmitrani/python_flask/blob/master/services/Jenkinsfile 



links I used:
Build a flask app:
- https://www.tutorialspoint.com/flask/flask_quick_guide.htm
- https://medium.com/@pemagrg/build-a-web-app-using-pythons-flask-for-beginners-f28315256893
- https://levelup.gitconnected.com/using-nginx-reverse-proxy-with-flask-and-docker-66854e940176 
- https://docs.docker.com/compose/compose-file/
- https://pythonise.com/series/learning-flask/building-a-flask-app-with-docker-compose#nginx

# Secound part: setup a Jenkins server
Repo link: https://github.com/zivmitrani/Jenkins_docker

## Jenkins_docker
Jenkins server with Ngnix for reverse-proxy.
I created a docker compose file which create the Jenkins Server as a container listening on 8080 default port, and 50000(slave will access via this port).
I also used Nginx for making the service accessible on port 80 for the world (it was onli accessible from localhost).

#### Jenkins creation overview:
```
version: '3.7'
services:
  jenkins:
    image: jenkins/jenkins:lts
    privileged: true
    user: root
    ports:
      - 8080:8080
      - 50000:50000
    container_name: jenkins
    networks:
      - docker-network
    volumes:
      - ~/jenkins:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
      - /usr/local/bin/docker:/usr/local/bin/docker

```
Docker-copmose will create a container named jenkins, from the official and latest image 'jenkins/jenkins:lts' it is defined as a privileged container so it can access to the host devices and use them.
I defineded a dokcer-network that will use for nginx & jenkins contaniers to work together and recognize each other.
Mounthed volumes to the jenkkins container: 
```- ~/jenkins:/var/jenkins_home ```
/var/jenkins_home will use the path ~/jenkins on my server to persist all data: configurations, plugins, pipelines, passwords, etc.
```- /var/run/docker.sock:/var/run/docker.sock
   - /usr/local/bin/docker:/usr/local/bin/docker  
```
These volumes will used Jenkins server to use docker inside.


#### Nginx creation overview:
```
  nginx:
    image: nginx
    container_name: nginx
    depends_on:
      - jenkins
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    networks:
      - docker-network
    ports:
      - 80:80
networks:
    docker-network:
```
Docker-compose will create an Ngninx container that will make the Jenkins server accessable from the world.
It will use the official and latest image of nginx.
Nginx container is depend on Jenkins conainer- while use the docker-compose up command it will wait  and just after Jenkins container is up, it will up too.
I mounted a local nginx.conf that I configured:

```
server {
    listen 80;
    server_name ${jenkins-server-host} ;
location / {
        proxy_pass http://jenkins:8080;
       # Fix the "It appears that your reverse proxy set up is broken" error.
       # proxy_pass          http://127.0.0.1:8080;
        proxy_read_timeout  90;
        proxy_set_header Host "${jenkins-server-host}";
    }
}

```
Nginx container will use the docker-network for communitation with the jenkins-server container.
Nginx is accessible from port 80 and will route the requests to jenkins container on port 8080.

Run the command:
running the Jeknins-server
$ docker-compose up -d 
Get the administrator password to log in the first time and unlock jenkins.
$ docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

#### configuraing a Slave server
I created an EC2 instance ( type t2.micro ) that will use as a Jenkins slave.
steps:
- Install the nceccesary packages:
```
sudo yum update -y
sudo yum install java-1.8.0-openjdk
sudo yum install -y docker
sudo usermod -a -G docker ec2-user
```

- Create a user on the slave to be used by Jenkins, define the path /var/lib/jenkins as home path.
```
sudo useradd -d /var/lib/jenkins jenkins
passwd jenkins # change to any password
```

- Generate an ssh key
```
su - jenkins
ssh-keygen -t rsa -C "jenkins agent key"
```
– Add the public SSH key id_rsa.pub to the list of authorized_keys file:
```
 cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
 chmod 600 ~/.ssh/authorized_keys
```
- Copy the private SSH key ~/.ssh/id_rsa: (will use the Jenkins server)
`cat ~/.ssh/id_rsa`

- add the user Jenkins to the docker-gourp so it will be able to execute docker binary.
```
sudo groupadd docker
sudo usermod -a -G docker jenkins
``` 

On Jenkins UI:
- add the slave private key to jenkins credentials
Manage Jenkins -> Manage Nodes and Clouds -> New Node.
You can follow this detailed guide:
https://yallalabs.com/linux/how-to-add-linux-slave-node-agent-node-jenkins/ 


### Define Github webhook for Jenkins
Push some new code to repo and Jenkins will run build without an intervention.
You can follow this detailed guide:
https://k6.io/blog/bootstrap-your-ci-with-jenkins-and-github

Turorials I used:
- https://dev.to/andresfmoya/install-jenkins-using-docker-compose-4cab
- http://web.archive.org/web/20190723112236/https://wiki.jenkins.io/display/JENKINS/Jenkins+behind+an+NGinX+reverse+proxy

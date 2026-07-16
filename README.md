# test-repo
https://skillbuilder.aws/category/getstarted  
AWS Cloud Practitioner Essentials 12h 45m  
https://aws.amazon.com/getting-started/hands-on/  
https://aws.amazon.com/getting-started/  
https://docs.aws.amazon.com/eks/  
https://builder.aws.com/learn  
https://builder.aws.com/learn/topics/kubernetes  
https://aws.amazon.com/whitepapers/  
https://docs.aws.amazon.com/whitepapers/latest/aws-overview/introduction.html  
https://skillbuilder.aws/  
AWS+EKS bootcamp: thu-fri-sat-sun 6.25-6.25-6.25-6.25 = 25 hours 9-5 focussed (excl. lunch + school, no talking)  
rancher yaml-container-Kubernetes, Linux foundation Kubernetes intro, skillbuilder AWS, skillbuilder EKS, Kubernetes.io docs, aws eks docs, w3schools AWS
docker, terraform, GitHub, GitHub Actions, GitHub Copilot, jFrog Artifactory, ArgoCD, chatGPT DevOps prompt, DevSecOps book, API Gateway, CI/CD pipelines  
docker for beginners (3 hours)

objectives:

understand containers and docker
run a docker container
build your own docker image
networking and docker compose
docker registry, how to deploy your private registry
how docker works under the hood
orchestration: swarm and Kubernetes
docker sandbox
docker desktop
podman
containerd, runc, oci standard

docker installation
https://docs.docker.com/install/linux/docker-ce/ubuntu/
docs.docker.com -> manuals -> left pane "docker engine" -> install -> debian
systemctl status docker
docker run hello-world

docker version
docker images
docker ps -a
docker run nodejs
docker stop <container id or container name>
docker pull
docker compose up
docker compose down
docker login
docker service
docker system prune
docker rm <name of stopped container>
docker rmi <name of image>
docker exec 538d cat /etc/hosts (538d is firs few chars of container id)
docker run -d kodecloud/simple-webapp
docker attach a043d
docker run alpine sleep 10

hub.docker.com

images: debian, nodejs, alpine, nginx
registries: docker hub, ghcr.io, google, aws, azure
limit 200 pulls per 6 hours


kubectl get pods
kubectl run nginx --image=nginx

image tags
interactive mode
docker run -it kodecloud/simple-prompt-docker
cat schema.sql | docker run -i postgres psql -U postgres -d mydb
-i alone (stdin, no tty) to pipe data into the container
docker run -t alpine ls --color=always /
-t alone (tty, no stdin) for colored output or progress bars


port mapping
docker run -p 80:5000 kodecloud/webapp (maps port 80 of docker host to port 5000 of container)
http://localhost:80

volumes (key=value pairs)
docker run --mount type=bind,source=/opt/datadir,target=/var/lib/mysql MySQL
docker run -v /opt/datadir:/var/lib/mysql MySQL

inspect & logs
docker inspect blissful_hopper
docker logs 3550f5


Dockerfile

FROM ubuntu

RUN apt-get update

RUN apt-get install -y python3-flask

COPY app.py /opt/app.py

ENV FLASK_APP=/opt/app.py

ENTRYPOINT flask run --host=0.0.0.0


docker build -t my-custom-app .
docker push mmumshad/my-custom-app
docker history mmushad/my-custom-app --no-trunc
docker run -p 5000:5000 my-custom-app

docker run -it ubuntu
apt-get update
apt-get install -y python3-flask
cat > app.py (copy & paste, then ctrl+c)
export FLASK_APP=/opt/app.py
mv app.py /opt/
flask run --host=0.0.0.0

run, don't install

ENTRYPOINT vs CMD
multistage


from flask import Flask
app = Flask(__name__)

@app.route("/")
def main():
   return "Welcome!"

@app.route('/how-are-you')
def hello():
   return 'I am good, how about you?'

if __name__ == "__main__":
   app.run(host="0.0.0.0", port=5000)

http://localhost:5000/how-are-you

# syntax=docker/dockerfile:1

RUN --mount=type=cache,targe=/root/.cache/pip \
    pip install flask flask-MySQL

RUN --mount=type=secret,id=myey \
    curl =H "Authorization: Bearer $(cat /run/secrets/mykey)" \
    https://api.example.com/private

cat ./key.txt
docker buildx build --secret id=mykey,src=./key.txt -t webapp .


docker buildx build -t webapp .

docker buildx build --platform Linux/amd64,linux/arm64 -t mmumshad/webapp --push .

cd my-new-app
docker init
generates 4 files: Dockerfile, .dockerignore, compose.yaml, and README.Docker.md

ENTRYPOINT ["sleep"]
CMD ["5"]

docker run ubuntu-sleeper
docker run ubuntu-sleeper 10
docker run --entrypoint sleep2.0 ubuntu-sleeper 10

with entrypoint, the command line argument gets appended to the command
with cmd, the command line parameters replaces the command entirely

docker run -e APP_COLOR=blue simple-webapp-color

https://github.com/dockersamples/example-voting-app

docker run -d --name=redis --network voting-app redis:alpine

docker network create vote-net
compose creates a default network for you
docker network ls

docker compose -f docker-stack.yml up
docker compose -f docker-stack.yml down

https://docs.docker.com/compose/
https://docs.docker.com/engine/reference/commandline/compose/
https://github.com/dockersamples/example-voting-app

yaml:


docker engine -> dockerd (handles images, containers, volumes, networks), REST API (programs, tools, integrations), docker cli (docker run, docker stop, docker rmi etc)
below dockerd, there is containerd, and below it is runc (namespaces, cgroups)
docker cli can be on another system too, in which case you use -H option for remote host & port
docker -H=10.123.2.1:2376 --tlsverify run nginx
or
docker ssh://user@host


docker run --privileged -d ubuntu
now you can install docker inside docker container

docker run --cpus=0.5 --memory=500m ubuntu

/var/lib/docker
containers/
image/
volumes/
overlay2/

docker volume create datavolume
/var/lib/docker/volumes/datavolume/

volume mount from /var/lib/docker/volumes
bind mount from any host path

networking:

bridged, host, and none
172.17.0.2, 172.17.0.3 etc
docker network create \
    ---driver bridge \
    --subnet 182.118.0.0/16 \
    my-custom-isolated-network
docker network ls

default docker bridge network needs ip addresses only
the network that you create (user defined), name resolution works due to embedded DNS running on 127.0.0.11

docker network create --driver bridge my-network
docker run -d --name=mysql --network=my-network MySQL
docker run -d --name=web --network=my-network my-webapp

docker registry

docker run nginx
docker.io/library/nginx
ghcr.io
<account>.dkr.ecr.<region>.amazonaws.com
<name>.azurecr.io
*-docker.pkg.dev

docker login private-registry.io
docker run private-registry.io/apps/internal-app

docker run -d -p 5000:5000 --name registry registry:2


run your own registry:

docker image tag my-image localhost:5000/my-image
docker push localhost:5000/my-image
docker pull localhost:5000/my-image


orchestration:

gke (google cloud), aks (azure), eks (aws)


docker swarm init (run on manager)
docker swarm join (run on nodes/workers)
docker swarm init --advertise-addr 192.168.1.12 (run on manager)
docker swarm join --token xxx-x-xxxx 192.168.1.12:2377 (command generated automatically, run on each node)
docker service create --replicas=3 nodejs (run on manager node)
or
docker service create -e MODE=prod -p 80:80 --network web --replicas=3 my-web-server (run on manager node)


architecture:

kubernetes cluster consists of a set of nodes (physical or virtual machines)
master is the node with the kubernetes control plane components installed
controle plane -> api-server (front end), etcd (distributed reliable key-value store), scheduler (places pods on nodes), controllers
nodes have container runtime (containerd or CRI-O; earlier this used to be docker) that runs containers, and kubelet (agent)

kubectl cluster-info
kubectl get nodes
kubectl get pods
kubectl create deployment my-web-server --image=my-web-server --replicas=3


yaml:

<servers>
  <server>
    <name>Server1</name>
    <owner>John</owner>
  </server>
</servers>

{
  "servers": [
    { "name": "Server1",
      "owner": "John" }
    ]
  }

servers:
  - name: Server1
    owner: John

yaml:

key: value

Fruit: Apple
Vegetable: Carrot

arrays (lists):

Fruits:
  - Orange
  - Apple
  - Banana

Vegetables:
  - Carrot
  - Cauliflower
  - Tomato

dictionaries (maps):

Banana:
  Calories: 105
  Fat: 0.4 g
  Carbs: 27 g

Grapes:
  Calories: 62
  Fat: 0.3 g
  Carbs: 16 g

lists of dictionaries:

# list of fruits (this line is a comment)
Fruits:
  - Banana:
      Calories: 105
      Fat: 0.4 g
      Carbs: 27 g

  - Grapes:
      Calories: 62
      Fat: 0.3 g
      Carbs: 16 g

important: dictionary is an unordered collection, whereas a list is an ordered collection



main purpose of docker is to package and containerize applications, and to ship them, and to run them, anywhere, anytime, and as many times as you want

vm vs container
vm is usually gb in size, dontainer is usually mb in size
image vs container: image is like a template, and containers are running instances of image (1 to many relationship)
you can create your own image and push it to image repository, then create your own containers







git & GitHub crash course:

create a repository
cloning
how to see differences between files
staging
committing
pushing
PR
fork

create a GitHub account
press "New repository" on web UI
repo = a place to store your code or stuff
only public repo creation is free, private repo creation is paid

echo "# example-repo" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/shrikantpise/example-repo.git
git push -u origin master

install git on computer
apt install git
git --version

mkdir git-practice; cd git-practice
git clone <url from GitHub.com> example-repo
cd example-repo
sublime
https://github.com/KalobTaulien/example-repo
git status
git add README.md (this is staging)
git status
git commit -m "first commit" (still on local computer)
git status
git push origin main (this places on GitHub)
modify README.md and create a new file index.html
git status
git add index.html
git status
git diff README.md
git add README.md
git status
git commit -m "second commit"
git log
git push origin main
git checkout README.md (the unchanged file)
git checkout <hash obtained from git log command)

git log --topo-order --all --graph --date=local --pretty=format:'%C(green)%h%C(reset) %><(55,trunc)%s%C(red)%d%C(reset) %C(blue)[%an]%C(reset) %C(yellow)%ad%C(reset)%n'

Here's the free code snippet, and what your .gitconfig file would look like when you add it to your global git config

https://codingforeverybody.com/snippets/git-lg

staging sets up files to be committed (git add folder/filename.html)
committing means they take staged files and slap a "this is done" label on it before sending it to GitHub
pushing means sending the files from your computer to GitHub

do the same from vscode directly
GitLab.com
kalob.io/blog
kalob.io/courses

https://www.diagrams.net/ (formerly draw.io)


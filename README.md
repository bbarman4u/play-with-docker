# play-with-docker

Play With Docker gives you the experience of having a free Alpine Linux Virtual Machine in the cloud
where you can build and run Docker containers and even create clusters with Docker features like Swarm Mode.

Under the hood DIND or Docker-in-Docker is used to give the effect of multiple VMs/PCs.

A live version is available at: http://play-with-docker.com/

Notes:

* There is a hard-coded limit to 5 Docker playgrounds per session. After 4 hours sessions are deleted on the live hosted site. On Self hosted sites, it expires in 24 hours.
* Only http, https, websocket and SSH connections are allowed.
* If you want to override the DIND version or image then set the environmental variable i.e.
  `DIND_IMAGE=franela/docker<version>-rc:dind`. Take into account that you can't use standard `dind` images, only [franela](https://hub.docker.com/r/franela/) ones work.

## Getting Started With Self Hosted Version
## On Mac
#### Software Requirements for Mac
* [Docker `18.06.0+`](https://docs.docker.com/install/)
* [dnsmasq on Mac](https://formulae.brew.sh/formula/dnsmasq)
* [franela dind](https://hub.docker.com/r/franela/dind)
* A modern browser like Chrome, Firefox, Safari

#### Steps for Starting Up PWD on Mac
- Initialize docker swarm and then clone the git repo and start the PWD stack with docker compose
  ```
  # Verify the Docker daemon is running else restart docker engine or docker desktop 
  docker run hello-world

  # Ensure Docker daemon is running in swarm mode
  docker swarm init

  # Clone this forked git repo or the main git repo
  git clone https://github.com/play-with-docker/play-with-docker
  cd play-with-docker

  # Get the latest franela/dind image
  docker pull franela/dind

  # Start PWD as a container
  docker-compose up -d

  ```
- Now navigate to [http://localhost](http://localhost) and click the green "Start" button
to create a new session, followed by "ADD NEW INSTANCE" to launch a new terminal instance or choose any of the predefined templates to quickly launch the sessions.

- If you deploy any service and you need to browse it, please open the appropriate port from the session and browse it.

- In order for port forwarding to work correctly in development/local machine you need to make `*.localhost` to resolve to `127.0.0.1`. That way when you try to access to `pwd10-0-0-1-8080.host1.localhost`, then you're forwarded correctly to your local PWD server.

- You can achieve this by setting up a `dnsmasq` server (you can run it in a docker container also) and adding the following configuration(for detailed steps for implementing in Mac[refer here](https://hedichaibi.com/how-to-setup-wildcard-dev-domains-with-dnsmasq-on-a-mac/):

```
address=/localhost/127.0.0.1
```


Don't forget to change your computer default DNS to use the dnsmasq server to resolve.

## On Linux
#### Requirements for Linux

* [Docker `18.06.0+`](https://docs.docker.com/install/)
* [Go](https://golang.org/dl/) (stable release)

#### Start Up on Linux

```bash
# Clone this repo locally
git clone https://github.com/play-with-docker/play-with-docker
cd play-with-docker

# Verify the Docker daemon is running
docker run hello-world

# Load the IPVS kernel module. Because swarms are created in dind,
# the daemon won't load it automatically
sudo modprobe xt_ipvs

# Ensure Docker daemon is running in swarm mode
docker swarm init

# Get the latest franela/dind image
docker pull franela/dind

# Optional (with go1.14): pre-fetch module requirements into vendor
# so that no network requests are required within the containers.
# The module cache is retained in the pwd and l2 containers so the
# download is a one-off if you omit this step.
go mod vendor

# Start PWD as a container
docker-compose up
```

Now navigate to [http://localhost](http://localhost) and click the green "Start" button
to create a new session, followed by "ADD NEW INSTANCE" to launch a new terminal instance.



### Port forwarding

In order for port forwarding to work correctly in development you need to make `*.localhost` to resolve to `127.0.0.1`. That way when you try to access to `pwd10-0-0-1-8080.host1.localhost`, then you're forwarded correctly to your local PWD server.

You can achieve this by setting up a `dnsmasq` server (you can run it in a docker container also) and adding the following configuration:

```
address=/localhost/127.0.0.1
```

Don't forget to change your computer default DNS to use the dnsmasq server to resolve.

## FAQ

### How can I connect to a published port from the outside world?

If you need to access your services from outside, use the following URL pattern `http://ip<hyphen-ip>-<session_jd>-<port>.direct.labs.play-with-docker.com` (i.e: http://ip-2-135-3-b8ir6vbg5vr00095iil0-8080.direct.labs.play-with-docker.com).

### Why is PWD running in ports 80 and 443?, Can I change that?.

No, it needs to run on those ports for DNS resolve to work. Ideas or suggestions about how to improve this
are welcome

### How can SSH into a running host ?
- Generate an SSH Key Pair(if not already existing). e.g. on Linux or Mac, run ssh-keygen and fill out name of file and empty passphrase.
- Connect to the target docker host as follows: Run ssh -i <File name entered in first step> <docker ssh address>. Example: ssh -i ida_rsa ip172-18-0-9-bfe8s7iv9dig00cuhdsg@direct.labs.play-with-docker.com
- Note: For Hosted Instances, SSH Port is 22, For Self Hosted instance started from the compose file, by default it is mapped to 8022
  - e.g. `ssh -p 8022 ip10-0-3-4-c32gd7np3hegju4q3tn0@direct.localhost`
- If connected to the hosted play with docker instance

# Docker Installation Instructions For  [Zorin OS 15.2](https://zorinos.com/) (Based on Ubuntu 18.04 Bionic Beaver)
## Commands come from [Syed Komail Abbas'](https://hackernoon.com/dont-install-postgres-docker-pull-postgres-bee20e200198) and [Brian Hogan's](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04) Docker tutorials

### Add GPG key for official Docker repo 
`curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`

Flags:  
`-f` Fail silently and don't show HTTP errors  
`-s` Don't show progress  
`-S` Show error if curl fails  
`-L` Allow for redirects  


### Add the Docker Stable repo to a .list file in /etc/apt/sources.list.d/
`echo 'deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable' | sudo tee --append /etc/apt/sources.list.d/docker.list`

Note:  
`tee` allows for reading from standard input and writing to a file.  `--append` flag wil not overwrite the file if it already exists and will just append to the file. 

	
Doing something like `sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"` is also an option, which will add the repo to `/etc/apt/sources.list`.  I like having the repos in individual .list files in case I need to change something or just need to get rid of the repo, so I don't have to go in the `sources.list` file to look for the specific repo.  
	
### Update your package database to pull in the new Docker repo and make sure there are no errors
`sudo apt update`

### Ensure you're installing from the Docker repo and not Ubuntu default repo
`apt-cache policy docker-ce`

Output will look something like the below: 
```
docker-ce:
  Installed: (none)
  Candidate: 18.03.1~ce~3-0~ubuntu
  Version table:
     18.03.1~ce~3-0~ubuntu 500
        500 https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
```

### Install Docker Community Edition
`sudo apt install -y docker-ce`

### Check Docker staus And make sure it's running
`sudo systemctl status docker`

Output: 
```
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: 
   Active: active (running) since Fri 2020-05-03 17:43:31 PDT; 11min ago
     Docs: https://docs.docker.com
 Main PID: 8324 (dockerd)
    Tasks: 12
   Memory: 40.3M
   CGroup: /system.slice/docker.service
           └─8324 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/contai

May 31 17:43:31 <youruser> dockerd[8324]: time="2020-05-03T17:43:31.366671980-07:00
May 31 17:43:31 <youruser> dockerd[8324]: time="2020-05-03T17:43:31.366682440-07:00
May 31 17:43:31 <youruser> dockerd[8324]: time="2020-05-03T17:43:31.367023487-07:00
May 31 17:43:31 <youruser> dockerd[8324]: time="2020-05-03T17:43:31.368620032-07:00
May 31 17:43:31 <youruser> dockerd[8324]: time="2020-05-03T17:43:31.529253221-07:00
May 31 17:43:31 <youruser> dockerd[8324]: time="2020-05-03T17:43:31.655852930-07:00
May 31 17:43:31 <youruser> dockerd[8324]: time="2020-05-03T17:43:31.797542770-07:00
May 31 17:43:31 <youruser> dockerd[8324]: time="2020-05-03T17:43:31.797631727-07:00
May 31 17:43:31 <youruser> systemd[1]: Started Docker Application Container Engine.
May 31 17:43:31 <youruser> dockerd[8324]: time="2020-05-03T17:43:31.821129140-07:00
```

### Add your user to the Docker group to avoid having to use `sudo` each time with Docker
`sudo usermod -aG docker ${USER}`

### Confirm that your user is part of the Docker group
`id -nG`

Flags:  
`-n` Print a name instead of a number  
`-G` Print the group IDs

Output:   
`<youruser> adm sudo docker`

### Pull down [Geographica's](https://github.com/GeographicaGS/Docker-PostGIS) Postgres/PostGIS [Docker image](https://hub.docker.com/r/geographica/postgis).  At the time this image was using PostgreSQL 12.2 PostGIS 3.0.1 GEOS 3.8.0, PROJ 6.3.1.  
`docker pull geographica/postgis`

### Check and make sure the above is in your Docker images
`docker images ls`

### Create a data directory for your container so the data is persistent (othewise data is lost once the container is shutdown)
`mkdir -p $HOME/docker/volumes/postgres`

Flags:  
`-p` Will create directory if it doesn't exist and won't complain with errors (even if it does already exist)

### Startup the new Docker container from your downloaded image
`docker run --rm --name <name for your container> -e POSTGRES_PASSWORD=<password> -d -p 5432:5432 -v $HOME/docker/volumes/postgres:/var/lib/postgresql/data geographica/postgis:latest`

Flags:  
`--rm`  Removes the container and its file system once container has been shutdown. It's generally regarded as good practice to pass this flag and if data needs to be persisten the `--v` flag can be passed.  
`--name`  Name you would like for the container to identify it.    
`--e`  Expose the `POSTGRES_PASSWORD` varible so we can set a a password for the DB.   
`-d`  Launch the container in the background.    
`-p` Bind port on local host to port in container.  So in this instance port 5432 on local host is bound to 5432 of the container.  Port 5432 should be available on your local host but may not be if you're already running a PostgreSQL instance.  You can use something like `-p 5433:5432` if this is the case.  
`-v` Mount the newly created `$HOME/docker/volumes/postgres` dir to the /`var/lib/postgresql/data` dir of the container so the data is persistent.  

### Determine IP of your container so you can connect to it with [pgAdmin](https://www.pgadmin.org/) or something like [DBeaver](https://dbeaver.io/).  Depends on your preference if you want a stand-alone desktop application (DBeaver) or one that lanuches in a browser (pgAdmin)
`docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' pg-docker`

### 
Launch your preferred DB tool to connect with and connect to your database with the IP from above. 
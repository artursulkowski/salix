# Home Assistant development - Tips & Tricks
Here I gather information related to setting up development and test environment.  
My development configuration:
* Mac M1 (ARM architecture)
* [Podman](https://podman.io/) used as [Docker Desktop](https://www.docker.com/products/docker-desktop/) replacement
* Development is done using VS Code Studio and [Dev Container](https://code.visualstudio.com/docs/devcontainers/containers)

## How to use Podman as replace of Docker Desktop
I use [Podman](https://podman.io/) as a replacement for [Docker Desktop](https://).  
[Run Docker without Docker Desktop on macOS](https://dhwaneetbhatt.com/blog/run-docker-without-docker-desktop-on-macos) presents explanation about element of "Docker ecosystem". Lists which of them are open source and which (e.g. Docker Desktop) are proprietary solutions.

As reference instruction I've used second part (related to M1) of [Goodbye Docker Desktop, Hello Minikube!](https://itnext.io/goodbye-docker-desktop-hello-minikube-3649f2a1c469)

### Podman = Docker (almost) 
podman can be used as a direct replacement for docker.  
I simply use:
``` bash
alias docker=podman
```  

Installing podman:

``` bash
brew install podman
```

Add podman helper 
``` bash 
sudo /opt/homebrew/Cellar/podman/4.3.1/bin/podman-mac-helper install
```


### Creating VM for Podman 
Podman (similarly to Docker Desktop on macOS) needs to create a VM in which containers can be spin up

``` bash
podman machine init --cpus 6 --memory 12288 --disk-size 50 --image-path next
podman machine set --rootful
podman machine start
```

VM can be start and stop: 
``` bash
podman machine start
podman machine stop
```

Adjust `devcontainer.json` in Home Assistant

Add root as user: 
``` json
"dockerFile": "../Dockerfile.dev",
"remoteUser": "root", // <= added
"postCreateCommand": "script/setup",
```  

### Maximum open files error
Default configuration in podman and Home Assistant devcontainer has limited number of maximum open files (in my case it was 1024).  
This resulted in unstable operations and multiple `too many open files` errors.  

I've increased number of maximum open files:

* Change VM setting
    1. Connect to podman machine 
        ```
        podman machine ssh
        ```
    2. Change limit of maximum open files
        ``` 
        ulimit -Sn 200000
        cat /proc/self/limits
        ```


* Modify `devcontainer.json` by adding `--ulimit=host`
    ``` json
    "runArgs": ["--ulimit=host","-e", "GIT_EDITOR=code --wait"],
    ```

## Debugging
Using devcontainer I can debug Home Assistant code.  
Additional change in `configuration.yaml` was needed:
```yaml
debugpy:
  start: true
  wait: false
```

## Recorder - SQLite configuration
Initially [Recoder](https://www.home-assistant.io/integrations/recorder/) could not start.  
By default it's using SQLite was trying to create `/workspaces/core/config/home-assistant_v2.db`.  File was created but SQlite was failing at reading it.  
I've worked around this problem by changing location of DB file - adding in `configuration.yaml`: 
```yaml
recorder:
   db_url: sqlite:////workspaces/home-assistant_as.db
```
Still it's not a final solution as data is lost after re-creation of container. However it allows me to progress with work. 

- [ ] To fix location of storing SQlite DB.





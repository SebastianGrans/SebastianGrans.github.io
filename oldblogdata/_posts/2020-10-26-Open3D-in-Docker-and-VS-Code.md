---
layout: post
title: Open3D in Docker and VS Code
img: /images/2020-10-26-Open3DDocker/open3ddocker.png
---

<style>
.mono {
  font-family: monospace;
}

</style>


<center>
<img src="{{ site.baseurl }}/images/2020-10-26-Open3DDocker/open3ddocker.png" alt="Open3D, CUDA, and Chrome running in Docker" style="display: block;"/>
</center>
<div markdown="1" style="display:block; background-color:#eff0f1; border-radius: 3px; padding: 5px 20px; margin: 10px;">
**TL;DR:** An example setup with a `Dockerfile` and `devcontainer.json` is available on my [Github](https://github.com/SebastianGrans/Open3DinDocker)
</div>

Getting GUI applications to run in a Docker container is relatively straight forward. Some methods are more safe than others, but [ROS wiki](http://wiki.ros.org/docker/Tutorials/GUI) has you covered. 

Instead of changing the access permission to the X server, a better method is to mount all the relevant user information files (`passwd`, `shadow`, etc.) into the Docker container and log in as your regular user. This user already has access permission to the X server. 

```bash
docker run -it \
    --user=$(id -u $USER):$(id -g $USER) \
    --env="DISPLAY" \
    --volume="/etc/group:/etc/group:ro" \
    --volume="/etc/passwd:/etc/passwd:ro" \
    --volume="/etc/shadow:/etc/shadow:ro" \
    --volume="/etc/sudoers.d:/etc/sudoers.d:ro" \
    --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw"
    yourImage
```

> **Note:** Some images don't come with sudo installed, so don't forget to add that to your `Dockerfile`.

Implementing this in VS Code turned out to be more difficult than one would expect. 

VS Code tries to create a `.vscode-server` folder inside your home folder, but your home folder doesn't exist. It tries to create this folder, but since it is running as your default user, it doesn't have write permission in the `/home/`-folder, it fails.

After a lot of trail and error I figured out that the folder is automatically created if your mount the workspace in your container home folder. Like this:
```
    "-v", "${localWorkspaceFolder}:/home/grans/", 
    "--workdir", "/home/grans/"
```

But then my next issue appeared!  

Let's say that you use `zsh` on your host machine, then the `passwd` file will containa line like this:
```
user:x:1000:1000:user,,,:/home/user:/bin/zsh
```
But containers don't typically come with `zsh` installed, so VS Code will throw an error:
```
The terminal process failed to launch: Path to shell executable "/bin/zsh" does not exist.
```
You could of course add `zsh` to your Dockerfile, but that is going against the philosophy that Docker images should be kept lean. 

How do we fix it?  
Add to your `devcontainer.json` file:
```json 
"settings": { 
    "terminal.integrated.shell.linux": "/bin/bash"
}
```
Now one would expect it to work after `Rebuild and Reopen in container`, but don't be fooled. The only way I could make it work was to delete the `.vscode` folder and then restart the container! I've made an [issue](https://github.com/microsoft/vscode-remote-release/issues/3900) over at Github.

Maybe my use case is just so unique that these kind of issues haven't appeared before. 
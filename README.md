## What

This repo lets you build and run [lean theorem prover](https://leanprover.github.io/)
(version 3) in vscode. Lean is built in a self-contained debian-based docker
environment so nothing is installed on the host system. With a little
configuration (given here) you can use the [Remote
Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
extension to use this docker container as the main development environment in
vscode.

Some possible benefits:

- Build and run lean purely from within vscode with minimal configuration required!
- Any system that runs docker and vscode should work and give a consistent experience.
  - Tested on archlinux and macOS Catalina. Windows should probably work too.
- Self-contained lean installation that does not install into your host system.

## How

### Requirements

- Make sure you are using the MS build of vscode. If you are on a linux
  distro be aware you may be using an OSS build and you may not be able to use
  Remote Containers extension. If you install it directly as .vsix file, the
  extension will probably break when trying to launch your container. So stick
  with the MS build.
- Install Microsoft's [Remote
  Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
  extension in vscode.
- Ensure your system is running docker (eg `docker ps` should work)

### Setup

Clone this repo somewhere and then copy the `.devcontainer/` folder into your lean project.

For example:

```
  cd projects/
  mkdir my-lean-project
  git clone https://github.com/danielbush/lean-remote-containers.git # this is outside my-lean-project btw
  cp -a lean-remote-containers/.devcontainer my-lean-project/ # linux / macOS
  code my-lean-project
```

vscode should notice that you have a `.devcontainer/` and may prompt you to
open the remote container. Initially vscode will be in "local" mode and will
not launch the container.

### Checks

Now let's check it's working...

We can switch between Local and Container environments using 2 commands.
vscode will reopen each time.

- `ctrl/cmd+shift+p` + `Remote-Containers: Reopen in Container`
  - makes lean container the main development environment in vscode
    - **NOTE:** The first time you open in container, it may take a few
      minutes to build. (This will build the image for the `Dockerfile` in
      `.devcontainer/`.) Subsequent sessions should be almost instant.
- `ctrl/cmd+shift+p` + `Remote-Containers: Reopen Locally`
  - puts vscode back into your local environment again.
- Exiting vscode appears to exit the container.
- The `Remote-Containers: Rebuild and Reopen in Container` command rebuilds
  the MS part of the lean docker image. Note this won't rebuild lean. See
  further down to rebuild the whole lean docker image to get the latest lean.

**IMPORTANT:** You will want your container shell to load its profile when
opened in vscode's integrated terminal in order to set the correct paths for
lean:

- Open Settings UI in vscode (usually `ctrl/cmd + ,` in vscode)
- Search for "shell arg"
- Add `-l` to **Terminal › Integrated › Shell Args: Linux**

  - this makes vscode run `bash -l` when opening the terminal which should
    load `~/.bash_profile` inside the container
  - if you don't do this, you could manually source this file

- Open the integrated terminal in vscode and check the following commands

```
  type lean
  # => /home/lean/.elan/bin/lean
  type leanproject
  # => /home/lean/.local/bin/leanproject
  echo $HOME
  # => /home/lean
  pwd
  # => /workspaces/my-lean-project
  ls # should list the contents of my-lean-project
```

If you got here you're hopefully set.

Note:

- Your terminal should open in your **project workspace** in the container
  (ie `/workspaces/my-lean-project`). Your project workspace is accessible on
  both your host system and in the docker container.
- Your **home directory** in the lean container is `/home/lean` where some of
  the software is stored / configured. It exists purely within the container.

If this is a new project, you can now run

```
  # In /workspaces/my-lean-project
  leanproject new .
```

to install a mathlib project.

Yay, time to do some math!

![lean running as vscode remote container](https://github.com/danielbush/lean-remote-containers/blob/master/docs/lean-with-remote-containers.png?raw=true)

### Cleaning up and Upgrading

The Remote Containers extension in vscode will create several docker images
and docker containers. To completely rebuild your lean container so as to get
the latest version of lean, you can delete these and then re-open vscode.
There might be a better way to do this, but I can't see it in the docs.

lean container(s) and image(s) currently have names like `vsc-your-project-name-xxx`.

First, exit vscode. This will hopefully exit the lean container.

Then check what you have:

```
docker ps -a  # should show all containers
docker images # look for vsc-*
```

Now delete all exited containers and `vsc-*` images:

```
docker container prune # you may prefer to filter
docker images 'vsc-*' --format '{{.Repository}}' | xargs docker rmi
```

When you re-open vscode, the image will hopefully rebuild from scratch.

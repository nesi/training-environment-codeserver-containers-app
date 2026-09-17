# REANNZ training environment Containers VS Code app

The **VS Code** tile for the containers workshop on the NeSI/REANNZ
[training environment](https://github.com/nesi/training-environment): `code-server` with
[Apptainer](https://apptainer.org/) installed.

`code-server` runs with `--auth password` on port 8443, and the connect button posts to its login
page through the `/rnode` proxy, following
[training-environment-vscode-intro2nextflow-app](https://github.com/nesi/training-environment-vscode-intro2nextflow-app).
It opens in `~/containers-workshop`, where the workshop examples and the prebuilt chapter 2
containers are. Apptainer is used from VS Code's built-in terminal.

This repo has the same structure as
[training-environment-jupyter-containers-app](https://github.com/nesi/training-environment-jupyter-containers-app)
and builds its own image. The workshop has three tiles, each its own repo, because Open OnDemand
installs one app per git repo:

| Tile | Repo |
| --- | --- |
| JupyterLab | [training-environment-jupyter-containers-app](https://github.com/nesi/training-environment-jupyter-containers-app) |
| Terminal | [training-environment-terminal-containers-app](https://github.com/nesi/training-environment-terminal-containers-app) |
| VS Code | this one |

The three *docker/Dockerfile* files are the same, so a change to one usually belongs in all three,
as does a bump of *docker/workshop-version.txt*.

## Requirements in the training environment

`apptainer build --fakeroot` needs two things from the environment:

* `enable_privileged_pods: true` in *vars/ondemand-config.yml* — `apptainer-suid` will not work
  without it
* `/etc/subuid` and `/etc/subgid` from the worker node, bind mounted into the pod by
  *submit.yml.erb*. The training-environment `container-apps/k8s` role populates these for the
  training and trainer users.

## How a session is set up

*template/before.sh.erb* sources `find_host_port` and `save_passwd_as_secret` from */bin* (the ood
k8s utils, baked into the image) and exports `host`, `port` and `password`. `code-server` reads
`PASSWORD` from the environment, and *view.html.erb* posts the same value to its login page.
*template/after.sh.erb* waits for the port to open before the session is marked ready.

## Apptainer settings

*template/script.sh.erb* exports these before starting code-server, so its terminals inherit them:

* `APPTAINER_CACHEDIR=$HOME/.apptainer/cache` — pulled images persist across sessions
* `APPTAINER_TMPDIR=/tmp/apptainer-$USER` — builds use the pod's local disk, not the NFS home
  directory, which is slow and can fail for builds

## Workshop examples

This image ships no workshop content, and does not need to. The training environment provisions the
[workshop examples](https://github.com/nesi/reannz-containers-workshop) into every user's home
directory at deploy time (the `app-data/containers-workshop` role, controlled by
`provision_data_containers_workshop`), with the chapter 2 containers already built. Home
directories are shared over NFS, so they are already in `~/containers-workshop` when a session
starts, the same as they are in a terminal on the web node. `code-server` opens in that directory.

## Releasing a new version

1. Update the version in `script.native.container.image` in *submit.yml.erb*, commit it
2. `git tag -a v0.1.2 -m "..."` and `git push --tags`
3. Check the *Actions* tab — the workflow builds and pushes the image to ghcr.io
4. Update `k8s_container` and `version` for this app in *vars/ondemand-config.yml* in the
   training-environment repo

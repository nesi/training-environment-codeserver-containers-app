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

The examples come from the [workshop repo](https://github.com/nesi/reannz-containers-workshop),
pinned by commit in *docker/workshop-version.txt*, baked into the image at
*/opt/containers-workshop/examples* and rsynced into `~/containers-workshop` at startup with
`--ignore-existing`, so learners' edits survive a restart. The chapter 9 MPI examples are dropped,
since they are written for a Slurm cluster.

The two [chapter 2](https://nesi.github.io/reannz-containers-workshop/setup-containers/#2-the-basics-of-running-containers-on-apptainer)
containers ship prebuilt. They are built by *.github/workflows/build_container.yml* **on the runner**,
not in the docker build, because building a `.sif` needs mount privileges that a `RUN` step does not
have. The workflow drops them into *docker/prebuilt/*, which the Dockerfile moves into the chapter 2
example directory. Building the image by hand leaves that directory empty, which is fine — the
definition files are all still there.

## Releasing a new version

1. Update the version in `script.native.container.image` in *submit.yml.erb*, commit it
2. `git tag -a v0.1.2 -m "..."` and `git push --tags`
3. Check the *Actions* tab — the workflow builds and pushes the image to ghcr.io
4. Update `k8s_container` and `version` for this app in *vars/ondemand-config.yml* in the
   training-environment repo

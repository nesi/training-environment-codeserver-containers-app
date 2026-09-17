# REANNZ training environment Apptainer VS Code app

The **VS Code** tile for the containers workshop on the NeSI/REANNZ
[training environment](https://github.com/nesi/training-environment): `code-server` with
[Apptainer](https://apptainer.org/) installed.

There is no Dockerfile here. This app runs the image built by
[training-environment-jupyter-containers-app](https://github.com/nesi/training-environment-jupyter-containers-app),
which is also used by the JupyterLab and Terminal tiles — one image to build and pre-pull, three app
definitions. Every Open OnDemand dashboard tile has to be its own git repo, which is why this repo
exists.

The app definition follows
[training-environment-vscode-intro2nextflow-app](https://github.com/nesi/training-environment-vscode-intro2nextflow-app):
`code-server` runs with `--auth password`, listening on port 8443, and the connect button posts to
its login page through the `/rnode` proxy. It opens in `~/containers-workshop`, where the workshop
examples and the prebuilt chapter 2 containers are.

Apptainer is used from VS Code's built-in terminal, which inherits `APPTAINER_CACHEDIR` and
`APPTAINER_TMPDIR` from *template/script.sh.erb*.

## Keeping it in step

When a new version of the image is released in the containers-app repo, update
`script.native.container.image` in *submit.yml.erb* here to match, and the `version` of this app in
*vars/ondemand-config.yml* in the training-environment repo.

## Requirements in the training environment

`apptainer build --fakeroot` needs `enable_privileged_pods: true` in *vars/ondemand-config.yml*, and
the `/etc/subuid` and `/etc/subgid` mounts that *submit.yml.erb* sets up.

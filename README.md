# CollabTainer
This is intended to be a starter example container to house various apps which might
work together on common tasks. Apps might take outputs of other apps for interactive
visualiation. An app might connect other apps together in a pipeline. The container
might also support multiple contributers, working together in a modular way. Each app
might be installed with a separate set of dependency versions, using Conda or Julia
Project environments. This particular container bootstraps a container provisioned
with Julia, Micromamba, and R.

## Create Your Own Container
Click "Use this template", above, to create your own repository based on this working example.
You can explore this code, and maybe try it out after building a local [Apptainer](https://apptainer.org/docs/user/latest/quick_start.html) image. But then you'll want to remove the directories under [scif/apps](scif/apps) and edit [recipe.scif](recipe.scif) to serve your purposes. Don't forget to update the [.github/workflows/build_docker.yml](.github/workflows/build_docker.yml) file to add your own container location and name. That will trigger when you push a tag of the form "vx.x.x" (semantic versioning pattern). You'll need to create and set some repository secrets before that will work, though.
(https://github.com/settings/tokens)
(https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions#creating-secrets-for-a-repository)


## Contribute to a Container
First, make sure your software . . .
- has a basic input/output interface, to control what input(s) it needs and where it should send any output
- has some specification for it's top-level dependencies (e.g [enviroment.yml](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#create-env-file-manually), [requirements.in](https://github.com/jazzband/pip-tools/blob/main/examples/django.in), etc.)

If it's just scripts, edit the app's `%appinstall` section of the [scif recipe file](recipe.scif) to clone
the code into the app's `lib` directory. Install any files which can be called as exectuables into the app's
`bin` directory. 

## [Scientific Filesystem](https://sci-f.github.io) (SCIF)
(Vanessa Sochat; The Scientific Filesystem (SCIF), GigaScience, giy023, https://doi.org/10.1093/gigascience/giy023)

This container leverages SCIF. The [scif tool](https://pypi.org/project/scif/) is used
to install and utilze apps under a simple, standard directory structure, under
a `/scif` directory in the created container. There is also a [scif directory](scif) in
this repository. That is only to contain as-needed files which we might want to copy over
to app-specific locations. But this is **not** meant to contain all the files and directories
to instantiate the full scif directory structure. The scif tool will do that, based on the
[scif recipe file](recipe.scif). The repo's scif directory is a good place to keep small
app config files, like Conda `enviroment.yml` files or Julia `Project.toml` files. But it is encouraged
to pull in as many things via reliable reference. E.g. (shallow) clone a Github repo into an app's
lib directory, via `%appinstall` sections in the recipe file.

## Explore the apps Interface
The scif tool is used as the entrypoint to interface with the container apps. You can take a look
at what is available by just calling `--help` on at built container.
E.g,
```bash
# apptainer build CollabTainer.sif Apptainer
./CollabTainer.sif --help
usage: scif [-h] [--debug] [--quiet] [--writable] {version,pyshell,shell,preview,help,install,inspect,run,test,apps,dump,exec} ...

scientific filesystem tools

options:
  -h, --help            show this help message and exit
  --debug               use verbose logging to debug.
  --quiet               suppress print output
  --writable, -w        for relevant commands, if writable SCIF is needed

actions:
  actions for Scientific Filesystem

  {version,pyshell,shell,preview,help,install,inspect,run,test,apps,dump,exec}
                        scif actions
    version             show software version
    pyshell             Interactive python shell to scientific filesystem
    shell               shell to interact with scientific filesystem
    preview             preview changes to a filesytem
    help                look at help for an app, if it exists.
    install             install a recipe on the filesystem
    inspect             inspect an attribute for a scif installation
    run                 entrypoint to run a scientific filesystem
    test                entrypoint to test an app in a scientific filesystem
    apps                list apps installed
    dump                dump recipe
    exec                execute a command to a scientific filesystem
```
Get a list of the apps:
```bash
./CollabTainer.sif apps
py_data_viz
julia_data
 r_epinow2
     julia
```

Run one of the apps:
```bash
> ./CollabTainer.sif run julia
[julia] executing /bin/bash /scif/apps/julia/scif/runscript
Julia Version 1.9.3
Commit bed2cd540a1 (2023-08-24 14:43 UTC)
Build Info:
  Official https://julialang.org/ release
Platform Info:
  OS: Linux (x86_64-linux-gnu)
  CPU: 32 × Intel(R) Xeon(R) Silver 4110 CPU @ 2.10GHz
  WORD_SIZE: 64
  LIBM: libopenlibm
  LLVM: libLLVM-14.0.6 (ORCJIT, skylake-avx512)
  Threads: 1 on 32 virtual cores
Environment:
  JULIA_PATH = /usr/local/julia
  LD_LIBRARY_PATH = /scif/apps/julia/lib:/.singularity.d/libs
```

. . . and . . .
```bash
> ./CollabTainer.sif run julia_data
[julia_data] executing /bin/bash /scif/apps/julia_data/scif/runscript
names(Arrow) = [:Arrow, :ArrowTypes]
```

. . . and . . .
```bash
> ./CollabTainer.sif run py_data_viz
[py_data_viz] executing /bin/bash /scif/apps/py_data_viz/scif/runscript
running micromamba run --name py_data_viz python lib/show_altair.py
module location: /opt/conda/envs/py_data_viz/lib/python3.11/site-packages/altair/__init__.py
```

. . . finally . . .
```bash
> ./CollabTainer.sif run r_epinow2 | head -n 2
[r_epinow2] executing /bin/bash /scif/apps/r_epinow2/scif/runscript
running Rscript lib/show_packages.R
> ./CollabTainer.sif run r_epinow2 | grep EpiNow2 | grep site-library
EpiNow2        "EpiNow2"        "/usr/local/lib/R/site-library" "1.4.0"
```
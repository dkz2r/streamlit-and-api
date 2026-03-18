---
title: "UV and the Environment"
teaching:  # teaching time in minutes
exercises: 0 # exercise time in minutes
---

:::::::::::::::::::::::::::::::::::::: questions

- What is a Python Package Manager?
- What is a Virtual Environment?
- What is uv and how does it compare to pip and conda?
- How do I install uv and create a new Python project?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Create a new Python project using uv
- Add dependencies to a project with uv

::::::::::::::::::::::::::::::::::::::::::::::::

## Python Package Management and Virtual Environments

There are a number of tools already out there for managing Python packages. You may already be
familiar with `pip` and `conda`. These tools are great for installing packages - you can easily
send someone a `requirements.txt` file and they can install the same packages with
`pip install -r requirements.txt`. And that's great! They can install all the same packages that
you have and run your code! But what if they are working on a big project that's stuck on an older
version of Python? Or what if they have a different version of a package that your code relies on?

This is where virtual environments come in. A virtual environment is a self-contained directory
that contains all the packages and dependencies for a specific project. This allows you to have
different projects with different dependencies and Python versions on the same machine without
conflicts.

There are several tools you might have heard of for creating and managing virtual environments:

- `venv`
- `virtualenv`
- `conda`
- `pipenv`
- `poetry`
- `pyenv`

These tools all have strengths and weaknesses - for example, `venv` is built into Python and is
simple to use. `conda` is great for managing complex dependencies and different Python versions,
but it can be slow and has its own ecosystem of packages.

We're going to use a tool called `uv` that is one of the more recent entries into this space. `uv`
is written in Rust and is designed to be a fast, all-in-one Python package and project manager. It
provides a unified workflow for managing packages, virtual environments, and Python versions, and
can be easily installed using `pip` or platform-specific installers.

## Installing UV

If you don't have `uv` installed yet, you can find detailed installation instructions in the
[uv documentation](https://docs.astral.sh/uv/getting-started/installation/). For this workshop,
we will be using the `pip` package manager to install `uv` into our base environment:

```bash
python -m pip install --user uv
```

```
Collecting uv
  Downloading uv-0.10.11-py3-none-win_amd64.whl.metadata (12 kB)
Downloading uv-0.10.11-py3-none-win_amd64.whl (24.2 MB)
   ---------------------------------------- 24.2/24.2 MB 153.4 MB/s  0:00:00
Installing collected packages: uv
Successfully installed uv-0.10.11
```

(The exact version ma differ by the time you read this.) After the installation is complete, you
can verify that `uv` is installed correctly by running:

```bash
$ uv --version
```

And you should see output similar to:

```
uv 0.10.11
```

::: callout

### Troubleshooting

If you get a message like "bash: uv: command not found":

1. Try restarting the terminal to refresh the PATH environment variable.
2. Try running the command `python -m pip install --user uv` again - if the installation was
   successful, it should say "Requirement already satisfied" and provide you with a path to the
   installed package. Navigate to this folder, then look for a `Scripts` subfolder. Inside, you
   should find the `uv` executable. You can add this folder to your PATH environment variable to
   make the `uv` command available globally.

:::

## Starting a new project

Now that we have `uv` installed, let's start our project. Navigate to the folder where you want to
create your project and run:

```bash
uv init
```

`uv` will automatically create six files in the current directory:

```
my-project/
├── .git
├── .gitignore
├── .python-version
├── main.py
├── pyproject.toml
└── README.md
```

Handy! Let's walk through the files that were created:

- `.git` and `.gitignore` are for version control with Git. `uv` essentially just ran `git init`
    for us and created a basic `.gitignore` file that ignores common Python artifacts like
    `__pycache__` and  `*.pyc` files.
- `.python-version` is a file that specifies the Python version for this project. This is used by
    `uv` to create a virtual environment with the correct Python version.
- `main.py` is a starter Python script that we can run to verify that our environment is set up
    correctly.
- `pyproject.toml` is a configuration file that specifies the project metadata and dependencies. We
    will look closer at this in a minute.
- `README.md` is a markdown file that provides an introduction to the project.

The main.py file is just a hello world kind of script. Let's run it using uv instead of python:

```bash
uv run main.py
```

You should see some output like this:

```
Using CPython 3.13.7
Creating virtual environment at: .venv
Hello from my-project!
```

And if you look in the project folder, you'll see that a new `.venv` folder has been created. This
is the virtual environment for this project. `uv` automatically created it for us when we ran the
`uv run` command. This is one of the key benefits of using `uv` - it handles virtual environment
creation and management for us, so we don't have to worry about it.

## Adding Dependencies

We know for this project, that we want to use the `streamlit` package. With `uv`, we can add this
dependency to our project with `uv add {package-name}`. So let's run:

```bash
uv add streamlit
```

You should see a bunch of lines as `uv` starts downloading the various dependencies of `streamlit`,
and finally a message like this:

```
Resolved 39 packages in 563ms
Prepared 38 packages in 35.75s
Installed 38 packages in 2.36s
 + altair==6.0.0
 + attrs==25.4.0
 + blinker==1.9.0
...
```

If we check the `pyproject.toml` file, we can see that there's a new section of the file:

```toml
dependencies = [
    "streamlit>=1.55.0",
]
```

This is where `uv` keeps track of our project dependencies. When we run `uv add streamlit`, it
adds `streamlit` to this list of dependencies. There's also a new file called `uv.lock` that was
created. This is a lockfile that contains the exact versions of all the packages that were
installed, along with their dependencies. This is useful for ensuring exact reproducibility of our
environment - all we have to do in order for someone to create the exact same environment that we
are developing in is to have them clone our repository and run `uv sync`.

We have a couple additional dependencies that we need to add for our project, so let's add those
now:

```bash
uv add pandas requests python-dotenv
```

And with that, we're ready to start building a Streamlit app!


::::::::::::::::::::::::::::::::::::: keypoints

- We can use `uv` to create a new Python project with `uv init`.
- We can add dependencies to our project with `uv add {package-name}`.
- `uv` automatically creates and manages a virtual environment for our project, aiding with
    reproducibility and avoiding conflicts between projects.

::::::::::::::::::::::::::::::::::::::::::::::::

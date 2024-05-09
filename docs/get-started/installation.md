# Installation
This document describes how to install RAG Application.

We recommend you create a virtual environment for dependency isolation. See the [Conda documentation](https://www.anaconda.com/download) or the [Python documentation](https://docs.python.org/3/library/venv.html) for details.

## Prerequisites
- [Python](https://www.python.org/downloads/) and pip installed.
- [Conda](https://www.anaconda.com/download) for virtual environment management.  
- [Docker](https://www.docker.com/) for simplify software application deployment processes.

## Environment Setup

Clone the repo using git:

```shell
git clone https://github.com/Miciox5/ICOS.git
```

Create and activate a new virtual environment:

```shell
conda create -n ICOS python=3.10.0
conda activate ICOS
```

Install the dependencies:

```shell
pip install -r requirements.txt
```

!!! warning
    If you encounter an error while building a wheel during the `pip install` process, you may need to install a C++
    compiler on your computer.
# Data analysis of tile beacons

## Getting started

1. Have [Conda](https://docs.conda.io/en/latest/). If you don't have it yet, install it as [Miniconda](https://docs.conda.io/en/latest/miniconda.html). You don't need admin credentials, and it's actually better to have Conda installed as user.
1. Open a terminal, go to the project's directory, and create the Conda environment for the project:

        conda env create

1.  From there, you have two possibilities.
    1.  Are you not running Jupyter Lab yet? Then you can start your own.

            conda activate iot-hunters
            jupyter lab

        If you are running this on your own workstation, this will open a web browser window with Jupyter for you. If you are running this in some remote VM, you now want to forward port local 8888 (or whatever you prefer) to the port that Jupyter's server binded on. Then copy and paste the URL printed on the terminal.
    1.  Are you using Jupyter out of a machine giving you access through JupyterHub? Then you want to expose our new Conda environment as a [Jupyter](https://docs.jupyter.org/en/latest/projects/kernels.html) [kernel](https://ipython.org/). There are two possibilities in play once again.
        1. The first one is that the owners of the machine have deployed the [nb\_conda\_kernels](https://github.com/Anaconda-Platform/nb_conda_kernels) package, which will automate the transformation of the environment to a kernel. Just open a notebook from the project, and play with the kernel change menu for up to two minutes, or until the kernel option corresponding to your environment appears.
        1. If that fails, we must manually export the environment as a kernel using the following command (**without** activating your environment, so as to use the `jupyter` command associated to JupyterHub):

                python -k ipykernel install --user --display-name iot-hunters --name iot-hunters
1. Every time you start a new shell or terminal that requires the use of our Conda computing environment (to run a script, for instance), your must first *activate* it like you would to start Jupyter Lab.

        conda activate iot-hunters

    Remark that you can do that from any directory.


## Notebook index

[Distinguishing tiles in one's pocket from tiles in the wild ](tiles-in-pocket-tiles-in-the-wild.ipynb)

> Answers the question as to whether we can discern nearby tags from remote tags using signal strength.


## Best practices

- Never commit a notebook with cell outputs in it: always use **Edit | Clear All Outputs** to first evacuate anything that might capture raw data into the notebook file.
- Always look at the data! Use histograms. Eyeball the data frames. Get a sense for what is there.
- Use `tqdm_notebook` to distinguish between a long-running computation and an endless loop.
- Put your data access credentials in a file named `.env` in the root directory of the project, and use [`dotenv`](https://github.com/theskumar/python-dotenv) to load them into the running kernel. Never hardcode access credentials into code.

# Spinning up a Python Data Science Environment on Mac ARM
So I got a 'day one' m1 Macbook Air (i.e. an Apple Silicon ARM Mac) and I had to do some work to spin up my Data Science environment... and do it well. I'm going to leave my notes here in the hopes it (1) helps me not lose these and (2) helps someone out on the internet.

# Rosetta Terminal
Rosetta is a way to emulate Intel/x86 Software on your ARM processor. There are two ways to use Rosetta in your terminal

1. Add `arch -x86_64` before your terminal commands OR
2. [Recommended: Just create a terminal that always launches in Rosetta](https://osxdaily.com/2020/11/18/how-run-homebrew-x86-terminal-apple-silicon-mac/)

Once you do that, you can install Homebrew using Rosetta and get access to most of its packages.

Note that not everything compiles in Rosetta and you should be wary of it. I have not tested the world of software but of note, Docker is one of the most prominent examples that will not work with Rosetta. Scroll down to the very last section to see the list of all programs that won't work at all.

---

# Homebrew & Apple Command Line Tools
Homebrew now has native support for Apple Silicon as of Version 3.0 and you can follow the  default installation at https://brew.sh. This'll install to `/opt/homebrew/bin/brew`. Bonus: The ARM native version now installs the Apple Command Line Tools for you for free.

Even if Apple Command Line Tools + Homebrew comes with Python, and they do, I still recommend going out of your way to install Miniforge below to get the complete Conda ecosystem.

---

# Python/R/Fortran (i.e. Anaconda/Miniforge)
So normally I'd use Anaconda but Anaconda is only optimized for Intel Macs at the type this guide was written. So I used Miniforge which fortunately has an ARM equivilant and basically wraps Conda + Python in a happy ARM architecture.

### How should I install Python/Conda on ARM?
To install Miniforge, use the installer link at https://github.com/conda-forge/miniforge#miniforge3 and then install it in your normal terminal. Remember not to run this in a Rosetta Terminal, if you created one, because this installer script can run in your regular ARM terminal. 

After you do this, to ensure you truly got an ARM version of Python, verify this:
1. Type `which python` in your terminal. It should point to your MiniForge version, which for me, was at `/Users/YOURUSERNAMEHERE/miniforge3/bin/python`
2. Launch Python in your Terminal by running `python`
3. Then go to Activity Monitor.
4. Search for Python.
5. Under the architecture column, it should say 'Apple' (if you don't see an architecture column, right click on that toolbar with all the columns and add the architecture column).

### What packages are available on ARM?
That said even though it has Conda, that doesn't mean all packages are ARM ready -- only some are. The quick way to see what packages may be available for ARM are if you:

1. See a package made by conda-forge (https://anaconda.org/conda-forge) AND
2. That package has an architecture of `osx-arm64` or the package is `noarch` and all its dependencies are built.

If your package meets both requirements, you can install it by running `conda -c conda-forge <PACKAGE>` once you have Miniforge installed.

### Things you can definately get on an ARM Python
Note this doesn't mean these are ARM 'optimized' to use all of the fancy GPUs and Neural Cores. It just means unlike Intel, these aren't emulated so they can run on the CPU natively. That said, take all these with a grain of salt, I haven't got the chance to put all of these packages to their paces so some functionality may still be broken.

- Bokeh (Added on 11/23/2020)
- JupyterLab & Notebooks: The latest version of `appnope` (updated around 11/30/2020) has fixed a Ctypes bug on ARM, which now permits Jupyter to run on ARM.
- Matplotlib
- Numpy
- Pandas
- PyTorch by manually installing the latest version from source (i.e. you cannot use `pip` or `conda` at this time, Nov 2020, to install it). More on this below.
- Scikit-Learn (Added on 11/21/2020)
- Scipy
- Statsmodel (Added on 11/21/2020)
- TensorFlow by manually installing Apple's version (more on this below)
- R language (`conda install r-base` for a minimal R installation or `conda install r` for R plus recommended packages)
- Fortran compiler (`conda install gfortran`)

### Things that kinda or totally don't work
- Keras: Just no.
- PyTorch: Yes, PyTorch is in the above list of working software, but it has side packages such as TorchVision that isn't ARM Compatible quite yet. You can install it using Rosetta, under a Rosetta translated version of Python, pretty easily though.
- TensorFlow: Yes, TensorFlow is also in the list above, but it's also here because while Apple's version works, it is currently just forked from the real TensorFlow. The officially updated TensorFlow that you can get from `conda` or `pip` has not been ARM updated and thus these paths could diverge unless Apple and TensorFlow work together a bit closer.

### If you prefer Anaconda
If you must use Anaconda because a package isn't ARM ready, you can use Rosetta to emulate an Intel Anaconda as per the tips in the [Homebrew Section](homebrew-and-rosetta-terminal). You can then use said terminal to install Anaconda. Remember that just because you can emulate Anaconda doesn't mean you will be able to successfully run every package.

### Python 3.8 downgrade
Many packages, including Apple's TensorFlow, prefers Python 3.8 over the Python 3.9 that Miniconda comes with. Remember that is still an option with Miniforge, you can downgrade by creating a new environment using something like this: `conda create --name python38 python=3.8`.

### Conda Version
Each recipe for Python Packages appear to flip-flop between wanting Conda version 4.9.0 or something newer. If you have to jump around to make things work, remember the command is `conda install conda=PUT_VERSION_HERE`.

### TensorFlow
[Apple has created a TensorFlow](https://github.com/apple/tensorflow_macos) that is optimized for their ARM Processors and GPUs. I wrote the instructions to install this on TensorFlow for Apple and these instructions are at https://github.com/apple/tensorflow_macos/issues/153.

### PyTorch
At the time this section was written, Nov 27 2020, PyTorch had a merge request that has been merged to add ARM Compilation (https://github.com/pytorch/pytorch/pull/48275) but this has not been oficially released. This means, to use PyTorch, you must compile PyTorch on your own. Furthermore even if you go down this route, several PyTorch 'side' packages do not work, such as TorchVision.

If you still want to install PyTorch, you'll want to first install Python using Miniforge linked above. After doing that, you should be able to follow the 'Build from Source' instructions at https://github.com/pytorch/pytorch#from-source. I don't think I needed Homebrew nor anything from it, but if you do, I am using a 'Rosetta compiled' version of Homebrew.

---

## Software that is kind of ARM Ready
- [MATLAB](https://www.mathworks.com/matlabcentral/answers/641925-is-matlab-supported-on-apple-silicon-macs?s_tid=srchtitle): The latest version can run under Rosetta, with a few exceptions for some packages. An ARM Version is in development but there's no release date as of yet.
-  [Visual Studio Code](https://code.visualstudio.com/insiders/): There is an ARM version but only for the 'Beta' Insiders Track which is linked here. The regular version works perfectly well using Rosetta emulation.

## Software that is not ARM ready at all
- [Docker](https://www.docker.com/blog/expanding-dockers-developer-preview-program/): Update as of 12/10, there is a Developer Preview Program to test an Apple Silicon Docker.

# Spinning up a Python Data Science Environment on Mac ARM
So I got a 'day one' m1 Macbook Air (i.e. an Apple Silicon ARM Mac) and I had to do some work to spin up my Data Science environment... and do it well. I'm going to leave my notes here in the hopes it (1) helps me not lose these and (2) helps someone out on the internet.

# Rosetta Terminal
Rosetta is a way to emulate Intel/x86 Software on your ARM processor. There are two ways to use Rosetta in your terminal

1. Add `arch -x86_64` before your terminal commands OR
2. [Recommended: Just create a terminal that always launches in Rosetta](https://osxdaily.com/2020/11/18/how-run-homebrew-x86-terminal-apple-silicon-mac/)

Once you do that, you can install Homebrew using Rosetta and get access to most of its packages.

# Homebrew & Apple Command Line Tools
To learn more about Homebrew & Apple's Command Line Tools (to setup Apple's compilers which they used to compile C and other code), check out https://github.com/mikelxc/Workarounds-for-ARM-mac.

---

# Python (i.e. Anaconda/Miniforge)
So normally I'd use Anaconda but Anaconda is only optimized for Intel Macs at the type this guide was written. So I used Miniforge which fortunately has an ARM equivilant and basically wraps Conda + Python in a happy ARM architecture.

To install Miniforge, use the installer link at https://conda-forge.org/blog/posts/2020-10-29-macos-arm64/ and then install it in your normal terminal. Remember not to run this in a Rosetta Terminal, if you created one, because this installer script can run in your regular ARM terminal. 

That said even though it has Conda, that doesn't mean all packages are ARM ready -- only some are. The quick way to see what packages may be available for ARM are if you:

1. See a package made by conda-forge (https://anaconda.org/conda-forge) AND
2. That package has an architecture of `osx-arm64` or `noarch` (note that while these will install on ARM, these are not guaranteed to work -- of note, JupyterLab did not work).

If your package meets both requirements, you can install it by running `conda -c conda-forge <PACKAGE>` once you have Miniforge installed.

### Things you can definately get on an ARM Python
Note this doesn't mean these are ARM 'optimized' to use all of the fancy GPUs and Neural Cores. It just means unlike Intel, these aren't emulated so they can run on the CPU natively. That said, take all these with a grain of salt, I haven't got the chance to put all of these packages to their paces so some functionality may still be broken.

- Bokeh (Added on 11/23/2020)
- JupyterLab & Notebooks: It currently works via native ARM, but you need to change one line in the `ipykernel` source code to get it running at the time this was written (11/2020). Those instructions are at: https://github.com/ipython/ipykernel/pull/561
- Matplotlib
- Numpy
- Pandas
- PyTorch by manually installing the latest version from source (i.e. you cannot use `pip` or `conda` at this time, Nov 2020, to install it). More on this below.
- Scikit-Learn (Added on 11/21/2020)
- Scipy
- Statsmodel (Added on 11/21/2020)
- TensorFlow by manually installing Apple's version (more on this below)

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
[Apple has created a TensorFlow](https://github.com/apple/tensorflow_macos) that is optimized for their ARM Processors and GPUs. Their default installation works well if you're willing to let it create its own Python Virtual Environment. But if you want to use TensorFlow in your Miniconda enviornment, [download the tar.gz release from the Git Repo](https://github.com/apple/tensorflow_macos/releases) and then adapt the bash script below to install it in your conda environment.

```bash
# Put a path to where the arm64 libraries are. For example...
libs="/Users/Matthew/Downloads/tensorflow_macos/arm64/"

# Replace this with the path of your Conda environment
env="/Users/Matthew/miniforge3/envs/python38"

# The rest should work
conda upgrade -c conda-forge pip setuptools cached-property six

pip install --upgrade -t "$env/lib/python3.8/site-packages/" --no-dependencies --force "$libs/grpcio-1.33.2-cp38-cp38-macosx_11_0_arm64.whl"

pip install --upgrade -t "$env/lib/python3.8/site-packages/" --no-dependencies --force "$libs/h5py-2.10.0-cp38-cp38-macosx_11_0_arm64.whl"

pip install --upgrade -t "$env/lib/python3.8/site-packages/" --no-dependencies --force "$libs/tensorflow_addons-0.11.2+mlcompute-cp38-cp38-macosx_11_0_arm64.whl"

conda install -c conda-forge -y absl-py
conda install -c conda-forge -y astunparse
conda install -c conda-forge -y gast
conda install -c conda-forge -y opt_einsum
conda install -c conda-forge -y termcolor
conda install -c conda-forge -y typing_extensions
conda install -c conda-forge -y wheel
conda install -c conda-forge -y typeguard

pip install tensorboard

pip install wrapt flatbuffers tensorflow_estimator google_pasta keras_preprocessing protobuf

pip install --upgrade -t "$env/lib/python3.8/site-packages/" --no-dependencies --force "$libs/tensorflow_macos-0.1a0-cp38-cp38-macosx_11_0_arm64.whl"
```

### PyTorch
At the time this section was written, Nov 27 2020, PyTorch had a merge request that has been merged to add ARM Compilation (https://github.com/pytorch/pytorch/pull/48275) but this has not been oficially released. This means, to use PyTorch, you must compile PyTorch on your own. Furthermore even if you go down this route, several PyTorch 'side' packages do not work, such as TorchVision.

If you still want to install PyTorch, you'll want to first install Python using Miniforge linked above. After doing that, you should be able to follow the 'Build from Source' instructions at https://github.com/pytorch/pytorch#from-source. I don't think I needed Homebrew nor anything from it, but if you do, I am using a 'Rosetta compiled' version of Homebrew.

---

## Software that'll take some time
* [Docker](https://www.docker.com/blog/apple-silicon-m1-chips-and-docker/)
* [R or Fortran](https://developer.r-project.org/Blog/public/2020/11/02/will-r-work-on-apple-silicon/)

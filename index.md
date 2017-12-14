# PyTorch Without Anaconda

Recently, I had to install PyTorch on an Amazon P3 instance. The `pip` package manager wasn't able to get the job done, and I didn't want to get the recommended Anaconda, so I went at it on my own, wandering down various dead-ends and message boards in search of the light. Turns out it wasn't that complicated.

**Technical note:** This is for installing PyTorch v0.3.0 on Python 2.7.13, running on Ubuntu 16.04 on Amazon's new P3 instances, which come equipped with (at least one) monster Telsa V100 (16GB) Volta class GPU. Prior to attempting this, the box had been meticuluously purged of all old NVIDIA drivers and software, following which CUDA 9.0 and CuDNN 7 were installed, but I will not cover that here.

Finally, if you want to get PyTorch the correct way, go to [their official github](https://github.com/pytorch/pytorch). Beware all ye who enter, for here be dragons. I cannot guarantee it works for you, and if you break anything in the process, that's on you.

### Dependencies

If you have not already done so, install CUDA 9.0, CuDNN 7, and a BLAS library like MKL, ATLAS, or OpenBLAS. You're on your own for these.

You will also have to get NCCL from NVIDIA, which, if you have already got CuDNN, you know is an annoying process of downloading it from NVIDIA's website after signing in and doing a survey, and then you have to upload it to your AWS instance. Once you have done that, it's straightforward to install it with `dpkg`:
```
dpkg nccl-repo-ubuntu1604-2.1.2-ga-cuda9.0_1-1_amd64.deb
```

Next thing we need to install is [MAGMA](http://icl.cs.utk.edu/magma/software/index.html). I discovered this the hard way, after skipping it on my first pass, successfully building PyTorch from sources, and have it fail the tests, crying for MAGMA. Get the MAGMA sources with:

```
wget http://icl.utk.edu/projectsfiles/magma/downloads/magma-2.3.0.tar.gz
tar -xvzf magma-2.3.0.tar.gz
cd magma-2.3.0.tar.gz
```

We need to create a make.inc file here by modifying one of the template files given in `make.inc-examples`. Since I have OpenBLAS, I used `make.inc-examples/make.inc.openblas`

```
cp make.inc-examples/make.inc.openblas make.inc
```

Edit `make.inc` and make the following changes: 1) Uncomment GPU_TARGET and point it to the right GPU, and 2) Add paths to CUDA and your BLAS libraries. For me:

```
GPU_TARGET ?= Volta
CUDADIR = /usr/local/cuda-9.0
OPENBLASDIR = /usr/lib
```
Save and exit your editor, type `make` and hit Enter. And wait. Get coffee, or lunch, or get some other work done, because building MAGMA takes time! You can test your build of MAGMA with:

```
cd testing
python run_tests.sh
```
...but be warned that this too takes a LOT of time.

Next, we get the python dependencies listed on the PyTorch github page linked above. Just install them with pip as you would, maybe upgrade them in the process:

`sudo -H pip install numpy pyyaml mkl setuptools cmake cffi --upgrade`

I use sudo often because things seem to work better that way for me, but YMMV. 

### Building PyTorch from sources

With these preliminaries out of the way, it's fairly painless to build PyTorch from sources. All we have to take a little care of is to checkout a released version rather than the current master.

```
git clone --recursive https://github.com/pytorch/pytorch
cd pytorch
git checkout v0.3.0
sudo python setup.py install
```

Once this completes sucessfully, you can test your PyTorch build with:
```
cd test
./run_test.sh
```

I'm not a PyTorch developer, so I don't know about the fidelity of this build, but it passes the tests, and the [examples](https://github.com/pytorch/examples) seem to run fine, so I suppose it works. Happy to look into any challenges in getting this setup set up.

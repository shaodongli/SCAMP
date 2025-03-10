[![Build and Test](https://github.com/zpzim/SCAMP/actions/workflows/build-and-test.yml/badge.svg)](https://github.com/zpzim/SCAMP/actions/workflows/build-and-test.yml)
[![RTD Build Status](https://img.shields.io/readthedocs/scamp-docs)](https://scamp-docs.readthedocs.io/en/latest/)

[![Docker Build and Push](https://github.com/zpzim/SCAMP/actions/workflows/docker-build.yml/badge.svg)](https://github.com/zpzim/SCAMP/actions/workflows/docker-build.yml)
![Docker Image Version (latest semver)](https://img.shields.io/docker/v/zpzim/scamp?label=Docker%20Version)
![Docker Image Size (latest semver)](https://img.shields.io/docker/image-size/zpzim/scamp)
![Docker Pulls](https://img.shields.io/docker/pulls/zpzim/scamp)

![PyPI](https://img.shields.io/pypi/v/pyscamp?label=pyscamp%20version)
![PyPI - Downloads](https://img.shields.io/pypi/dm/pyscamp?label=pypi%20downloads)

![Conda (channel only)](https://img.shields.io/conda/vn/conda-forge/pyscamp-gpu?label=conda-forge::pyscamp-gpu)
![Conda](https://img.shields.io/conda/pn/conda-forge/pyscamp-gpu?label=pyscamp-gpu)
![Conda](https://img.shields.io/conda/dn/conda-forge/pyscamp-gpu?label=Downloads%3A%20pyscamp-gpu)

![Conda (channel only)](https://img.shields.io/conda/vn/conda-forge/pyscamp-cpu?label=conda-forge::pyscamp-cpu)
![Conda](https://img.shields.io/conda/pn/conda-forge/pyscamp-cpu?label=pyscamp-cpu)
![Conda](https://img.shields.io/conda/dn/conda-forge/pyscamp-cpu?label=Downloads%3A%20pyscamp-cpu)

[![DOI](https://zenodo.org/badge/127799249.svg)](https://zenodo.org/badge/latestdoi/127799249)


# SCAMP: SCAlable Matrix Profile

## Table of Contents
[Overview](https://github.com/zpzim/SCAMP#overview) \
[Documentation](https://github.com/zpzim/SCAMP#documentation) \
[Performance](https://github.com/zpzim/SCAMP#performance) \
[Python Module](https://github.com/zpzim/SCAMP#python-module) \
[Run Using Docker](https://github.com/zpzim/SCAMP#run-using-docker) \
[Distributed Operation](https://github.com/zpzim/SCAMP#distributed-operation) \
[Reference](https://github.com/zpzim/SCAMP#reference)

## Overview
This is a GPU/CPU implementation of the SCAMP algorithm. SCAMP takes a time series as input and computes the matrix profile for a particular window size. You can read more at the [Matrix Profile Homepage](http://www.cs.ucr.edu/~eamonn/MatrixProfile.html)
This is a much improved framework over [GPU-STOMP](https://github.com/zpzim/STOMPSelfJoin) which has the following additional features:
  * Tiling for large inputs 
  * Computation in fp32, mixed fp32/fp64, or fp64 (double is recommended for most datasets, single precision will work for some)
  * fp32 version should get good performance on GeForce cards
  * AB joins (you can produce the matrix profile from 2 different time series)
  * Distributable (we use GCP but other cloud platforms can work) with verified scalability to billions of datapoints
  * More types of matrix profiles! KNN, Matrix Summary, Sum, and 1NN without index! See the Docs!
  * Extremely Efficient Implementation
  * Extensible to adding optimized versions of custom join operations.
  * CPU Support (Only enabled for double precision; does not support KNN joins yet)
  * Handles NaN input values. The matrix profile will be computed while excluding any subsequence with a NaN value
  * Python module: Use SCAMP in Python with pyscamp
  * conda-forge integration: Install pyscamp seamlessly with conda.
  * Extensive integration testing: SCAMP has thousands of input configurations tested with every PR.
  * Automatic benchmarking: Helps ensure performance doesn't slip with future updates.

## Why use SCAMP?

  * It is [faster](https://scamp-docs.readthedocs.io/en/latest/performance.html) than other matrix profile libraries. For example, it is **20x** to **100x** faster than stumpy.
  * It is very easy to install using conda and has very few dependencies.
  * It handles real data: very large inputs, missing values, and flat regions with little issue.
  * It can compute various other types of matrix profiles, including efficiently computing KNN matrix profiles, and matrix summaries (a.k.a. mplots). And can be extended to compute other types of profile efficiently.

## Documentation
SCAMP's documentation can be found at [readthedocs](https://scamp-docs.readthedocs.io/en/latest/).

## Python module
`pyscamp` is available through conda-forge:
~~~
# To install pyscamp with cpu/gpu support on Linux and Windows.
conda install -c conda-forge pyscamp-gpu

# To install pyscamp with cpu support only on Windows, Linux, or MacOS.
conda install -c conda-forge pyscamp-cpu
~~~

Note that `pyscamp-gpu` can be installed and used even if you don't have a GPU, it will simply fall back to using your CPU. However, `pyscamp-cpu` is preferrable if you don't have a GPU because it builds with a newer compiler and does not require installing the `cudatoolkit` depencency.

If you run into problems using GPUs with `pyscamp-gpu` make sure your NVIDIA drivers are up to date. This is the most common cause of issues.

### Installing from source

If you want you can build pyscamp from source which will have improved performance. A source distribution for a python3 module using pybind11 is available on pypi.org to install run:
~~~
# Python 3 and a c/c++ compiler is required.
# cmake is required (if you don't have it you can pip install cmake)
pip install pyscamp
~~~

Once installed you can use SCAMP in Python as follows:
~~~
import pyscamp as mp

# Allows checking if pyscamp was built with CUDA and has GPU support.
has_gpu_support = mp.gpu_supported()

# Self join
profile, index = mp.selfjoin(a, sublen)

# AB join using 4 threads, outputting pearson correlation.
profile, index = mp.abjoin(a, b, sublen, pearson=True, threads=4)
~~~

### Installing from source 2
~~~
git clone https://github.com/zpzim/SCAMP
cd SCAMP
git submodule update --init --recursive
mkdir build && cd build

cmake ..
cmake --build . --config Release
pip install .
~~~


More information and the API documentation for pyscamp is available on [readthedocs](https://scamp-docs.readthedocs.io/en/latest/)

## Run Using Docker
You can run SCAMP via [nvidia-docker](https://github.com/NVIDIA/nvidia-docker) using the prebuilt [image](https://hub.docker.com/r/zpzim/scamp) on dockerhub.

In order to expose the host GPUs nvidia-docker must be installed correctly. Please follow the directions provided on the nvidia-docker github page. The following example uses docker 19.03 functionality:
~~~
docker pull zpzim/scamp:latest
docker run --gpus all \
   --volume /path/to/host/input/data/directory:/data \
   --volume /path/to/host/output/directory:/output \
   zpzim/scamp:latest /SCAMP/build/SCAMP \
   --window=<window_size> --input_a_file_name=/data/<filename> \
   --output_a_file_name=/output/<mp_filename> \
   --output_a_index_file_name=/output/<mp_index_filename>
~~~

## Distributed Operation
We have a client/server architecture built using grpc. Tested on [GKE](https://cloud.google.com/kubernetes-engine/) but should be possible to get working on [Amazon EKS](https://aws.amazon.com/eks/) as well. 

For more information on how to use the scamp client and server, please take a look at the [documentation](https://scamp-docs.readthedocs.io/en/latest/)

## Reference
If you use SCAMP in your work, please reference the following paper:
~~~
Zimmerman, Zachary, et al. "Matrix Profile XIV: Scaling Time Series Motif Discovery with GPUs to Break a Quintillion Pairwise Comparisons a Day and Beyond." Proceedings of the ACM Symposium on Cloud Computing. 2019.
~~~

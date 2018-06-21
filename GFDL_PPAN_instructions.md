Install notes for GFDL PPAN Cluster 
***********************************

This installation uses conda to install BLAS, openmpi, and the various Python packages required by dedalus. The only exceptions are HDF5, FFTW, and h5py which are built manually from source. For these manual installations, the source download must be conducted on the `public` nodes, which can access the internet. The installation, however, must be done on the `analysis` nodes, which have different compiliers. By default, these instructions create the directory ``\nbhome\${USER}\software`` and install dedalus, HDF5, FFTW, and h5py within this directory. We assume the user is running the default c-shell. 

Download Source Files to Public 
-------------------------------

These instructions assume you have installed Anaconda (or Miniconda) to your ``\nbhome\${USER}``. To start, let's create a new conda environemnt for your dedalus installation. 

Login into ``public`` and create a  ``dedalus.yml`` file with following contents:

```
name: dedalus 
channels:
  - defaults
  - conda-forge 
dependencies:
  - numpy=1.12 
  - cython 
  - scipy 
  - mpi4py 
  - openblas 
  - openmpi 
  - matplotlib
  - pathlib
  - docopt
```

Setup and activate the dedalus environment, 
```
conda env create -f dedalus.yml
source activate dedalus 
```

We will now create a file that sets the environment variables necessary for subsequently building the packages.

Create a file entitled ``dedalus_paths.csh`` with the following content:
```
#DEDALUS SETUP

module load gcc

source /nbhome/${USER}/anaconda3/etc/profile.d/conda.csh
conda activate dedalus

setenv BLAS /nbhome/${USER}/anaconda3/envs/dedalus/lib/libopenblas.so
setenv MPI_PATH /nbhome/${USER}/anaconda3/envs/dedalus/lib/openmpi
setenv LD_LIBRARY_PATH ${MPI_PATH}:${BLAS}:${LD_LIBRARY_PATH}

setenv HDF5_DIR /nbhome/${USER}/software/hdf5
setenv HDF5_VERSION 1.10.1
setenv HDF5_MPI "ON"
setenv PATH ${HDF5_DIR}/bin:${PATH}
setenv LD_LIBRARY_PATH ${HDF5_DIR}/lib:${LD_LIBRARY_PATH}

setenv FFTW_PATH /nbhome/${USER}/software/fftw
setenv FFTW_VERSION 3.3.6-pl2
setenv PATH ${FFTW_PATH}/bin:${PATH}
setenv LD_LIBRARY_PATH ${FFTW_PATH}/lib:${LD_LIBRARY_PATH}

setenv DEDALUS_REPO /nbhome/${USER}/software/dedalus
setenv PYTHONPATH ${DEDALUS_REPO}:${PYTHONPATH}

setenv H5PY_REPO /nbhome/${USER}/software/h5py
```

Now run, ``source dedalus_paths.csh`` to set these environmental paths. 
 
We are now ready to create the directories and download the source files. To do this, run the following script:
 
```
    # download HDF5 from source
    mkdir -p ${HDF5_DIR}
    cd ${HDF5_DIR}
    wget https://support.hdfgroup.org/ftp/HDF5/current/src/hdf5-${HDF5_VERSION}.tar

    # download FFTW built from source
    mkdir -p ${FFTW_PATH}
    cd ${FFTW_PATH}
    wget http://www.fftw.org/fftw-${FFTW_VERSION}.tar.gz

    # Dedalus from mercurial
    hg clone https://bitbucket.org/dedalus-project/dedalus ${DEDALUS_REPO}
    cd ${DEDALUS_REPO}

    # download h5py from source
    cd /nbhome/${USER}/software/
    git clone https://github.com/h5py/h5py.git
```

Build and Install on Analysis
------------------------
Login into the analysis cluster and  ``source dedalus_paths.csh``. To build the packages, run the following script 

```
    # HDF5 built from source
    cd ${HDF5_DIR}
    tar -xvf hdf5-${HDF5_VERSION}.tar
    cd hdf5-${HDF5_VERSION}
    ./configure --prefix=${HDF5_DIR} \
        CC=mpicc \
        CXX=mpicxx \
        F77=mpif90 \
        MPICC=mpicc \
        MPICXX=mpicxx \
        --enable-shared \
        --enable-parallel
    make -j4
    make install
    
    #pip3 install --user --no-binary=h5py h5py

    # FFTW built from source
    cd ${FFTW_PATH}
    tar -xvzf fftw-${FFTW_VERSION}.tar.gz
    cd fftw-${FFTW_VERSION}
    ./configure --prefix=${FFTW_PATH} \
        CC=mpicc \
        CXX=mpicxx \
        F77=mpif90 \
        MPICC=mpicc \
        MPICXX=mpicxx \
        --enable-shared \
        --enable-mpi \
        --enable-openmp \
        --enable-threads
    make -j4
    make install

    # Dedalus from mercurial
    cd ${DEDALUS_REPO}
    #pip install --user -r requirements.txt
    python setup.py build_ext --inplace
```

Notes
-----


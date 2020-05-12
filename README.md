# Install-Caffe-Ubuntu-18.04
## Step 1: Install prerequisites
You need to install CUDA 10.0 and cuDNN first.
```
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get install -y build-essential cmake git pkg-config
$ sudo apt-get install -y libprotobuf-dev libleveldb-dev libsnappy-dev libhdf5-serial-dev protobuf-compiler
$ sudo apt-get install -y libatlas-base-dev
$ sudo apt-get install -y --no-install-recommends libboost-all-dev
$ sudo apt-get install -y libgflags-dev libgoogle-glog-dev liblmdb-dev
$ sudo apt-get install -y python3-dev
$ sudo apt-get install -y python3-numpy python3-scipy
$ sudo apt-get install -y libopencv-dev
$ sudo apt-get install -y python3-pip
$ sudo apt-get install libopenblas-dev
$ sudo apt-get install libgflags-dev libgoogle-glog-dev liblmdb-dev
$ cd /usr/lib/x86_64-linux-gnu
$ sudo ln -s libboost_python-py35.so libboost_python3.so
```
Optionally, you can also install OpenCV for Python.
```
$ sudo pip3 install opencv-python
```
## Step 2: Clone Caffe
```
$ cd ~
$ git clone https://github.com/BVLC/caffe.git
```
## Step 3: Modify blocking_queue.cpp
You need to modify blocking_queue.cpp.
```
$ gedit ~/caffe/src/caffe/util/blocking_queue.cpp
```
After line 89, add this line of code
```
template class BlockingQueue<Datum*>;
```
## Step 4: Create Makefile.config
```
$ cp ~/caffe/Makefile.config.example ~/caffe/Makefile.config
```
## Step 5: Modify Makefile.config
Open Makefile.config
```
$ gedit ~/caffe/Makefile.config
```
and make sure you have this in your configuration
```
USE_CUDNN := 1
OPENCV_VERSION := 4
PYTHON_INCLUDE := /usr/include/python3.5 /usr/lib/python3.5/dist-packages/numpy/core/include
PYTHON_LIBRARIES := boost_python3 python3.5m
WITH_PYTHON_LAYER := 1 
INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial
LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu /usr/lib/x86_64-linux-gnu/hdf5/serial
CUDA_DIR := /usr/local/cuda-10.0
CUDA_ARCH := -gencode arch=compute_30,code=sm_30 \
    -gencode arch=compute_35,code=sm_35 \
    -gencode arch=compute_50,code=sm_50 \
    -gencode arch=compute_52,code=sm_52 \
    -gencode arch=compute_60,code=sm_60 \
    -gencode arch=compute_61,code=sm_61 \
    -gencode arch=compute_61,code=compute_61
```
## Step 6: Symlink HDF5
```
$ cd /usr/lib/x86_64-linux-gnu
$ sudo ln -s libhdf5_serial.so.8.0.2 libhdf5.so
$ sudo ln -s libhdf5_serial_hl.so.8.0.2 libhdf5_hl.so
```
## Step 7: Modify Makefile
```
$ gedit ~/caffe/Makefile
```
Replace this line
```
NVCCFLAGS += -ccbin=$(CXX) -Xcompiler -fPIC $(COMMON_FLAGS)
```
with this one
```
NVCCFLAGS += -D_FORCE_INLINES -ccbin=$(CXX) -Xcompiler -fPIC $(COMMON_FLAGS)
```
Replace this line
```
LIBRARIES += glog gflags protobuf boost_system boost_filesystem m hdf5_hl hdf5
```
with this line
```
LIBRARIES += glog gflags protobuf leveldb snappy \
    lmdb boost_system boost_filesystem hdf5_hl hdf5 m \
    opencv_core opencv_highgui opencv_imgproc opencv_imgcodecs opencv_videoio
```
Replace this line
```
PYTHON_LIBRARIES ?= boost_python python2.7
```
with this line
```
PYTHON_LIBRARIES ?= boost_python3 python3.5m
```
## Step 8: Change CMakeLists.txt
```
$ gedit ~/caffe/CMakeLists.txt
```
Add this line:
```
# ---[ Includes
set(${CMAKE_CXX_FLAGS} "-D_FORCE_INLINES ${CMAKE_CXX_FLAGS}")
```
## Step 9: Install Python requirements
```
$ cd ~/caffe/python
$ for req in $(cat requirements.txt); do sudo -H pip3 install $req --upgrade; done
```
## Step 10: Make your Caffe
```
$ cd ~/caffe
$ make all -j $(($(nproc) + 1))
$ make test -j $(($(nproc) + 1))
$ make runtest -j $(($(nproc) + 1))
$ make pycaffe -j $(($(nproc) + 1))
$ make distribute -j $(($(nproc) + 1))
```
## Step 11: Add PYTHONPATH
You need to open your .bashrc
```
$ gedit ~/.bashrc
```
and add this
```
export PYTHONPATH=~/caffe/python:$PYTHONPATH 
```
and don't forget to
```
$ source ~/.bashrc
```
## Common Problems
For different OpenCV version, you might have different library. If something's missing from the library, you can remove it from LIBRARIES inside the Makefile.
## Libboost
Checkout https://gist.github.com/melvincabatuan/a5a4a10b15ef31a5a481
If necessary, you can also build your own Boost.
```
$ cd ~
$ wget --no-verbose https://dl.bintray.com/boostorg/release/1.65.1/source/boost_1_65_1.tar.gz
$ tar xzf boost_1_65_1.tar.gz
```
You need to modify user-config.jam
```
$ gedit ~/boost_1_65_1/tools/build/example/user-config.jam
```
Make sure you have this configuration (at the very bottom)
```
# using python : 3.5 : /usr/local/bin/python3 : /usr/local/include/python3.5m : /usr/local/lib ;
```
Then, proceed
```
$ sudo ln -s /usr/local/include/python3.5m /usr/local/include/python3.5
$ cd ~
$ ./bootstrap.sh --with-python=/usr/local/bin/python3 --with-python-version=3.5 --with-python-root=/usr/local/lib/python3.5
$ sudo ./b2 --enable-unicode=ucs4 -j $(($(nproc) + 1)) toolset=gcc cxxflags="-std=c++11" install
$ sudo ldconfig
```

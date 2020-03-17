# Install-Caffe-Ubuntu-18.04
## Step 1: Install prequisites
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
## Step 3: Modify
Modify blocking_queue.cpp.
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
and make sure you have this in your configuration.
```
USE_CUDNN := 1
OPENCV_VERSION := 4
PYTHON_INCLUDE := /usr/include/python3.6 /usr/lib/python3.6/dist-packages/numpy/core/include
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
## Step 6: Solve HDF5
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
$ NVCCFLAGS += -ccbin=$(CXX) -Xcompiler -fPIC $(COMMON_FLAGS)
```
with this one
```
NVCCFLAGS += -D_FORCE_INLINES -ccbin=$(CXX) -Xcompiler -fPIC $(COMMON_FLAGS)
```
Replace this line
```
LIBRARIES += glog gflags protobuf boost_system boost_filesystem m
```
with this line
```
LIBRARIES += glog gflags protobuf leveldb snappy \
    lmdb boost_system boost_filesystem hdf5_hl hdf5 m \
    opencv_core opencv_highgui opencv_imgproc opencv_imgcodecs opencv_videoio
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
$ for req in $(cat requirements.txt); do sudo -H pip install $req --upgrade; done
```
## Step 10: Make your Caffe
```
$ cd ~/caffe
$ make all -j $(($(nproc) + 1))
$ make test
$ make runtest
$ make pycaffe
$ make distribute
```
## Step 11: Add PYTHONPATH
You need to open your .bashrc and add this PYTHONPATH.
```
$ export PYTHONPATH=~/caffe/python:$PYTHONPATH 
```

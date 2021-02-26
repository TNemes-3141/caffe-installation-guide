# Caffe installation guide

Caffe is a Deep Learning framework for Linux which can be used either through the console or Python. Note that this guide features the installation process with GPU support.

### Dependencies:
- Ubuntu 16.04 - 20.04 LTS
- Python 3 (here, I'm using Python 3.8, please adapt commands for your Python version)
- CUDA >= 10 and corresponding cuDNN version
- OpenCV 4

---

0. If required: Upgrade GNU-compiler to version 8 (at least 6.5 is needed):
```
$ sudo apt-get remove gcc-4.9 g++-4.9
$ sudo apt autoremove
$ sudo apt-get install gcc-8 g++-8
$ sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 20
$ sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-8 20
$ sudo update-alternatives --install /usr/bin/cc cc /usr/bin/gcc 40
$ sudo update-alternatives --set cc /usr/bin/gcc
$ sudo update-alternatives --install /usr/bin/c++ c++ /usr/bin/g++ 40
$ sudo update-alternatives --set c++ /usr/bin/g++
```

1. Install cuDNN (I'm assuming CUDA is already installed, please adapt commands for your CUDA/cuDNN version):

https://developer.nvidia.com/cudnn
```
# Download package manually:
    cuDNN Library for Linux (under Download cuDNN v7.6.5 (November 5th, 2019), for CUDA 10.1)
$ tar -xzvf cudnn-10.1-linux-x64-v7.6.5.32.tgz
$ sudo cp cuda/include/cudnn*.h /usr/local/cuda/include
$ sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
$ sudo chmod a+r /usr/local/cuda/include/cudnn*.h /usr/local/cuda/lib64/libcudnn*
$ cd /usr/local/cuda/lib64
$ sudo rm libcudnn.so
$ sudo rm libcudnn.so.7
$ sudo ln -s libcudnn.so.7.6.5 libcudnn.so.7
$ sudo ln -s libcudnn.so.7 libcudnn.so
$ sudo ldconfig
```

2. Install dependencies for OpenCV:
```
$ sudo apt-get install build-essential cmake git unzip pkg-config libjpeg-dev libpng-dev libtiff-dev libavcodec-dev libavformat-dev libswscale-dev libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev libfaac-dev libmp3lame-dev libtheora-dev libavresample-dev libvorbis-dev libopencore-amrnb-dev libopencore-amrwb-dev libgtk2.0-dev libcanberra-gtk-module libcanberra-gtk3-module x264 libxvidcore-dev libx264-dev libgtk-3-dev python3-dev python3-numpy python3-pip python3-testresources libtbb2 libtbb-dev libdc1394-22-dev libv4l-dev v4l-utils libxine2-dev software-properties-common
$ cd /usr/include/linux
$ sudo ln -s videodev2.h videodev.h
$ cd ~
$ sudo add-apt-repository "deb http://security.ubuntu.com/ubuntu xenial-security main"
$ sudo apt-get update
$ sudo apt-get install libjasper-dev libopenblas-dev libatlas-base-dev libblas-dev liblapack-dev gfortran libhdf5-dev protobuf-compiler libprotobuf-dev libgoogle-glog-dev libgflags-dev
# Note: If the last command for some reason fails (e.g. outdated repository), just install all packages except libjasper-dev. It's not included in current Ubuntu repos anymore and recommended for OpenCV, but not required.
```

3. Install OpenCV 4:
```
$ wget -O opencv.zip https://github.com/opencv/opencv/archive/4.4.0.zip
$ wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.4.0.zip
$ unzip opencv.zip
$ unzip opencv_contrib.zip
$ mv opencv-4.4.0 opencv
$ mv opencv_contrib-4.4.0 opencv_contrib
$ cd opencv
$ mkdir build
$ cd build
# Execute Cmake with following flags:

$ cmake -D CMAKE_BUILD_TYPE=RELEASE \
        -D CMAKE_INSTALL_PREFIX=/usr/local \
        -D OPENCV_EXTRA_MODULES_PATH=/home/[YOUR_USERNAME]/opencv_contrib/modules \
        -D BUILD_TIFF=ON \
        -D WITH_FFMPEG=ON \
        -D WITH_GSTREAMER=ON \
        -D WITH_TBB=ON \
        -D BUILD_TBB=ON \
        -D WITH_EIGEN=ON \
        -D WITH_V4L=ON \
        -D WITH_LIBV4L=ON \
        -D WITH_VTK=OFF \
        -D WITH_OPENGL=ON \
        -D OPENCV_ENABLE_NONFREE=ON \
        -D INSTALL_C_EXAMPLES=OFF \
        -D INSTALL_PYTHON_EXAMPLES=OFF \
        -D BUILD_NEW_PYTHON_SUPPORT=ON \
        -D OPENCV_GENERATE_PKGCONFIG=ON \
        -D BUILD_TESTS=OFF \
        -D BUILD_EXAMPLES=OFF \
        -D WITH_CUDA=ON \
        -D CUDA_ARCH_BIN=7.0 \ # please substitute this number for the compute capability of your own card, found here: https://developer.nvidia.com/cuda-gpus
        -D OPENCV_DNN_CUDA=ON \
        -D WITH_CUDNN=ON \
        -D ENABLE_FAST_MATH=ON \
        -D CUDA_FAST_MATH=ON \
        -D CUDNN_LIBRARY=/usr/local/cuda/lib64/libcudnn.so.7.6.5 \ # look up your own directory
        -D CUDNN_INCLUDE_DIR=/usr/local/cuda/include \
        -D WITH_CUBLAS=ON ..
        
$ make -j$(nproc)
$ sudo make install
$ sudo ldconfig
$ gedit .bashrc
# Append the following:
    export PKG_CONFIG_PATH="/home/[YOUR_USERNAME]/opencv/build/unix-install" # or your folder with 'opencv4.pc'
$ source .bashrc

# Test your installation with:
$ python3
>>> import cv2
>>> print(cv2.__version__)
>>> 4.4.0                  # expected result
```
4. Install Caffe dependencies:
```
$ sudo apt-get install libleveldb-dev liblmdb-dev libsnappy-dev libhdf5-serial-dev python3-skimage graphviz
$ sudo apt-get install --no-install-recommends libboost-all-dev
$ pip3 install pydot protobuf
```
5. Compile and install Caffe:
```
$ cd ~/opencv/build/unix-install # or your OpenCV installation folder
$ sudo ln -s opencv4.pc opencv.pc
$ cd ~
$ wget -O caffe.zip https://github.com/Qengineering/caffe/archive/ssd.zip
# Note: The above is a modified Caffe with OpenCV 4 and cuDNN 8 compatibility. It has all the functions of normal Caffe, it just adds some extra networks like SSD. So whenever you can, use this instead since the original is deprecated and not compatible with current software.
$ unzip caffe.zip
$ mv caffe-ssd caffe
$ cd caffe
# Note: Included in the Caffe repo, there are many example Makefile configurations included. Choose the one which best fits your needs and system.
$ cp [CHOSEN_CONFIG_FILE].example Makefile.config
$ gedit Makefile.config
# Note: Here, you can adjust some settings to your liking, but I wouldn't recommend touching it too much as the settings are optimized to work on your chosen system.
$ gedit src/caffe/util/math_functions.cpp
# Note: Edit the following function to this:
# void caffe_rng_uniform(const int n, Dtype a, Dtype b, Dtype* r) {
#   CHECK_GE(n, 0);
#   CHECK(r);

#   if(a > b) {
#     Dtype c = a;
#     a = b;
#     b = c;
#   }
#   CHECK_LE(a, b);
# }
$ make clean
$ make all -j$(nproc)
$ make test -j$(nproc) # important: run all test to see if there's any isuue somewhere!
$ make runtest -j$(nproc)
$ make pycaffe -j$(nproc)
$ make pytest -j$(nproc)
$ cd ~
$ gedit .bashrc
# Append the following:
    export PYTHONPATH="${PYTHONPATH}:/home/[YOUR_USERNAME]/caffe/python" # or the location of the Python folder of your Caffe installation
$ source .bashrc

# Test installation with:
$ python3
>>> import caffe
>>> print(caffe.__version__)
>>> 1.0.0 or 1.0.0-rc3       # expected result
```
Hope this guide helps, I included my own experiences on the installation process which most guides miss. Check out my sources in the "References" tab.
Good luck!

---
    
## References:
- Official guides: https://qengineering.eu/install-caffe-on-ubuntu-20.04-with-opencv-4.4.html, https://github.com/weiliu89/caffe/tree/ssd, https://github.com/chuanqi305/MobileNet-SSD, https://docs.nvidia.com/deeplearning/sdk/cudnn-install/index.html
- GitHub-issues: https://github.com/BVLC/caffe/issues/1325, https://github.com/BVLC/caffe/issues/4843, https://github.com/opencv/opencv/issues/14205, https://github.com/weiliu89/caffe/issues/669#issuecomment-339542120
- Stack Overflow and Ask Ubuntu: https://stackoverflow.com/questions/38680593/importerror-no-module-named-google-protobuf, https://stackoverflow.com/questions/51350998/7515-fatal-error-stdlib-h-no-such-file-or-directory-include-next-stdlib-h, https://askubuntu.com/questions/1025928/why-do-i-get-sbin-ldconfig-real-usr-local-cuda-lib64-libcudnn-so-7-is-not-a
- "Caffe Users" Google-group: https://groups.google.com/g/caffe-users/

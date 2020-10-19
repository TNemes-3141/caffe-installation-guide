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

1. Install cuDNN (CUDA already installed, please adapt commands for your CUDA/cuDNN version):

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

2. Install dependencies for OpenCV
```
$ sudo apt-get install build-essential cmake git unzip pkg-config libjpeg-dev libpng-dev libtiff-dev libavcodec-dev libavformat-dev libswscale-dev libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev libfaac-dev libmp3lame-dev libtheora-dev libavresample-dev libvorbis-dev libopencore-amrnb-dev libopencore-amrwb-dev libgtk2.0-dev libcanberra-gtk-module libcanberra-gtk3-module x264 libxvidcore-dev libx264-dev libgtk-3-dev python3-dev python3-numpy python3-pip python3-testresources libtbb2 libtbb-dev libdc1394-22-dev libv4l-dev v4l-utils libxine2-dev software-properties-common
$ cd /usr/include/linux
$ sudo ln -s -f ../libv4l1-videodev.h videodev.h
$ cd ~
$ sudo add-apt-repository "deb http://security.ubuntu.com/ubuntu xenial-security main"
$ sudo apt-get update
$ sudo apt-get install libjasper-dev libopenblas-dev libatlas-base-dev libblas-dev liblapack-dev gfortran libhdf5-dev protobuf-compiler libprotobuf-dev libgoogle-glog-dev libgflags-dev
```

3. Install OpenCV 4
```
$ wget -O opencv.zip https://github.com/opencv/opencv/archive/4.1.2.zip
$ wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.1.2.zip
$ unzip opencv.zip
$ unzip opencv_contrib.zip
$ mv opencv-4.1.2 opencv
$ mv opencv_contrib-4.1.2 opencv_contrib
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
        -D WITH_CUDNN=ON \
        -D ENABLE_FAST_MATH=ON \
        -D CUDA_FAST_MATH=ON \
        -D WITH_CUBLAS=ON ..
        
$ make -j$(nproc)
$ sudo make install
$ sudo ldconfig
$ gedit .bashrc
# Append following lines:
    export PKG_CONFIG_PATH="/home/[YOUR_USERNAME]/opencv/build/unix-install" # or your folder with 'opencv4.pc'
$ source .bashrc

# Test your installation with:
$ python3
>>> import cv2
>>> print(cv2.__version__)
>>> 4.1.2                    # expected result
```
4. Install Caffe dependencies:
```
$ sudo apt-get install libleveldb-dev liblmdb-dev libsnappy-dev libhdf5-serial-dev python3-skimage graphviz
$ sudo apt-get install --no-install-recommends libboost-all-dev
$ pip3 install pydot protobuf
```
5. Option 1: Standard-Caffe installieren:
    `$ cd /usr/lib/x86_64-linux-gnu`
    `$ sudo ln -s libboost_python38.so libboost_python3.so`
    `$ cd ~`
    `$ wget -O caffe.zip https://github.com/Qengineering/caffe/archive/master.zip`
    `$ unzip caffe.zip`
    `$ cd caffe-master`
    `$ kate Makefile`
    `# Folgende Änderungen vornehmen:`
        `# In Zeile 218: python3.6 ändern zu python3.8`
    `$ cp Makefile.config.example Makefile.config`
    `$ kate Makefile.config`
    `# Folgende Änderungen vornehmen:`
        `# In Zeile 5: USE_CUDNN := 1 entkommentieren`
        `# In Zeile 40: -gencode arch=compute_20,code=sm_20 und -gencode arch=compute_20,code=sm_21 löschen`
        `# In Zeile 80: Ändern zu PYTHON_LIBRARIES := boost_python3 python3.8`
        `# In Zeile 81: Ändern zu PYTHON_INCLUDE := /usr/include/python3.8 \`
        `# In Zeile 96: Ändern zu INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial/ /usr/include`
    `$ make clean`
    `$ make all -j$(nproc)`
    `$ make test -j$(nproc)`
    `$ make runtest -j$(nproc)`
    `$ make pycaffe -j$(nproc)`
    `$ make pytest -j$(nproc)`
    `$ cd ~`
    `$ kate .bashrc`
    `# Folgende Zeilen anfügen:`
        `export PYTHONPATH="${PYTHONPATH}:/home/tamas/caffe-master/python"`
    `$ source .bashrc`
    `# Installation testen mit:`
    `$ python3`
    `>>> import caffe`
    `>>> import caffe.proto.caffe_pb2 as pb`
    `>>> print(caffe.__version__)`
    `# Output: 1.0.0`

6. Option 2: Caffe-SSD installieren:
    `$ cd /usr/lib/x86_64-linux-gnu`
    `$ sudo ln -s libboost_python38.so libboost_python3.so`
    `$ cd ~/opencv/build/unix-install`
    `$ sudo ln -s opencv4.pc opencv.pc`
    `$ cd ~`
    `$ git clone https://github.com/weiliu89/caffe.git`
    `$ cd caffe`
    `$ git checkout ssd`
    `$ kate Makefile`
    `# Folgende Änderungen vornehmen:`
        `# In Zeile 202: python2.7 ändern zu python3.8`
    `$ cp Makefile.config.example Makefile.config`
    `$ kate Makefile.config`
    `# Folgende Änderungen vornehmen:`
        `# In Zeile 5: USE_CUDNN := 1 entkommentieren`
        `# In Zeile 21: OPENCV_VERSION := 3 entkommentieren`
        `# In Zeile 25: USE_PKG_CONFIG := 1 vorziehen und entkommentieren`
        `# In Zeile 39: -gencode arch=compute_20,code=sm_20 und -gencode arch=compute_20,code=sm_21 löschen, -gencode arch=compute_60,code=sm_60, -gencode arch=compute_61,code=sm_61 und -gencode arch=compute_61,code=compute_61 einfügen`
        `# In Zeile 70 und 71: PYTHON_INCLUDE auskommentieren`
        `# In Zeile 80: PYTHON_LIBRARIES entkommentieren und zu 'boost_python3 python3.8' ändern`
        `# In Zeile 81: PYTHON_INCLUDE entkommentieren und zu /usr/include/python3.8 \ /usr/lib/python3/dist-packages/numpy/core/include ändern`
        `# In Zeile 93: WITH_PYTHON_LAYER := 1 entkommentieren`
        `# IN Zeile 96: INCLDUE_DIRS ändern zu: /usr/local/include /usr/include/hdf5/serial/ /usr/local/include/opencv4`
        `# In Zeile 97: LIBRARY_DIRS ändern zu: /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu/hdf5/serial/`
    `$ kate src/caffe/layers/window_data_layer.cpp`
    `# Folgende Änderungen vornehmen:`
        `$ In Zeile 293: Ändern zu cv_img = cv::imread(image.first, cv::IMREAD_COLOR);`
    `$ kate src/caffe/layers/video_data_layer.cpp`
    `# Folgende Änderungen vornehmen:`
        `# In Zeile 55: Ändern zu total_frames_ = cap_.get(cv::CAP_PROP_FRAME_COUNT);`
        `# In Zeile 60: Ändern zu cap_.set(cv::CAP_PROP_POS_FRAMES, 0);`
    `$ kate src/caffe/util/io.cpp`
    `# Folgende Änderungen vornehmen:`
        `# In Zeile 86 und 87: Ändern zu int cv_read_flag = (is_color ? cv::IMREAD_COLOR : cv::IMREAD_GRAYSCALE);`
        `# In Zeile 666 und 667: Ändern zu int cv_read_flag = (is_color ? cv::IMREAD_COLOR : cv::IMREAD_GRAYSCALE);`
    `$ kate src/caffe/util/bbox_util.cpp`
    `# Folgende Änderungen vornehmen:`
        `# In Zeile 2186: Ändern zu CV_RGB(255, 255, 255), cv::FILLED);`
        `# In Zeile 2212: Ändern zu color, cv::FILLED);`
        `# In Zeile 2221: Ändern zu cv::VideoWriter outputVideo(save_file, cv::VideoWriter::fourcc('D', 'I', 'V', 'X'),`
    `$ kate src/caffe/util/im_transforms.cpp`
    `# Folgende Änderungen vornehmen:`
        `# In Zeile 246, 248, 251, 429, 430, 442, 452, 465, 475, 488, 491, 539, 546, 617, 628, 651 und 662: Fehler ausbessern`
    `$ kate src/caffe/test/test_io.cpp`
    `# Folgende Änderungen vornehmen:`
        `# In Zeile 23 und 24: Ändern zu int cv_read_flag = (is_color ? cv::IMREAD_COLOR : cv::IMREAD_GRAYSCALE);`
    `$ make -j$(nproc)`
    `$ make py -j$(nproc)`
    `$ make test -j$(nproc)`
    `$ make runtest -j$(nproc)`
    `$ cd ~`
    `$ kate .bashrc`
    `# Folgende Zeilen anfügen:`
        `export PYTHONPATH="${PYTHONPATH}:/home/tamas/caffe/python"`
    `$ source .bashrc`
    `# Installation testen mit:`
    `$ python3`
    `>>> import caffe`
    `>>> import caffe.proto.caffe_pb2 as pb`
    `>>> print(caffe.__version__)`
    `# Output: 1.0.0-rc3`

---
    
## Referenzen:
- Offizielle Guides: https://qengineering.eu/install-caffe-on-ubuntu-18.04-with-opencv-4.1.html, https://github.com/weiliu89/caffe/tree/ssd, https://github.com/chuanqi305/MobileNet-SSD, https://docs.nvidia.com/deeplearning/sdk/cudnn-install/index.html
- GitHub-Issues: https://github.com/BVLC/caffe/issues/1325, https://github.com/BVLC/caffe/issues/4843, https://github.com/opencv/opencv/issues/14205
- Stack Overflow und Ask Ubuntu: https://stackoverflow.com/questions/38680593/importerror-no-module-named-google-protobuf, https://stackoverflow.com/questions/51350998/7515-fatal-error-stdlib-h-no-such-file-or-directory-include-next-stdlib-h, https://askubuntu.com/questions/1025928/why-do-i-get-sbin-ldconfig-real-usr-local-cuda-lib64-libcudnn-so-7-is-not-a
- "Caffe Users" Google-Gruppe: https://groups.google.com/g/caffe-users/

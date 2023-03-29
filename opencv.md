更新本地软件包列表：打开终端，运行以下命令。
sudo apt update
安装依赖项：运行以下命令来安装OpenCV所需的依赖项。
sudo apt install build-essential cmake git pkg-config libgtk-3-dev \
    libavcodec-dev libavformat-dev libswscale-dev libv4l-dev \
    libxvidcore-dev libx264-dev libjpeg-dev libpng-dev libtiff-dev \
    gfortran openexr libatlas-base-dev python3-dev python3-numpy \
    libtbb2 libtbb-dev libdc1394-22-dev
下载OpenCV源代码：进入一个适合的目录，运行以下命令来下载OpenCV源代码。
cd ~/
git clone https://github.com/opencv/opencv.git
cd opencv
git checkout 4.5.4  # 选择需要的版本 
编译和安装OpenCV：运行以下命令来编译和安装OpenCV。
mkdir build
cd build
cmake -D CMAKE_BUILD_TYPE=RELEASE \
    -D CMAKE_INSTALL_PREFIX=/usr/local \
    -D BUILD_opencv_python3=yes \
    -D INSTALL_PYTHON_EXAMPLES=yes \
    -D BUILD_EXAMPLES=yes ..
make -j4   # 使用4个线程编译
sudo make install
验证安装：运行以下命令来验证OpenCV是否已正确安装。如果OpenCV已成功安装，则应该可以看到OpenCV的版本号。
pkg-config --modversion opencv4
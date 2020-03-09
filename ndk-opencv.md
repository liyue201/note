
## Docker编译opencv for Android

为了不污染宿主机，在CI中使用docker来编译android ndk是最佳选择。

####  基础镜像Dockerfile
 
  基础镜像中安装CMake、android-ndk、android-sdk
  
 ```
 FROM  centos
 
 RUN yum  -y install  gcc gcc-c++ kernel-devel make zip unzip wget
 
 RUN wget https://cmake.org/files/v3.14/cmake-3.14.2.tar.gz
 
 RUN tar xvf cmake-3.14.2.tar.gz && cd cmake-3.14.2 && ./bootstrap && make &&  make install
 
 RUN yum remove cmake -y
 
 RUN ln -s /usr/local/bin/cmake /usr/bin/
 
 RUN cp -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
 
 WORKDIR /home
 RUN wget https://dl.google.com/android/repository/android-ndk-r16b-linux-x86_64.zip
 RUN unzip android-ndk-r16b-linux-x86_64.zip
 RUN rm android-ndk-r16b-linux-x86_64.zip
 
 RUN wget https://dl.google.com/android/repository/tools_r25.2.5-linux.zip
 RUN unzip tools_r25.2.5-linux.zip

 ```
 
#### 编译基础镜像

```
docker build . -t ndkr16b-sdk25
```
过程优点漫长，我已经编译好的镜像传到阿里云镜像仓库了，方便以后使用，地址

```
registry.cn-hangzhou.aliyuncs.com/stirlingx/ndkr16b-sdk25
```
 
#### 下载opencv源码

- [opencv-3.4.9](https://github.com/opencv/opencv/archive/3.4.9.zip)  
- [opencv_contrib-3.4.9](https://github.com/opencv/opencv_contrib/archive/3.4.9.zip)

将opencv_contrib-3.4.9解压之后放到opencv-3.4.9目录中。

修改opencv-3.4.9中的CMakeLists.txt，增加一行
```
INCLUDE_DIRECTORIES("/code/opencv_contrib-3.4.9/modules/xfeatures2d/include")
```

- 下载[xfeatures2d模块依赖的文件](https://files-cdn.cnblogs.com/files/arxive/boostdesc_bgm.i,vgg_generated_48.i%E7%AD%89.rar)，
解压后放入  `opencv_contrib-3.4.9/modules/xfeatures2d/src`

#### 添加编译脚本
 
在opencv-3.4.9目录中添加编译脚本build.sh

```
ANDROID_NDK_HOME=/home/android-ndk-r16b
ANDROID_SDK_ROOT=/home/tools
rm -rf build
mkdir build
cd build

cmake -DCMAKE_TOOLCHAIN_FILE="$ANDROID_NDK_HOME/build/cmake/android.toolchain.cmake" \
    -DANDROID_SDK_ROOT=$ANDROID_SDK_ROOT \
    -DANDROID_NDK=$ANDROID_NDK_HOME \
    -DANDROID_NATIVE_API_LEVEL=24 \
    -DANDROID_PLATFORM=android-21 \
    -DANDROID_ABI=armeabi-v7a \
    -DANDROID_CPP_FEATURES="rtti exceptions" \
    -DANDROID_ARM_NEON=TRUE \
    -DANDROID_TOOLCHAIN=clang \
    -DANDROID_STL=c++_static \
    -DCMAKE_BUILD_TYPE=Release \
    -DOPENCV_EXTRA_MODULES_PATH="/code/opencv_contrib-3.4.9/modules"  \
    -DBUILD_ANDROID_PROJECTS=OFF \
    -DBUILD_ANDROID_EXAMPLES=OFF \
    ..
    
make -j 4
```

####  编译

在opencv-3.4.9目录中执行
```
docker run --rm -v $PWD:/code -w /code  registry.cn-hangzhou.aliyuncs.com/stirlingx/ndkr16b-sdk25 ./build.sh 
```


####  参考

- https://www.cnblogs.com/ahfuzhang/p/11069832.html
- https://www.cnblogs.com/arxive/p/11778731.html
- https://blog.csdn.net/qq_33475105/article/details/82819850



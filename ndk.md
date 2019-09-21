## Docker编译Android NDK

使用docker来编译android ndk，支持cmake。

#### Dockerfile

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

```


#### 生成镜像
```
docker build . -t ndk-r16b

```

#### 编译

```
ANDROID_NDK_HOME=/home/android-ndk-r16b

rm -rf build
mkdir build
cd build

cmake -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_TOOLCHAIN_FILE=$ANDROID_NDK_HOME/build/cmake/android.toolchain.cmake \
  -DANDROID_NDK=$ANDROID_NDK_HOME \
  -DANDROID_ABI=armeabi-v7a \
  -DANDROID_TOOLCHAIN=clang \
  -DANDROID_PLATFORM=android-21 \
  -DANDROID_STL=c++_static \
  ..
  
make
```


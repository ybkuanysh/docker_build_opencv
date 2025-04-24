# Сборка OpenCV и FFmpeg в Docker

## FFmpeg
Ссылка с инструкцией из документации: https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu

Устанавливаем необходимые зависимости
```bash
apt-get update -qq && apt-get -y install \
  autoconf \
  automake \
  build-essential \
  cmake \
  git-core \
  libass-dev \
  libfreetype6-dev \
  libgnutls28-dev \
  libmp3lame-dev \
  libsdl2-dev \
  libtool \
  libva-dev \
  libvdpau-dev \
  libvorbis-dev \
  libxcb1-dev \
  libxcb-shm0-dev \
  libxcb-xfixes0-dev \
  meson \
  ninja-build \
  pkg-config \
  texinfo \
  wget \
  yasm \
  nasm \
  zlib1g-dev \
  libx264-dev \
  libx265-dev \
  libnuma-dev \
  libvpx-dev \
  libfdk-aac-dev \
  libopus-dev \
  libdav1d-dev \
  libunistring-dev
```
Создаем папки для сборки
```bash
mkdir -p ~/ffmpeg_sources ~/bin
```
Скачиваем исходники
```bash
cd ~/ffmpeg_sources && \
wget -O ffmpeg-snapshot.tar.bz2 https://ffmpeg.org/releases/ffmpeg-snapshot.tar.bz2 && \
tar xjvf ffmpeg-snapshot.tar.bz2 && \
cd ffmpeg
```
Настраиваем PATH
```bash
PATH="$HOME/bin:$PATH" PKG_CONFIG_PATH="$HOME/ffmpeg_build/lib/pkgconfig"
```
Настраиваем параметры сборки
```bash
./configure \
  --prefix="$HOME/ffmpeg_build" \
  --pkg-config-flags="--static" \
  --extra-cflags="-I$HOME/ffmpeg_build/include" \
  --extra-ldflags="-L$HOME/ffmpeg_build/lib" \
  --extra-libs="-lpthread -lm" \
  --ld="g++" \
  --bindir="$HOME/bin" \
  --enable-gpl \
  --enable-gnutls \
  --enable-libass \
  --enable-libfdk-aac \
  --enable-libfreetype \
  --enable-libmp3lame \
  --enable-libopus \
  --enable-libdav1d \
  --enable-libvorbis \
  --enable-libvpx \
  --enable-libx264 \
  --enable-libx265 \
  --enable-nonfree
```
Собираем и устанавливаем
```bash
PATH="$HOME/bin:$PATH" make -j$(nproc) && \
make install && \
hash -r
```

## OpenCV
Ссылка с инструкцией из документации: https://docs.opencv.org/4.x/d2/de6/tutorial_py_setup_in_ubuntu.html

Устанавливаем необходимые зависимости
```bash
apt-get update && apt-get install -y \
    cmake \
    gcc \
    g++ \
    python3 python3-pip python3-dev \
    libavcodec-dev \
    libavformat-dev \
    libswscale-dev \
    libpng-dev \
    libjpeg-dev \
    libopenexr-dev \
    libtiff-dev \
    libwebp-dev \
    git \
    && rm -rf /var/lib/apt/lists/*
```
Ставим numpy
```
pip install numpy --break-system-packages
```
Переходим в папку /opt и клонируем файлы для сборки
```bash
cd /opt && \
git clone https://github.com/opencv/opencv.git && \
git clone https://github.com/opencv/opencv_contrib.git && \
cd opencv
```
Создаем папку для сборки
```bash
mkdir build && \
cd build
```
Настраиваем параметры сборки
```shell
cmake \
	-D CMAKE_BUILD_TYPE=Release \
    -D CMAKE_INSTALL_PREFIX=/usr/local \
    -D OPENCV_EXTRA_MODULES_PATH=/opt/opencv_contrib/modules \
    -D OPENCV_ENABLE_NONFREE=OFF \
    -D WITH_JPEG=ON \
    -D WITH_PNG=ON \
    -D WITH_TIFF=ON \
    -D WITH_WEBP=ON \
    -D WITH_JASPER=OFF \
    -D WITH_EIGEN=OFF \
    -D WITH_TBB=ON \
    -D WITH_LAPACK=ON \
    -D WITH_PROTOBUF=OFF \
    -D WITH_V4L=OFF \
    -D WITH_GSTREAMER=OFF \
    -D WITH_GTK=OFF \
    -D WITH_QT=OFF \
    -D WITH_CUDA=OFF \
    -D WITH_VTK=OFF \
    -D WITH_OPENEXR=OFF \
    -D WITH_FFMPEG=ON \
    -D WITH_OPENCL=OFF \
    -D WITH_OPENNI=OFF \
    -D WITH_XINE=OFF \
    -D WITH_GDAL=OFF \
    -D WITH_IPP=OFF \
    -D BUILD_OPENCV_PYTHON3=ON \
    -D BUILD_OPENCV_PYTHON2=OFF \
    -D BUILD_OPENCV_JAVA=OFF \
    -D BUILD_TESTS=OFF \
    -D BUILD_IPP_IW=OFF \
    -D BUILD_PERF_TESTS=OFF \
    -D BUILD_EXAMPLES=OFF \
    -D BUILD_ANDROID_EXAMPLES=OFF \
    -D BUILD_DOCS=OFF \
    -D BUILD_ITT=OFF \
    -D INSTALL_PYTHON_EXAMPLES=OFF \
    -D INSTALL_C_EXAMPLES=OFF \
    -D INSTALL_TESTS=OFF \
    ..
```
Собираем и устанавливаем
```shell
make -j$(nproc) && \
make install
```
Проверяем
```bash
python3 -c "import cv2; print(cv2.__version__)"
```
#### Дополнительно
<span style="color:#f00">**Для повторной сборки!!!**</span>
```bash
cd ../ && \
rm -r build && \
mkdir build && \
cd build
```
Чтобы уменьшить Timeout при подключении и считывании, меняем значение в этом файле 
<span style="background-color:#424242; border:1px solid #424242; padding:2px 6px; border-radius:4px;">
modules/videoio/src/cap_ffmpeg_impl.hpp
</span>
```cpp 
#define LIBAVFORMAT_INTERRUPT_OPEN_TIMEOUT_MS 5000
#define LIBAVFORMAT_INTERRUPT_READ_TIMEOUT_MS 5000
```

## Упаковываем все
```bash
cd /usr/local && \
tar -czvf /opt/opencv-install.tar.gz include lib bin share && \
cd "$HOME" && \
tar -czvf ffmpeg-custom.tar.gz bin ffmpeg_build
```
Копируем на хост
```bash
docker cp <container_id>:/opt/opencv-install.tar.gz .
```
Копируем
```bash
docker cp <container_id>:/root/ffmpeg-custom.tar.gz .
```

## Установка в другом контейнере
Доступна по ссылке:
https://github.com/ybkuanysh/docker_unpack_opencv.git
---
sidebar_label: 'OpenCV-Mobile'
sidebar_position: 30
---

# OpenCV-Mobile
プロジェクトリンク:https://github.com/nihui/opencv-mobile  
By Nihui
## 紹介

![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg?style=for-the-badge)
![build](https://img.shields.io/github/actions/workflow/status/nihui/opencv-mobile/release.yml?style=for-the-badge)
![download](https://img.shields.io/github/downloads/nihui/opencv-mobile/total.svg?style=for-the-badge)

![Android](https://img.shields.io/badge/Android-3DDC84?style=for-the-badge&logo=android&logoColor=white)
![iOS](https://img.shields.io/badge/iOS-000000?style=for-the-badge&logo=ios&logoColor=white)
![ARM Linux](https://img.shields.io/badge/ARM_Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Windows](https://img.shields.io/badge/Windows-0078D6?style=for-the-badge&logo=windows&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![MacOS](https://img.shields.io/badge/mac%20os-000000?style=for-the-badge&logo=apple&logoColor=white)
![Firefox](https://img.shields.io/badge/Firefox-FF7139?style=for-the-badge&logo=Firefox-Browser&logoColor=white)
![Chrome](https://img.shields.io/badge/Chrome-4285F4?style=for-the-badge&logo=Google-chrome&logoColor=white)

:heavy_check_mark: このプロジェクトは、**Android**、**iOS**、および**ARM Linux**プラットフォーム向けの OpenCV ライブラリの最小ビルドを提供します。

:heavy_check_mark:  **Windows**、**Linux**、**MacOS**、および **WebAssembly** 向けのパッケージが利用可能です。

:heavy_check_mark: **2.4.13.7**、**3.4.20**、および**4.8.0**の事前ビルドバイナリパッケージを提供します。

:heavy_check_mark: 公式パッケージにはない **ビットコードを有効にしたiOS/iOS-Simulator**用のプレビルド済バイナリパッケージも提供します。

:heavy_check_mark:公式パッケージにはない **Mac-Catalyst**および**Apple xcframework**用のビルド済バイナリパッケージも提供します。

:heavy_check_mark: すべてのバイナリはGitHubアクション上でソースからコンパイルされています。**ウイルスなし**、**バックドアなし**、**秘密のコードなし**。

|opencv 4.8.0 android|package size|
|:-:|:-:|
|The official opencv|189 MB|
|opencv-mobile|17.7 MB|

|opencv 4.8.0 ios|package size|package size with bitcode|
|:-:|:-:|:-:|
|The official opencv|197 MB|missing :(|
|opencv-mobile|9.88 MB|34 MB|

## ダウンロード

https://github.com/nihui/opencv-mobile/releases/latest

[opencv4-milkv-duo-url]: https://github.com/nihui/opencv-mobile/releases/latest/download/opencv-mobile-4.8.0-milkv-duo.zip


* Android パッケージは ndk r25c と android api 24 でビルドされています
* iOS / iOS-Simulator / MacOS / Mac-Catalyst パッケージは Xcode 13.4.1で ビルドされています
* ARM Linux パッケージは Ubuntu-22.04 上のクロスコンパイラでビルドされています
* WebAssembly パッケージは Emscripten 3.1.28 でビルドされています

## ARM Linux、Windows、Linux、WebAssemblyでの使用方法

1. 圧縮ファイルを ```<project dir>/``` に展開します。
2.  ```<project dir>/CMakeListst.txt``` の箇所を見つけて OpenCV へのリンクに変更します
3. Windows でのビルドのためには、cmake option  ```-DOpenCV_STATIC=ON``` を追加します

```cmake
set(OpenCV_DIR ${CMAKE_SOURCE_DIR}/opencv-mobile-4.8.0-armlinux/arm-linux-gnueabihf/lib/cmake/opencv4)
find_package(OpenCV REQUIRED)

target_link_libraries(your_target ${OpenCV_LIBS})
```

## カスタムパッケージをビルドする方法

**ステップ1. OpenCV のソースをダウンロード**
```shell
wget -q https://github.com/opencv/opencv/archive/4.8.0.zip -O opencv-4.8.0.zip
unzip -q opencv-4.8.0.zip
cd opencv-4.8.0
```

**ステップ2. zlib 依存関係を削除し、stb ベースの highgui 実装を使用する (オプション)**
```shell
patch -p1 -i ../opencv-4.8.0-no-zlib.patch
truncate -s 0 cmake/OpenCVFindLibsGrfmt.cmake
rm -rf modules/gapi
rm -rf modules/highgui
cp -r ../highgui modules/
```

**ステップ3. no-rtti ビルドのために opencv ソースにパッチを追加 (オプション)**
```shell
patch -p1 -i ../opencv-4.8.0-no-rtti.patch
```

**step 4. opencv のオプションを opencv4_cmake_options.txt に適用します**

**ステップ5. cmake で opencv パッケージをビルドします**
```shell
mkdir -p build
cd build
cmake -DCMAKE_INSTALL_PREFIX=install \
-DCMAKE_BUILD_TYPE=Release \
`cat ../../opencv4_cmake_options.txt` \
-DBUILD_opencv_world=OFF ..
```

**sステップ6. パッケージを作成します**
```shell
zip -r -9 opencv-mobile-4.8.0.zip install
```

## 注意点

* 最小限の opencv ビルドには、最も基本的な opencv 演算子と一般的な画像処理関数が含まれており、キーポイント特徴の抽出とマッチング、画像修復、オプティカルフロー推定などの便利な追加機能も含まれています。

* 顔検出など、専用モジュールに存在する多くのコンピューター ビジョン アルゴリズムは破棄されます。[モバイル用に最適化されたニューラル ネットワーク推論ライブラリを使用したディープラーニング ベースのアルゴリズムを試すことができます。](https://github.com/Tencent/ncnn)

* `cv::imread`や`cv::imwrite`などの highgui モジュールのイメージ IO 関数は、コードサイズを小さくするために [stb](https://github.com/nothings/stb) を使用して再実装されています。 `cv::imshow`などの GUI 関数は破棄されます。

* cuda と opencl は無効になっています。モバイルには cuda が無く、ios には opencl が無く、android の opencl は遅いためです。 GPU 上の opencv は実際の制作には適していません。優れた GPU アクセラレーションが必要な場合は、iOS では metal、Android では opengles/vulkan を書いてください。

* C++ RTTI と例外は、モバイル プラットフォーム上の最小限のビルドと WebAssembly ビルドでは無効になっています。書くときは気をつけてね```cv::Mat roi = image(roirect);```  :P

## opencv modules に含まれるもの

|module|comment|
|---|---|
|opencv_core|Mat, matrix operations, etc|
|opencv_imgproc|resize, cvtColor, warpAffine, etc|
|opencv_highgui|imread, imwrite|
|opencv_features2d|keypoint feature and matcher, etc (not included in opencv 2.x package)|
|opencv_photo|inpaint, etc|
|opencv_video|opticalflow, etc|

## opencv modules で捨てたもの

|module|comment|
|---|---|
|opencv_androidcamera|use android Camera api instead|
|opencv_calib3d|camera calibration, rare uses on mobile|
|opencv_contrib|experimental functions, build part of the source externally if you need|
|opencv_dnn|very slow on mobile, try ncnn for neural network inference on mobile|
|opencv_dynamicuda|no cuda on mobile|
|opencv_flann|feature matching, rare uses on mobile, build the source externally if you need|
|opencv_gapi|graph based image processing, little gain on mobile|
|opencv_gpu|no cuda/opencl on mobile|
|opencv_imgcodecs|link with opencv_highgui instead|
|opencv_java|wrap your c++ code with jni|
|opencv_js|write native code on mobile|
|opencv_legacy|various good-old cv routines, build part of the source externally if you need|
|opencv_ml|train your ML algorithm on powerful pc or server|
|opencv_nonfree|the SIFT and SURF, use ORB which is faster and better|
|opencv_objdetect|HOG, cascade detector, use deep learning detector which is faster and better|
|opencv_ocl|no opencl on mobile|
|opencv_python|no python on mobile|
|opencv_shape|shape matching, rare uses on mobile, build the source externally if you need|
|opencv_stitching|image stitching, rare uses on mobile, build the source externally if you need|
|opencv_superres|do video super-resolution on powerful pc or server|
|opencv_ts|test modules, useless in production anyway|
|opencv_videoio|use android MediaCodec or ios AVFoundation api instead|
|opencv_videostab|do video stablization on powerful pc or server|
|opencv_viz|vtk is not available on mobile, write your own data visualization routines|

## Milk-V Duo での使うには?
オリジナルリンク: https://community.milkv.io/t/opencv-mobile-opencv-milkv-duo/557

### TL;DR

パッケージ化された zip ファイルをコンパイルして準備を整えます。

1. Webサイト https://github.com/nihui/opencv-mobile を開きます。
2. opencv-mobile-4.8.0-milkv-duo.zip をダウンロードし解凍します。
3. cmake `-DOpenCV_DIR=<unzip_directory>/lib/cmake/opencv4` + `find_package(OpenCV)` + `target_link_libraries(your-program ${OpenCV_LIBS})` を実行します。

### opencv-mobile

opencv-mobile はコンパイル後の opencv ライブラリを最小化するために、コンパイル パラメーターを調整し、一部の opencv ソース コードを削除しています。 

* 画像の読み取り/書き込み、画像処理、行列演算などの一般的な opencv 機能を提供します。 
* アップストリームと同期されており、サードパーティの依存関係はありません。 

ほとんどの場合、公式の opencv を 1/10 のサイズで置き換えることができるため、特別なサイズ要件があるモバイル環境や組み込み環境に特に適しています。

Plain Gopro コミュニティの修正された opencv と比較して、opencv-mobile はアップストリームの RVV 最適化を直接享受でき、より独創的で軽量になります。

### Milkv-duo 用に opencv-mobile をコンパイルする


Milkv-duo ボードのハードウェア リソースは非常に限られています。

- クロスコンパイル ツールチェーンを準備します。たとえば、それを `/home/nihui/osd/host-tools`に解凍し、後で環境変数として設定します。



https://github.com/milkv-duo/duo-buildroot-sdk/tree/develop

- 最新版のopencv のソースコードをダウンロードします。

https://github.com/opencv/opencv/releases

- opencv-mobile のコンパイル設定とパッチをダウンロードします。

https://github.com/nihui/opencv-mobile

opencv-mobile のコンパイル手順に従い、toolchain ディレクトリ環境変数を設定し、highgui モジュールを変更し、パッチを適用し、 `riscv64-unknown-linux-musl.toolchain.cmake`を使用します。

```shell
export RISCV_ROOT_PATH=/home/nihui/osd/host-tools/gcc/riscv64-linux-musl-x86_64

git clone https://github.com/nihui/opencv-mobile.git
cd opencv-mobile

wget -q https://github.com/opencv/opencv/archive/4.8.0.zip
unzip -q opencv-4.8.0.zip
cd opencv-4.8.0

truncate -s 0 cmake/OpenCVFindLibsGrfmt.cmake
rm -rf modules/gapi
patch -p1 -i ../opencv-4.8.0-no-rtti.patch
patch -p1 -i ../opencv-4.8.0-no-zlib.patch
patch -p1 -i ../opencv-4.8.0-link-openmp.patch
rm -rf modules/highgui
cp -r ../highgui modules/

mkdir build
cd build
cmake -DCMAKE_TOOLCHAIN_FILE=../../toolchains/riscv64-unknown-linux-musl.toolchain.cmake -DCMAKE_C_FLAGS="-fno-rtti -fno-exceptions" -DCMAKE_CXX_FLAGS="-fno-rtti -fno-exceptions" -DCMAKE_INSTALL_PREFIX=install -DCMAKE_BUILD_TYPE=Release `cat ../../opencv4_cmake_options.txt` -DBUILD_opencv_world=OFF -DOPENCV_DISABLE_FILESYSTEM_SUPPORT=ON ..
make -j4
make install

```

コンパイル後、opencv-mobile/build/install で opencv-mobile が準備されます。

コンパイル プロセス中、CMake が RVV ベクター サポートを正常に検出して有効にしたことがわかります。これにより、milkv-duo チップが高速化されます。

```txt
-- Performing Test HAVE_CPU_RVV_SUPPORT (check file: cmake/checks/cpu_rvv.cpp)
-- Performing Test HAVE_CPU_RVV_SUPPORT - Success
```

opencv-mobile の toolschains/riscv64-unknown-linux-musl.toolchain.cmake は、c906 カーネルに関連するコンパイル パラメーターをグローバルに有効にし、c906 最適化用に設定します。これらのパラメータはすべての opencv-mobile モジュールのコンパイルに自動的に適用され、最適なパフォーマンスが提供されます。

```txt
-- General configuration for OpenCV 4.8.0 =====================================
--   Version control:               v18-dirty
--
--   Platform:
--     Timestamp:                   2023-08-30T09:37:40Z
--     Host:                        Linux 6.4.12-200.fc38.x86_64 x86_64
--     Target:                      Linux riscv64
--     CMake:                       3.27.3
--     CMake generator:             Unix Makefiles
--     CMake build tool:            /usr/bin/gmake
--     Configuration:               Release
--
--   CPU/HW features:
--     Baseline:                    RVV
--       requested:                 DETECT
--
--   C/C++:
--     Built as dynamic libs?:      NO
--     C++ standard:                11
--     C++ Compiler:                /home/nihui/osd/host-tools/gcc/riscv64-linux-musl-x86_64/bin/riscv64-unknown-linux-musl-g++  (ver 10.2.0)
--     C++ flags (Release):         -march=rv64gcv0p7_zfh_xtheadc -mabi=lp64d -mtune=c906   -fsigned-char -W -Wall -Wreturn-type -Wnon-virtual-dtor -Waddress -Wsequence-point -Wformat -Wformat-security -Wmissing-declarations -Wundef -Winit-self -Wpointer-arith -Wshadow -Wsign-promo -Wuninitialized -Wsuggest-override -Wno-delete-non-virtual-dtor -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -fvisibility=hidden -fvisibility-inlines-hidden -fopenmp -O3 -DNDEBUG  -DNDEBUG
--     C++ flags (Debug):           -march=rv64gcv0p7_zfh_xtheadc -mabi=lp64d -mtune=c906   -fsigned-char -W -Wall -Wreturn-type -Wnon-virtual-dtor -Waddress -Wsequence-point -Wformat -Wformat-security -Wmissing-declarations -Wundef -Winit-self -Wpointer-arith -Wshadow -Wsign-promo -Wuninitialized -Wsuggest-override -Wno-delete-non-virtual-dtor -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -fvisibility=hidden -fvisibility-inlines-hidden -fopenmp -g  -O0 -DDEBUG -D_DEBUG
--     C Compiler:                  /home/nihui/osd/host-tools/gcc/riscv64-linux-musl-x86_64/bin/riscv64-unknown-linux-musl-gcc
--     C flags (Release):           -march=rv64gcv0p7_zfh_xtheadc -mabi=lp64d -mtune=c906   -fsigned-char -W -Wall -Wreturn-type -Waddress -Wsequence-point -Wformat -Wformat-security -Wmissing-declarations -Wmissing-prototypes -Wstrict-prototypes -Wundef -Winit-self -Wpointer-arith -Wshadow -Wuninitialized -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -fvisibility=hidden -fopenmp -O3 -DNDEBUG  -DNDEBUG
--     C flags (Debug):             -march=rv64gcv0p7_zfh_xtheadc -mabi=lp64d -mtune=c906   -fsigned-char -W -Wall -Wreturn-type -Waddress -Wsequence-point -Wformat -Wformat-security -Wmissing-declarations -Wmissing-prototypes -Wstrict-prototypes -Wundef -Winit-self -Wpointer-arith -Wshadow -Wuninitialized -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -fvisibility=hidden -fopenmp -g  -O0 -DDEBUG -D_DEBUG
--     Linker flags (Release):      -Wl,--gc-sections -Wl,--as-needed -Wl,--no-undefined
--     Linker flags (Debug):        -Wl,--gc-sections -Wl,--as-needed -Wl,--no-undefined
--     ccache:                      NO
--     Precompiled headers:         NO
--     Filesystem support is disabled
--     Extra dependencies:          /home/nihui/osd/host-tools/gcc/riscv64-linux-musl-x86_64/riscv64-unknown-linux-musl/lib64v0p7_xthead/lp64d/libgomp.so /home/nihui/osd/host-tools/gcc/riscv64-linux-musl-x86_64/sysroot/usr/lib64v0p7_xthead/lp64d/libpthread.a /home/nihui/osd/host-tools/gcc/riscv64-linux-musl-x86_64/riscv64-unknown-linux-musl/lib64v0p7_xthead/lp64d/libatomic.so dl m pthread rt
--     3rdparty dependencies:
--
--   OpenCV modules:
--     To be built:                 core features2d highgui imgproc photo video
--     Disabled:                    calib3d dnn flann imgcodecs ml objdetect stitching videoio world
--     Disabled by dependency:      -
--     Unavailable:                 java python2 python3 ts
--     Applications:                -
--     Documentation:               NO
--     Non-free algorithms:         NO
--
--   GUI:
--
--   Media I/O:
--     ZLib:                        build (ver )
--
--   Video I/O:
--     DC1394:                      NO
--
--   Parallel framework:            OpenMP
--
--   Other third-party libraries:
--     Lapack:                      NO
--     Custom HAL:                  NO
--
--   Python (for build):            /usr/bin/python2.7
--
--   Install to:                    /home/nihui/dev/opencv-mobile/opencv-4.8.0/build/install
-- -----------------------------------------------------------------
```

### opencv-mobile を使用して milkv-duo の画像スケーリングを実装する

opencv-mobile には、opencv-mobile を使用して画像を読み込み、拡大縮小し、画像を保存する方法を示すサンプル プロジェクトが含まれています。

キーコード：
```cpp
#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>
#include <opencv2/imgproc/imgproc.hpp>

int main()
{
    cv::Mat bgr = cv::imread("in.jpg", 1);
    cv::resize(bgr, bgr, cv::Size(200, 200));
    cv::imwrite("out.jpg", bgr);
    return 0;
}
```

CMake プロジェクト キー コード:
```cmake
project(opencv-mobile-test)
cmake_minimum_required(VERSION 3.5)

# opencv4 requires c++11
set(CMAKE_CXX_STANDARD 11)

# set OpenCV_DIR to where OpenCVConfig.cmake resides
find_package(OpenCV REQUIRED)

add_executable(opencv-mobile-test main.cpp)
target_link_libraries(opencv-mobile-test ${OpenCV_LIBS})
```

コンパイルプロセス:
```shell
export RISCV_ROOT_PATH=/home/nihui/osd/host-tools/gcc/riscv64-linux-musl-x86_64

cd opencv-mobile/test

mkdir build
cd build
cmake -DCMAKE_TOOLCHAIN_FILE=../../toolchains/riscv64-unknown-linux-musl.toolchain.cmake -DCMAKE_BUILD_TYPE=Release -DOpenCV_DIR=/home/nihui/dev/opencv-mobile/opencv-4.8.0/build/install/lib/cmake/opencv4 ..
make
```

(オプション) クロスコンパイル ツールチェーンのストリップ ツールを使用すると、コンパイルされたバイナリのサイズをさらに縮小できます。
```shell
/home/nihui/osd/host-tools/gcc/riscv64-linux-musl-x86_64/bin/riscv64-unknown-linux-musl-strip opencv-mobile-test
```

最新の Milkv-duo イメージをフラッシュし、USB 経由で接続した後、SSH できることを確認します。

https://github.com/milkv-duo/duo-buildroot-sdk/releases

opencv-mobile-test と .jpg のテスト画像を root@192.168.42.1:/root/ にコピーします。

この時点で、SSH シェルで実行すると、libgomp.so.1 が見つからないというエラーが発生する可能性があります。

```txt
[root@milkv]~# ./opencv-mobile-test
Error loading shared library libgomp.so.1: No such file or directory (needed by ./opencv-mobile-test)
Error relocating ./opencv-mobile-test: GOMP_loop_nonmonotonic_dynamic_start: symbol not found
Error relocating ./opencv-mobile-test: omp_get_thread_num: symbol not found
Error relocating ./opencv-mobile-test: omp_set_dynamic: symbol not found
Error relocating ./opencv-mobile-test: GOMP_parallel: symbol not found
Error relocating ./opencv-mobile-test: GOMP_loop_nonmonotonic_dynamic_next: symbol not found
Error relocating ./opencv-mobile-test: GOMP_loop_end_nowait: symbol not found
Error relocating ./opencv-mobile-test: omp_get_max_threads: symbol not found
Error relocating ./opencv-mobile-test: GOMP_loop_nonmonotonic_dynamic_start: symbol not found
Error relocating ./opencv-mobile-test: omp_get_thread_num: symbol not found
Error relocating ./opencv-mobile-test: omp_set_dynamic: symbol not found
Error relocating ./opencv-mobile-test: GOMP_parallel: symbol not found
Error relocating ./opencv-mobile-test: GOMP_loop_nonmonotonic_dynamic_next: symbol not found
Error relocating ./opencv-mobile-test: GOMP_loop_end_nowait: symbol not found
Error relocating ./opencv-mobile-test: omp_get_max_threads: symbol not found
```

ツールチェーンから以下を見つけ `/home/nihui/osd/host-tools/gcc/riscv64-linux-musl-x86_64/sysroot/lib64v0p7_xthead/lp64d/libgomp.so.1.0.0`, コピーして名前を `root@192.168.42.1:/root/libgomp.so.1`にリネームします。次に `LD_LIBRARY_PATH` を追加してプログラムを実行すると、正常に終了するはずです。

```txt
[root@milkv]~# LD_LIBRARY_PATH=. ./opencv-mobile-test
```

「Kill​​ed」エラーが発生した場合は、画像の解像度が大きすぎて、milkv-duo のメモリ容量を超えていることを意味します。この問題を回避するには、より小さい画像を使用してください。

最後に、これが役立つと思われた場合は、スターを付けて共有してください^^


https://github.com/nihui/opencv-mobile






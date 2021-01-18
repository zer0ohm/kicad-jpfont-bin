## 検証で使用した環境
 * OS: Windows 10 Home x64 20H2
   * MSYS2: MINGW64_NT-10.0-19042 <Device_Name> 3.1.7-340.x86_64 2020-11-08 12:32 UTC x86_64 Msys
 * CPU: AMD Ryzen 7 3700X
 * RAM: 16GB

* コンパイラ: gcc/g++ v10.2.0
* KiCad: v5.1.9 Stable

## 下準備
* まずはビルドに必要なパッケージを導入します

```terminal:
pacman -Syuu

pacboy -S --needed \
cairo:x curl:x fftw:x glew:x gnutls:x libxslt:x ngspice:x oce:x openssl:x rtmpdump:x toolchain:x wxPython:x wxWidgets:x zlib:x \
base-devel: cmake:x doxygen:x gcc:x glm:x pkg-config:x python2:x swig:x
```

<!--//
   * [公式ドキュメント](https://docs.kicad.org/doxygen/md_Documentation_development_compiling.html#build_windows)ではBoostのダウングレードを行うようになっていますが、そのままでもビルド出来たため今回は行っていません。しかし何らかの理由で下げなければならない場合は

```terminal:
pacman -Rdd mingw-w64-x86_64-gcc mingw-w64-x86_64-gcc-libs
pacman -U "https://repo.msys2.org/mingw/x86_64/mingw-w64-x86_64-gcc-libs-9.3.0-2-any.pkg.tar.xz" "https://repo.msys2.org/mingw/x86_64/mingw-w64-x86_64-gcc-9.3.0-2-any.pkg.tar.xz"
```
とします。
-->

   * KiCadのバージョンを指定します

   ```terminal:
   KICAD_VERSION=5.1.9
   ```

* 次にKiCad本体のソースコードを入手して展開します

```terminal:
cd $HOME
mkdir ./kicad_merge_jpfont && cd ./kicad_merge_jpfont

wget https://gitlab.com/kicad/code/kicad/-/archive/${KICAD_VERSION}/kicad-${KICAD_VERSION}.tar.gz
tar -xf kicad-${KICAD_VERSION}.tar.gz
```

## 日本語フォントのマージ
* [日本語フォントのマージ手順 - KiCad.jp Wiki](http://wiki.kicad.jp/%E6%97%A5%E6%9C%AC%E8%AA%9E%E3%83%95%E3%82%A9%E3%83%B3%E3%83%88%E3%81%AE%E3%83%9E%E3%83%BC%E3%82%B8%E6%89%8B%E9%A0%86#.E3.83.95.E3.82.A9.E3.83.B3.E3.83.88.E3.83.87.E3.83.BC.E3.82.BF.E3.81.AE.E3.83.9E.E3.83.BC.E3.82.B8.E4.BD.9C.E6.A5.AD)を参考にマージしたファイルを用意します
   * 用意したファイルをKiCadのソースツリー内にコピーします

   ```terminal:
   cp path/to/newstroke_font.cpp path/to/kicad-${KICAD_VERSION}/common/
   ```

## ビルド
* configure時は ```-DCMAKE_PREFIX_PATH=/mingw64 ``` を変更せずに実行して下さい(変更すると依存バイナリが見つけられずに失敗します)
 * また、```-DCMAKE_INSTALL_PREFIX```は任意の場所を指定するか省略して下さい (例: -DCMAKE_INSTALL_PREFIX=/opt/kicad-jpfont-5.1.9/)

```terminal:
mkdir ./kicad_build && cd ./kicad_build

cmake \
    -DCMAKE_BUILD_TYPE=Release \
    -G "MSYS Makefiles" \
    -DCMAKE_PREFIX_PATH=/mingw64 \
    -DCMAKE_INSTALL_PREFIX=/mingw64 \
    -DDEFAULT_INSTALL_PATH=/mingw64 \
    ../kicad-${KICAD_VERSION}

make -j`nproc`
make install
```

## 動作テスト
<!--//
* 完成したバイナリがあるディレクトリにDLL類をコピーします

```terminal:
cp \
/mingw64/bin/*wx*.dll \
/mingw64/bin/*glew*.dll \
/mingw64/bin/*jpeg*.dll \
/mingw64/bin/libcairo*.dll \
/mingw64/bin/*ssl*.dll \
/mingw64/bin/libgomp*.dll \
/mingw64/bin/libstd*.dll \
/mingw64/bin/libgcc*.dll \
/mingw64/bin/libwinpthread-1.dll \
/mingw64/bin/libboost*.dll \
/mingw64/bin/libeay32.dll \
/mingw64/bin/ssleay32.dll \
/mingw64/bin/libpng*.dll \
/mingw64/bin/libpixman*.dll \
/mingw64/bin/libfreetype*.dll \
/mingw64/bin/libfontconfig*.dll \
/mingw64/bin/libharfbuzz*.dll \
/mingw64/bin/libexpat*.dll \
/mingw64/bin/libbz2*.dll \
/mingw64/bin/libglib*.dll \
/mingw64/bin/libiconv*.dll \
/mingw64/bin/zlib*.dll \
/mingw64/bin/libintl*.dll \
/mingw64/bin/libtiff*.dll \
/mingw64/bin/liblzma*.dll \
/mingw64/bin/libpython*.dll \
/mingw64/bin/libxml*.dll \
/mingw64/bin/libxslt*.dll \
/mingw64/bin/libexslt*.dll \
/mingw64/bin/xsltproc.exe \
/mingw64/bin/libcurl*.dll \
/mingw64/bin/libidn*.dll \
/mingw64/bin/libssh*.dll \
/mingw64/bin/libbrotlicommon.dll \
/mingw64/bin/libbrotlidec.dll \
/mingw64/bin/librtmp*.dll \
/mingw64/bin/libgnutls*.dll \
/mingw64/bin/libhogweed*.dll \
/mingw64/bin/libnettle*.dll \
/mingw64/bin/libtasn*.dll \
/mingw64/bin/libp11-kit*.dll \
/mingw64/bin/libgmp*.dll \
/mingw64/bin/libffi*.dll \
/mingw64/bin/libFWOSPlugin.dll \
/mingw64/bin/libPTKernel.dll \
/mingw64/bin/libTK*.dll \
/mingw64/bin/libgraphite2.dll \
/mingw64/bin/libicu*.dll \
/mingw64/bin/libpcre*.dll \
/mingw64/bin/libngspice-0.dll \
/mingw64/bin/libfftw*.dll \
/mingw64/bin/libnghttp2*dll \
/mingw64/bin/libunistring-2.dll \
/mingw64/bin/libreadline7.dll \
/mingw64/bin/libtermcap-0.dll \
/mingw64/bin/libpsl-5.dll \
/mingw64/bin/libcrypto-1_1*.dll \
/mingw64/bin/libzstd.dll \
/mingw64/bin/libssp-0.dll \
/opt/kicad-jpfont-${KICAD_VERSION}/bin
```
-->

* KiCadを開きテキストで日本語入力して正しく表示できたら導入完了です

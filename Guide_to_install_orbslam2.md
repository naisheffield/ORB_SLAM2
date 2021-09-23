# Para Ubuntu 20.04 - Instalación de ORB-SLAM2 - Muy caserito
#### Tomé referencias de https://github.com/Mauhing/ORB_SLAM3/blob/master/README.md
#### Tomé como 'cwd' el directorio donde estaré trabajando, es decir, dónde tendré ORB-SLAM2 y todas las demás carpetas.

## Instalando la última versión de CMake (opcional)
#### Seguí instrucciones desde https://askubuntu.com/questions/355565/how-do-i-install-the-latest-version-of-cmake-from-the-command-line
* Técnicamente no es necesario instalar CMake aparte, yo lo hice porque tenía problemas y no sabía bien de donde venía el drama.

```
sudo apt update
sudo apt install build-essential libtool autoconf unzip wget

sudo apt remove --purge --auto-remove cmake
```

#### Para saber que version y build, ingresar a https://www.cmake.org/download y consultar cuál es la versión más reciente.
#### En mi caso, usé la versión 3.21 build .3 (3.21.3)

```
version=actual_version (3.21 para mí)
build=actual_build (3 para mí)
mkdir ~/temp
cd ~/temp
wget https://cmake.org/files/v$version/cmake-$version.$build.tar.gz
tar -xzvf cmake-$version.$build.tar.gz
cd cmake-$version.$build/

./bootstrap
make -j$(nproc)
sudo make install
```

* Nota: para conocer el número de procesadores que tiene la computadora, ejecutar el comando nproc

* Igual es relativo, a mi me salió nproc=4, sin embargo me dejó utilizar 8. El número de procesadores determina la velocidad de instalación, así que tener paciencia.

```
cmake --version
```

## Instalando Pangolin
### Seguí instruccinoes desde https://github.com/stevenlovegrove/Pangolin

```
git clone --recursive https://github.com/stevenlovegrove/Pangolin.git

cd ~/your_fav_code_directory
git clone --recursive https://github.com/stevenlovegrove/Pangolin.git
cd Pangolin 

./scripts/install_prerequisites.sh recommended

mkdir build && cd build
cmake ..
```

* En este punto obtuve errores de módulos, provenientes de Python, en particular me salió que faltaba el módulo 'setuptools'
* Solucioné (me solucionaron) instalándolo: ``` sudo apt install python3-pip ```

```
cmake --build .

cmake --build . -t pypangolin_pip_install
```

## Instalando OpenCV
### Seguí instrucciones desde https://vitux.com/opencv_ubuntu/

```
cd cwd
sudo apt install build-essential cmake git pkg-config libgtk-3-dev \
libavcodec-dev libavformat-dev libswscale-dev libv4l-dev \
libxvidcore-dev libx264-dev libjpeg-dev libpng-dev libtiff-dev \
gfortran openexr libatlas-base-dev python3-dev python3-numpy \
libtbb2 libtbb-dev libdc1394-22-dev libopenexr-dev \
libgstreamer-plugins-base1.0-dev libgstreamer1.0-dev

mkdir ~/opencv_build && cd ~/opencv_build
git clone https://github.com/opencv/opencv.git

git clone https://github.com/opencv/opencv_contrib.git

cd ~/opencv_build/opencv
mkdir -p build && cd build

cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/usr/local \
-D INSTALL_C_EXAMPLES=ON \
-D INSTALL_PYTHON_EXAMPLES=ON \
-D OPENCV_GENERATE_PKGCONFIG=ON \
-D OPENCV_EXTRA_MODULES_PATH=~/opencv_build/opencv_contrib/modules \
-D BUILD_EXAMPLES=ON ..

make -j(nproc) 

sudo make install

pkg-config --modversion opencv4
```

## Instalando Eigen

```
cd cwd
version=actual_version (3.4 para mí) // Corroborar la version y build más nueva en la página de Eigen https://eigen.tuxfamily.org/index.php?title=Main_Page
build=actual_build (0 para mí)
wget https://gitlab.com/libeigen/eigen/-/archive/$version.$build/eigen-$version.$build.tar.gz
tar -xf eigen-$version.$build.tar.gz
cd eigen-$version.$build
mkdir build_dir && cd build_dir
cmake ..
make install // Si tira error de permisos -> sudo make install
```

## Instalando ORB-SLAM2
### Esto ya es familiar al repo del orbslam en sí

```
cd cwd
git clone https://github.com/raulmur/ORB_SLAM2.git ORB_SLAM2
cd ORB_SLAM2
chmod +x build.sh
./build.sh
```

# Para Ubuntu 20.04 - Instalación de ORB-SLAM2 - Muy caserito

*Antes de empezar, es importante saber que hay varias páginas que ya tienen implementada estas instrucciones, las cuales yo desconocía y por eso armé la mía. Algunas son: https://www.jianshu.com/p/31c95d9a5f97 y https://programmerclick.com/article/4715354567/*

*Aún así, dejé esta guía porque contiene algunas explicaciones que recolecté de otras páginas, sumado a algunos errores que fueron surgiendo y como fui solucionando*

#### Tomé referencias de https://github.com/Mauhing/ORB_SLAM3/blob/master/README.md y de todos lados básicamente
#### Tomé como 'cwd' el directorio donde estaré trabajando (directorio de preferencia), es decir, dónde tendré ORB-SLAM2 y todas las demás carpetas.
#### Estamos asumiendo que este es un Ubuntu recién instalado, no tiene ningún programa aún.

Primeras cosas básicas a instalar:

```
sudo apt install git
sudo apt install gcc
sudo apt install cmake

sudo apt-get update
sudo apt-get -f install
// Esto limpia y actualiza el sistema, lo prepara para poder instalar otras cosas con dependencias.

sudo apt install build-essential libtool autoconf unzip wget
sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev

sudo apt-get install python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libdc1394-22-dev libjasper-dev

sudo apt-get install libglew-dev libboost-all-dev libssl-dev

sudo apt install libeigen3-dev
```

## Instalando la última versión de CMake (opcional pero altamente recomendable)
#### Seguí instrucciones desde https://askubuntu.com/questions/355565/how-do-i-install-the-latest-version-of-cmake-from-the-command-line (o sea, googleé)
* Técnicamente no es necesario instalar CMake aparte, yo lo recomiendo, porque la versión de CMake que Pangolin te instala en el siguiente paso es una versión vieja y trae problemas de compatibilidad más adelante. Si se instala primero CMake, y luego Pangolin, al instalar las dependencias recomendadas por este, el CMake desactualizado se pasa por alto y queda instalado el más nuevo.
Preparamos el ambiente:

```
sudo apt-get update
sudo apt install build-essential libtool autoconf unzip wget
sudo apt remove --purge --auto-remove cmake // Por si ya instalamos CMake antes y queremos empezar de cero
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

* Igual es relativo, a mi me salió nproc=4, sin embargo me dejó utilizar 8 (lo recomiendo). El número de procesadores determina la velocidad de instalación, así que tener paciencia.

```
cmake --version
```

## Instalando Pangolin
### Seguí instrucciones desde https://github.com/stevenlovegrove/Pangolin

```
cd ~/your_fav_code_directory
git clone --recursive https://github.com/stevenlovegrove/Pangolin.git
cd Pangolin 

./scripts/install_prerequisites.sh recommended

mkdir build && cd build
cmake ..
```

Un posible error que puede ocurrir es de módulos provenientes de Python, en particular me salió que faltaba el módulo 'setuptools'.

Solucioné (me solucionaron) instalándolo: ``` sudo apt install python3-pip ``` (luego de eso, se puede instalar módulos de Python con ```pip install```).

```
cmake --build .
cmake --build . -t pypangolin_pip_install

sudo make install
```

## Instalando OpenCV
### Seguí instrucciones desde https://vitux.com/opencv_ubuntu/

```
cd ~
sudo apt install build-essential cmake git pkg-config libgtk-3-dev \
libavcodec-dev libavformat-dev libswscale-dev libv4l-dev \
libxvidcore-dev libx264-dev libjpeg-dev libpng-dev libtiff-dev \
gfortran openexr libatlas-base-dev python3-dev python3-numpy \
libtbb2 libtbb-dev libdc1394-22-dev libopenexr-dev \
libgstreamer-plugins-base1.0-dev libgstreamer1.0-dev

mkdir opencv_build && cd opencv_build

git clone https://github.com/opencv/opencv.git

git clone https://github.com/opencv/opencv_contrib.git

cd opencv
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
git clone https://github.com/Windfisch/ORB_SLAM2.git // Tiene actualizaciones para trabajar con la versión 4 de OpenCV
cd ORB_SLAM2
chmod +x build.sh
./build.sh
```
### A continuación, algunos errores que surgieron y como los solucioné
*Errores de compilación repetidos, del estilo de 'variable not declared in scope'*

Al parecer está relacionado con la flag de c++ utilizada, dentro de ORB_SLAM2/CMakeLists.txt; para solucionar, cambié dentro del archivo donde dijera "c+11" por "c+14"

*Pangolin could not be found because dependency Eigen3 could not be found*

Eliminé el archivo FindEigen3.cmake que estaba dentro de ORB_SLAM2/cmake_modules

*Can't find -lpybind11::embed* O algo por el estilo

Al parecer el módulo pybind11 es necesario en alguna parte del proceso, así que modifiqué el CMakeLists.txt dentro de ORB_SLAM2 y añadí:

```
set(CMAKE_PREFIX_PATH "/usr/local/lib/python3.8/dist-packages/pybind11/share/cmake/pybind11")
find_package(pybind11 REQUIRED)

```
Debajo de ```find_package(Pangolin REQUIRED)```. La localiación del archivo pybind11Config.cmake puede variar (también puede llamarse pybind11-config.cmake), en cualquier caso, se puede buscar en /usr/ y copiar la dirección de la carpeta padre.

Luego de eso, logré instalar ORBSLAM2, aunque con muchos warnings del tipo *deprecated*.
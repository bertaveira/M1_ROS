# Install ROS Foxy/Galatic on Apple Silicon Mac 

Unfortunately no version of ROS currently has binary versions of ROS to install from. Therefore we need to build it from source. ROS foxy/galactic both haven't been design to support ARM architechture on MacOS and some of its' dependencies need some modifications to make it work. Here is a guide on how to make it work.

## Installing dependencies

Here following the official documentation for building from source on MacOS is enough. Despite being in the official documentation here is a copy of it (All the creddit goes to the official documentation [[link](https://docs.ros.org/en/foxy/Installation/macOS-Development-Setup.html)]).

1. Install **Xcode**

   1. Go to app store and search for Xcode and install it (it is over 12Gb so be patient)

   2. Install command line tools by typing the following in the terminal:

      ```bash
      xcode-select --install
      # This command will not succeed if you have not installed Xcode.app
      sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
      # If you installed Xcode.app manually, you need to either open it or run:
      sudo xcodebuild -license
      # To accept the Xcode.app license
      ```

2. Install **brew** (if you haven't already) by following the link http://brew.sh/

3. Install general dependencies

   ```bash
   brew install cmake cppcheck eigen pcre poco tinyxml wget bullet
   
   brew install python@3.8
   brew unlink python
   # Make the python command be Python 3.8
   brew link --force python@3.8
   
   # install dependencies for Fast-RTPS if you are using it
   brew install asio tinyxml2
   
   brew install opencv
   
   # install console_bridge for rosbag2
   brew install console_bridge
   
   # install OpenSSL for DDS-Security
   brew install openssl
   # if you are using ZSH, then replace '.bashrc' with '.zshrc'
   echo "export OPENSSL_ROOT_DIR=$(brew --prefix openssl)" >> ~/.bashrc
   
   # install dependencies for rcl_logging
   brew install log4cxx spdlog
   
   # install CUnit for Cyclone DDS
   brew install cunit
   ```

4. Rviz dependencies

   ```bash
   # install dependencies for Rviz
   brew install qt@5 freetype assimp
   
   # Add the Qt directory to the PATH and CMAKE_PREFIX_PATH
   export CMAKE_PREFIX_PATH=$CMAKE_PREFIX_PATH:/usr/local/opt/qt@5
   export PATH=$PATH:/usr/local/opt/qt@5/bin
   ```

5. rqt dependencies (rqt won't be installed unfortunately)

   ```bash
   brew install graphviz pyqt5 sip
   ```

6. Install python packages

   ```bash
   python3 -m pip install -U \
    argcomplete catkin_pkg colcon-common-extensions coverage \
    cryptography empy flake8 flake8-blind-except flake8-builtins \
    flake8-class-newline flake8-comprehensions flake8-deprecated \
    flake8-docstrings flake8-import-order flake8-quotes ifcfg \
    importlib-metadata lark-parser lxml mock mypy==0.761 netifaces \
    nose pep8 pydocstyle pydot pygraphviz pyparsing \
    pytest-mock rosdep setuptools vcstool rosdistro
   ```



## Build ROS2 from source

After all the dependencies have been dealt with we need to build ROS2 from source.

Unfortunately ROS2 and its' dependencies are not designed for ARM version of MacOS so some changes have to be made to make it possible to build it. Fortunately there had [Kliment Mamykin](http://mamykin.com/) has done an amazing job fixing what needed to be fixed and compiling it all in his blog. Full credit for this section gos to him.

### Download source code

Start by creating a folder where ROS will live and downloading the source code for the desired version (in my case foxy):

```
mkdir -p ~/ros2_foxy/src
cd ~/ros2_foxy
wget https://raw.githubusercontent.com/ros2/ros2/foxy/ros2.repos
vcs import src < ros2.repos
```



### Build code

Enter the `~/ros2_foxy` folder and try to build it using the following command:

```bash
colcon build \
  --symlink-install \
  --merge-install \
  --event-handlers console_cohesion+ console_package_list+ \
  --packages-skip-by-dep python_qt_binding \
  --cmake-args \
    --no-warn-unused-cli \
    -DBUILD_TESTING=OFF \
    -DINSTALL_EXAMPLES=ON \
    -DCMAKE_OSX_SYSROOT=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk \
    -DCMAKE_OSX_ARCHITECTURES="arm64" \
    -DCMAKE_PREFIX_PATH=$(brew --prefix):$(brew --prefix qt@5) 
```

You will most likely encounter build errors. 

#### Patches

The followign patches need to be made in order to make it work (more infor about it check Kliment Mamykin blog [here](http://mamykin.com/)). 

In this section diff commands are shown to indicate what you should change. Lines starting with a `-` are delleted/replaced, and lines starting with `+` are added. The rest of the lines are left as is.

Edit the file `src/osrf/osrf_testing_tools_cpp/osrf_testing_tools_cpp/src/memory_tools/vendor/bombela/backward-cpp/backward.hpp`

```diff
--- a/src/memory_tools/vendor/bombela/backward-cpp/backward.hpp
+++ b/src/memory_tools/vendor/bombela/backward-cpp/backward.hpp
@@ -3927,8 +3927,10 @@ public:
     error_addr = reinterpret_cast<void *>(uctx->uc_mcontext.gregs[REG_EIP]);
 #elif defined(__arm__)
     error_addr = reinterpret_cast<void *>(uctx->uc_mcontext.arm_pc);
-#elif defined(__aarch64__)
+#elif defined(__aarch64__) && !defined(__APPLE__)
     error_addr = reinterpret_cast<void *>(uctx->uc_mcontext.pc);
+#elif defined(__APPLE__) && defined(__aarch64__)
+    error_addr = reinterpret_cast<void *>(uctx->uc_mcontext->__ss.__pc);
 #elif defined(__mips__)
     error_addr = reinterpret_cast<void *>(reinterpret_cast<struct sigcontext*>(&uctx->uc_mcontext)->sc_pc);
 #elif defined(__ppc__) || defined(__powerpc) || defined(__powerpc__) ||        \
```



Edit the file `src/ros2/rviz/rviz_ogre_vendor/CMakeLists.txt`

```diff
diff --git a/CMakeLists.txt b/CMakeLists.txt
index faac7e1b..c36877c3 100644
--- a/CMakeLists.txt
+++ b/CMakeLists.txt
@@ -120,7 +120,7 @@ macro(build_ogre)
     set(OGRE_CXX_FLAGS "${OGRE_CXX_FLAGS} /w /EHsc")
   elseif(APPLE)
     set(OGRE_CXX_FLAGS "${OGRE_CXX_FLAGS} -std=c++14 -stdlib=libc++ -w")
-    list(APPEND extra_cmake_args "-DCMAKE_OSX_ARCHITECTURES='x86_64'")
+    list(APPEND extra_cmake_args "-DCMAKE_OSX_ARCHITECTURES='arm64'")
   else()  # Linux
     set(OGRE_C_FLAGS "${OGRE_C_FLAGS} -w")
     # include Clang -Wno-everything to disable warnings in that build. GCC doesn't mind it
```



Edit the file `build/rviz_ogre_vendor/ogre-v1.12.1-prefix/src/ogre-v1.12.1/OgreMain/include/OgrePlatformInformation.h`

```diff
--- build/rviz_ogre_vendor/ogre-v1.12.1-prefix/src/ogre-v1.12.1/OgreMain/include/OgrePlatformInformation.h.orig	2021-06-02 16:28:58.000000000 -0400
+++ build/rviz_ogre_vendor/ogre-v1.12.1-prefix/src/ogre-v1.12.1/OgreMain/include/OgrePlatformInformation.h	2021-06-02 16:30:50.000000000 -0400
@@ -50,11 +50,11 @@
 #   define OGRE_CPU OGRE_CPU_X86

 #elif OGRE_PLATFORM == OGRE_PLATFORM_APPLE && defined(__BIG_ENDIAN__)
 #   define OGRE_CPU OGRE_CPU_PPC
 #elif OGRE_PLATFORM == OGRE_PLATFORM_APPLE
-#   define OGRE_CPU OGRE_CPU_X86
+#   define OGRE_CPU OGRE_CPU_ARM
 #elif OGRE_PLATFORM == OGRE_PLATFORM_APPLE_IOS && (defined(__i386__) || defined(__x86_64__))
 #   define OGRE_CPU OGRE_CPU_X86
 #elif defined(__arm__) || defined(_M_ARM) || defined(__arm64__) || defined(__aarch64__)
 #   define OGRE_CPU OGRE_CPU_ARM
 #elif defined(__mips64) || defined(__mips64_)
```



Try to build again. It might work.

#### Other fixes that might be needed

If you still get an error about `Qt5` simillar to this:

```bash
CMake Error at CMakeLists.txt:124 (find_package):
  By not providing "FindQt5.cmake" in CMAKE_MODULE_PATH this project has
  asked CMake to find a package configuration file provided by "Qt5", but
  CMake did not find one.

  Could not find a package configuration file provided by "Qt5" with any of
  the following names:

    Qt5Config.cmake
    qt5-config.cmake

  Add the installation prefix of "Qt5" to CMAKE_PREFIX_PATH or set "Qt5_DIR"
  to a directory containing one of the above files.  If "Qt5" provides a
  separate development package or SDK, be sure it has been installed.
```

Then add the flag `-DQt5_DIR=$(brew --prefix qt5)/lib/cmake/Qt5` to the build command like so:

```bash
colcon build \
  --symlink-install \
  --merge-install \
  --event-handlers console_cohesion+ console_package_list+ \
  --packages-skip-by-dep python_qt_binding \
  --cmake-args \
    --no-warn-unused-cli \
    -DBUILD_TESTING=OFF \
    -DINSTALL_EXAMPLES=ON \
    -DCMAKE_OSX_SYSROOT=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk \
    -DCMAKE_OSX_ARCHITECTURES="arm64" \
    -DCMAKE_PREFIX_PATH=$(brew --prefix):$(brew --prefix qt@5) \
    -DQt5_DIR=$(brew --prefix qt5)/lib/cmake/Qt5
```



If you have problem with **assimp** uninstall assimp like this:

```bash
brew uninstall assimp
```



## After installation

Now you have a fully installed ROS2 on M1 Mac. 

Don't forget to source the installation setup file. It is advisable to add it to the rc file so that when you launch a terminal window it automatically soiurces the ROS environment. 

First you need to know if your terminal is using **bash** or **zsh**. Then add the following line to the `~/.bashrc` or `~/.zshrc` file (depending on which you are useing)

```bash
source ~/ros2_foxy/install/setup.bash       # If using bash
source ~/ros2_foxy/install/setup.zsh        # If using zsh
```


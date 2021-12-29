# Update ROS built from source

> This guide was based on the official documentation: https://docs.ros.org/en/foxy/Installation/Maintaining-a-Source-Checkout.html

To update just go to your ROS folder and replace the `.repos` file with a new updated one and then build with the same command used before. You might need to reapply the patches you had to for installing in the first place. 

So to download the new updated source code do these commands (in my case for foxy but change accordingly):

```bash
cd ~/ros2_ws
mv -i ros2.repos ros2.repos.old
wget https://raw.githubusercontent.com/ros2/ros2/foxy-release/ros2.repos
```



Then just build with the same command used in the [Install_ROS](Install_ROS.md) guide:

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

If any errors occur from trying to build check the `Patches` section of my guide on [how to install ROS on M1 Mac](Install_ROS.md).
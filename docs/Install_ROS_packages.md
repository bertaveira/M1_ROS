# Install ROS packages from source

One advantages of ROS is the vast index of packages shared and maintained by the community. One of the most annoying things bout using an unsupported platform is the lack of package binaries to easely install from with a single command. 

To make up for this I made a command to build from source and install in the correct folder a ROS package.

## Single command build + install

To acomplish this I made an `alias` where if I type the command `ros_pkg_install` it will try to build and install the code in the folder where I invoque the command from.

Add this line to your `~/.bashrc` or `~/.zshrc` file:

```bash
alias ros_pkg_install="colcon build --install-base /Users/<username>/ros2_foxy/install --merge-install"
```

Mkae sure to change the path to the path where your ROS is located. Add your username to the path instead of `<username>` and if you have a different name for your ROS installations folder change that as well.

## Install packages

Start by searching on https://index.ros.org/ for the desired package. Once you have one you want to install open the link to the GitHub repository. Clone it in a folder of your own choice (in my case I made a folder called ROS_packages under my documents). 

```bash
git clone <url>
```

Then make sure you are on the right branch. You probably need to be in a branch for the ROS version you want. In my case since I use foxy the common names for the branches are `foxy` or `foxy-devel`. Se on github the branches and then use the following command under the cloned folder to change:

```bash
git checkout foxy-devel
```

Check for dependencies:

```bash
rosdep install --from-paths .
```

If dependencies fail you might need to build them from source... it depends but if it is a ROS package then you should do the same for that dependencie first.

Finally use command

```bash
ros_pkg_instas
```

This should build and install the package of the folder you are currently in.
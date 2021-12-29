# :computer:  M1 ROS knowledge base

This repository was created to share my experience using ROS on an M1 base MacBook. Feel free to make suggestions and pull requests as you see fit.

I am using a M1 Pro machine so this documentations has been tested on that.

## Index

Guides:

- [Installing ROS Foxy/Galatic](docs/install_ROS)
  - Update ROS
- Installing ROS packages from index.ros.org
  - Installing PCL for ROS
- Using docker
  - Using ROS nodes in docker containers - Using docker containers helps a lot to make the code easely deployable and manageable on the target machine. The problem is that docker on MacOS runs the linux kernal on a VM making it not possible to have the `network=host` setting. To use Rviz and other tools some adaptations are needed
  - Docker to make a Development environment - with vscode and all the dependencies
- MacOS connected to virtual machine

## Development Environment 

If you want to develop ROS based code on your personal M1 Mac you have 3 option:

- Install ROS on macos - See the guide [here](docs/install_ROS)
- Docker dev env - See guide here
- Virtual Machine
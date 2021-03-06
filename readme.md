# Pointcloud Stitching for ARENA [![Build Status](https://travis-ci.com/ABalanuta/pointcloud_stitching.svg?branch=master)](https://travis-ci.com/ABalanuta/pointcloud_stitching)

## Overview
Scalable, multicamera distributed system for realtime pointcloud stitching in the ARENA (Augmented Reality Edge Network Area). This program is currently designed to use the **D400 Series Intel RealSense** depth cameras. Using the [Librealsense 2.0 SDK](https://github.com/IntelRealSense/librealsense), depth frames are grabbed and pointclouds are computed on the edge, before sending the raw XYZRGB values to a central computer over a TCP sockets. The central program stitches the pointclouds together and displays it a viewer using [PCL](http://pointclouds.org/) libraries.

This system has been designed to allow 10-20 cameras to be connected simultaneously. Currently, our set up involves each RealSense depth camera connected to its own Intel i7 NUC computer. Each NUC is connected to a local network via ethernet, as well as the central computer that will be doing the bulk of the computing. Our current camera calibration method is to use a Theodolite, a survey precision instrument, to obtain real world coordinates of each camera (this will be updated soon I hope).

## Installation
Different steps of installation are required for installing the realsense camera servers versus the central computing system. The current instructions are for running on Ubuntu 18.04.1 LTS.

#### Camera servers on the edge
- Go to [Librealsense Github](https://github.com/IntelRealSense/librealsense) and follow the instructions to install the Librealsense 2.0 SDK
- Ensure that your cmake version is 3.1 or later. If not, download and install a newer version from the [CMake website](https://cmake.org/download/)
- Navigate to this *pointcloud_stitching* repository<br />
`mkdir build && cd build`<br />
`cmake ..`<br />
`make && sudo make install`
- Install OpenCV
`sudo apt install libopencv-dev`
- Install OPENGL
`sudo apt-get install libglfw3-dev libgl1-mesa-dev libglu1-mesa-dev`

#### Central computing system
- Follow the instructions to download and install PCL libraries from their [website](http://www.pointclouds.org/documentation/tutorials/compiling_pcl_posix.php).
- Install python3 and other packages (Needed to run the camera-registration script)<br />
`sudo apt-get update`<br />
`sudo apt-get install python3.6`<br />
`sudo apt-get install python3-pip`<br />
`python3 -m pip install --user scipy`<br />
`python3 -m pip install --user numpy`<br />
`python3 -m pip install --user pandas`
- Navigate to this *pointcloud_stitching* repository<br />
`mkdir build && cd build`<br />
`cmake .. -DBUILD_CLIENT=true`<br />
`make && sudo make install`

## Usage
Each realsense is connected to an Intel i7 NUC, which are all accessible through ssh from the ALAN (central) computer. To start running, go through each ssh connection and run pcs-camera-server. If the servers are setup correctly, each one should say "Waiting for client...". Then on the ALAN computer, run "pcs-multicamera-client -v" to begin the pointcloud stitching (-v for visualizing the pointcloud). For more available options, run "pcs-multicamera-client -h" for help and an explanation of each option.

Build Camera Server
`mkdir build && cd build && cmake .. && make && make install`

## Optimized Code
To run the optimized version of pcs-camera-server, you will want to run pcs-camera-optimized. This contains benchmarking tools that show the runtime of the optimized version of processing the depth frames, performing transforms, and then converting the values. It also includes the theoretical FPS, but this is calculated without taking in to consideration the time it takes to grab a frame from the realsense as well as the time it takes to send the data over the network. To run the optimized code, run `pcs-camera-optimized`<br />
- Usage:<br />
  `-f <file> sample data set to run`<br />
  `-s        send data to central camera server if available`<br />
  `-m        use SIMD instructions`<br />
  `-t <n>    use OpenMP with n threads`<br />
  

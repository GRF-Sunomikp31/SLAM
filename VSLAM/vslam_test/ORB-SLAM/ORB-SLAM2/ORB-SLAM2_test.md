# ORB-SLAM2测试文档

本文档记录ORB-SLAM各个版本实际测试过程以及问题反馈；

## 测试过程

### ORB-SLAM2

参考资料：

- https://blog.csdn.net/qq_41667348/article/details/114902844
- https://github.com/raulmur/ORB_SLAM2

直接编译报错如参考资料[1]，按照方法解决问题。

上面文章没有讲述如何下数据集；

kitty数据集下载：TUM数据集下载：https://blog.csdn.net/qq_34591921/article/details/105568391

这里关于数据集的**问题**：官方kitty数据集是一个很大的数据集。

#### 单目运行



**Kitty数据集**：

也是在上面的目录下：

`./Examples/Monocular/mono_kitti Vocabulary/ORBvoc.txt Examples/Monocular/KITTI00-02.yaml Data/kitti_Image`` 

正常运行。

**调用摄像头（在线）**：

**单目**

https://blog.csdn.net/qq_17232031/article/details/79519695   //关于调用摄像头配置

现在的问题是：我直接将之前编译的ORB-SLAM2（编译成功的，修改了sleep的bug）直接放到ROS工作空间下面，添加了`-lboost_system`，报错如下：

![2](https://github.com/GRF-Sunomikp31/SLAM/blob/main/VSLAM/vslam_test/ORB-SLAM/ORB-SLAM2/IMG/2.png)

具体报错如下：

`[rosbuild] Building package ORB_SLAM2`
`Failed to invoke /opt/ros/melodic/bin/rospack deps-manifests ORB_SLAM2`
`Traceback (most recent call last):`
  `File "/usr/lib/python2.7/dist-packages/rosdep2/rospack.py", line 59, in init_rospack_interface`
    `lookup = _get_default_RosdepLookup(Options())`
  `File "/usr/lib/python2.7/dist-packages/rosdep2/main.py", line 134, in _get_default_RosdepLookup`
    `verbose=options.verbose)`
  `File "/usr/lib/python2.7/dist-packages/rosdep2/sources_list.py", line 603, in create_default`
    `sources = load_cached_sources_list(sources_cache_dir=sources_cache_dir, verbose=verbose)`
  `File "/usr/lib/python2.7/dist-packages/rosdep2/sources_list.py", line 561, in load_cached_sources_list`
    `with open(cache_index, 'r') as f:`
`IOError: [Errno 13] Permission denied: '/home/gipsy/.ros/rosdep/sources.cache/index'`
`[rospack] Error: could not call python function 'rosdep2.rospack.init_rospack_interface'`


`CMake Error at /opt/ros/melodic/share/ros/core/rosbuild/public.cmake:129 (message):`


  `Failed to invoke rospack to get compile flags for package 'ORB_SLAM2'.`
  `Look above for errors from rospack itself.  Aborting.  Please fix the`
  `broken dependency!`

`Call Stack (most recent call first):`
  `/opt/ros/melodic/share/ros/core/rosbuild/public.cmake:207 (rosbuild_invoke_rospack)`
  `CMakeLists.txt:4 (rosbuild_init)`

`-- Configuring incomplete, errors occurred!`
`See also "/home/gipsy/catkin_ws/src/ORB_SLAM2/Examples/ROS/ORB_SLAM2/build/CMakeFiles/CMakeOutput.log".`
`make: *** No targets specified and no makefile found.  Stop.`

这个问题应该是我`rosdep update`不能执行的问题；另一个方面反应，ORB-SLAM2是需要rosdep update的。

参考：https://blog.csdn.net/qq_44847636/article/details/115610906配置完rosdep，再`source ~/.bashrc`，出现以下问题

参考这个博客：https://blog.csdn.net/weixin_40425428/article/details/81559679

运行相机启动节点，有如下报错：

`ERROR: cannot launch node of type [usb_cam/usb_cam_node]: Cannot locate node of type [usb_cam_node] in package [usb_cam]. Make sure file exists in package path and permission is set to executable (chmod +x)`

解决方法：这里我对catkin_ws工作空间重新变异以下就可以了。

查看相机设备：`/dev/video`  0 1 2 ；

demo编译运行过程：

`roscore`

`roslaunch usb_cam usb_cam-test.launch`

`rosrun ORB_SLAM2 MonoAR Vocabulary/ORBvoc.txt Examples/ROS/ORB_SLAM2/Asus.yaml`   //这个是AR版本的

rosrun ORB_SLAM2 Mono Vocabulary/ORBvoc.txt Examples/ROS/ORB_SLAM2/Asus.yaml   //这个是正常版本的，这里在ORB-SLAM2路径下运行这个demo是可以的；

建议运行以下版本，加绝对路径的：

 `rosrun ORB_SLAM2 Mono /home/gipsy/catkin_ws/src/ORB_SLAM2/Vocabulary/ORBvoc.txt /home/gipsy/catkin_ws/src/ORB_SLAM2/Examples/ROS/ORB_SLAM2/`

但是现在问题是运行上面代码，显示界面，但是没有特征点:

![4](https://github.com/GRF-Sunomikp31/SLAM/blob/main/VSLAM/vslam_test/ORB-SLAM/ORB-SLAM2/IMG/4.png)

现在怀疑ORB-SLAM2这个节点没有正确订阅到相机的节点；Mono订阅的是/camera/image_raw话题，但是目前这个相机节点只会发布/camera/image_raw节点数据，现在的方法就是修改Mono中的节点订阅信息；在catkin_ws/src/ORB_SLAM2/Examples/ROS/ORB_SLAM2/src的ros_mono.cc  修改如下：

ros::Subscriber sub = nodeHandler.subscribe("/usb_cam/image_raw", 1, &ImageGrabber::GrabImage,&igb);修改之后
#ros::Subscriber sub = nodeHandler.subscribe("/camera/image_raw", 1, &ImageGrabber::GrabImage,&igb); 修改之前

修改之后，需要重新编译以下ROS下的ORB-SLAM2文件；

运行成功：

![3](https://github.com/GRF-Sunomikp31/SLAM/blob/main/VSLAM/vslam_test/ORB-SLAM/ORB-SLAM2/IMG/3.png)

参考资料：

- https://blog.csdn.net/weixin_45168199/article/details/107106811
- https://blog.csdn.net/weixin_43665653/article/details/103773436

#### RGB-D运行

数据集：

**TUM数据集**：

在`/Desktop/test/ORBSLAM2/ORB_SLAM2`下运行：

`./Examples/Monocular/mono_tum Vocabulary/ORBvoc.txt Examples/Monocular/TUM1.yaml Data/rgbd_dataset_freiburg1_xyz/`

正常运行。

期间，出现一个小问题，运行上述代码没反应，后来看了才知道我的数据集文件地址写错了。

**摄像头**：设备为D435i

使用在ROS下使用435i跑数据集，ORB-SLAM2上并没用使用IMU;

按道理说：这里使用intel realsense435i的ROS跑，发布相应的话题，然后相应的ORB-SLAM2节点订阅这个话题就可以了；

##### D435i跑ORB-SLAM2

###### intel realsenseD435i的依赖：

参考官方的依赖安装方法，其中包括两步，第一步安装SDK依赖，第二部安装ros包；

https://github.com/IntelRealSense/realsense-ros

编译ROS包的时候报错：

`-- Could NOT find ddynamic_reconfigure (missing: ddynamic_reconfigure_DIR)`
`-- Could not find the required component 'ddynamic_reconfigure'. The following CMake error indicates that you either need to install the package with the same name or change your environment so that it can be found.`

解决方法：https://blog.csdn.net/weixin_44401286/article/details/102943016

运行下面脚本报错：`roslaunch realsense2_camera rs_rgbd.launch`

`Resource not found: rgbd_launch`
`ROS path [0]=/opt/ros/melodic/share/ros`
`ROS path [1]=/home/gipsy/catkin_ws/src`
`ROS path [2]=/opt/ros/melodic/share`
`The traceback for the exception was written to the log file`

解决方法：重新编译以下：

`catkin_make clean`
`catkin_make -DCATKIN_ENABLE_TESTING=False -DCMAKE_BUILD_TYPE=Release`
`catkin_make install`

`echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc`
`source ~/.bashrc`

这时候运行能进去，但是还是有bug：

`[camera/realsense2_camera_manager-2] process has died [pid 11874, exit code -11, cmd /opt/ros/melodic/lib/nodelet/nodelet manager __name:=realsense2_camera_manager __log:=/home/gipsy/.ros/log/0adc83c8-a69d-11eb-9ac0-505bc2cc4003/camera-realsense2_camera_manager-2.log].`
`log file: /home/gipsy/.ros/log/0adc83c8-a69d-11eb-9ac0-505bc2cc4003/camera-realsense2_camera_manager-2*.log`

现在：roslaunch realsense2_camera rs_rgbd.launch

和：roslaunch realsense2_camera rs_camera.launch  都有报错！！！！

问题解决：https://blog.csdn.net/qq_44847636/article/details/116200626

现在就是roslaunch能进去，但是还是有报错：

![5](https://github.com/GRF-Sunomikp31/SLAM/blob/main/VSLAM/vslam_test/ORB-SLAM/ORB-SLAM2/IMG/5.png)

我参考这个`https://blog.csdn.net/fyf8733/article/details/107382599/`  删除了下面四个包：

![6](https://github.com/GRF-Sunomikp31/SLAM/blob/main/VSLAM/vslam_test/ORB-SLAM/ORB-SLAM2/IMG/6.png)

但还是有问题；

sudo apt-get install ros-melodic-ddynamic-reconfigure

然后又操作了以下就可以了；

https://github.com/IntelRealSense/realsense-ros/issues/1824

现在不报错，但是运行之后： 

`27/04 08:16:19,880 WARNING [140520617850624] (messenger-libusb.cpp:42) control_transfer returned error, index: 768, error: No data available, number: 61`

有这个提示；

https://blog.csdn.net/sinat_16643223/article/details/115289206

https://blog.csdn.net/sinat_16643223/article/details/115322425

###### ORB-SLAM2测试

`roslaunch realsense2_camera rs_rgbd.launch` 

ORB-SLAM2下：

`rosrun ORB_SLAM2 RGBD Vocabulary/ORBvoc.txt Examples/RGB-D/TUM1.yaml`

这里调试的时候一定要看清这些节点订阅/发布的都是什么话题；

 roslaunch realsense2_camera rs_camera.launch  然后rviz打开之后，接的坐标系的map要改。

后面参考这个：https://blog.csdn.net/sinat_16643223/article/details/109477874

**记得：修改完ros_rgb.cc文件之后，记得./build_ros.sh重新编译，在catkin_make**

![9](https://github.com/GRF-Sunomikp31/SLAM/blob/main/VSLAM/vslam_test/ORB-SLAM/ORB-SLAM2/IMG/9.png)

成功了！

参考资料：

- https://blog.csdn.net/sinat_16643223/article/details/109477874
- https://blog.csdn.net/qq_36898914/article/details/88780649
- https://blog.csdn.net/weixin_45702256/article/details/109582400

#### 双目

##### 数据集：

##### 摄像头：

这里没有摄像头 ==

## 总结

1、ORB-SLAM2有很多版本，rgbd的是运行RGB相机的版本；mono是单目版本的；stereo是对应双目版本的；各个版本都有应对ROS版本的，ROS版本是真实环境直接跑摄像头的；还有对应的数据集版本。

2、双目没测试，双目的仿真呢？还有 就是那些数据集文件如何跑仿真的以及数据集文件的具体内容；

## 思考

1、`./Examples/Monocular/mono_tum Vocabulary/ORBvoc.txt Examples/Monocular/TUMX.yaml PATH_TO_SEQUENCE_FOLDER`

这行命令行中的`PATH_TO_SEQUENCE_FOLDER`是什么意思?

这里其实是官网给的命令规范。命令书写参考。

官方给出的命令格式如下：
 PATH_TO_SEQUENCE_FOLDER为数据集的存储路径。
 tumx.yaml与数据集对应，比如TUM1.yaml、TUM2.yaml 、TUM3.yaml 分别对应 freiburg1、freiburg2 、 freiburg3。

2、SLAM算法在线和离线有什么区别？

离线是指跑数据集，在线是指使用相机设备实时测试。

为什么在线跑数据集就需要ROS，是因为相关接口吗？

为什么SLAM要引进ROS?

3、这里跑完ORB-SLAM2摄像头之后，有几个问题

- 因为我看Mono是在单目灰度版本图像上加的特征点，这里ORB-SLAM2使用的是RGB图像还是直接用的灰度图像
- 这里我看很多教程强调摄像头不要旋转，这里直接用摄像头跑是没有旋转不变性的吗？

4、evo分析工具的使用

5、跑rtabmap架构

https://blog.csdn.net/weixin_45702256/article/details/109582400

https://blog.csdn.net/a395381306/article/details/101521675

6、D435i跑VINS

https://blog.csdn.net/weixin_44580210/article/details/89789416

7、单目AR版本的ORB-SLAM2

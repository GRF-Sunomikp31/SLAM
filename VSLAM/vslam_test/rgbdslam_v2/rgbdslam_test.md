# rgbdslam_v2

## 配置过程

这里rgbdslam_v2太老了，就只更新了Kinect的版本，没有melodic的版本；目前只找到一个melodic的教程：https://blog.csdn.net/XindaBlack/article/details/102499364，但是设备用的是kinectV1，后面还要参考一个教程修改成v2版本；

自己在github更新一个melodic版本的rgbdslam_v2；

这里有一个问题，rgbdslam_v2官方只给了indigo、Kinect等版本，没有melodic版本的，这个不同版本的程序有什么区别，只是依赖不同吗？

### 利用数据集跑rgbdslam_v2

#### 环境配置

##### 使用indigo包

![1](https://github.com/GRF-Sunomikp31/SLAM/blob/main/VSLAM/vslam_test/rgbdslam_v2/IMG/1.png)

这里rosdep update有一个报错，先暂时跳过；

![2](https://github.com/GRF-Sunomikp31/SLAM/blob/main/VSLAM/vslam_test/rgbdslam_v2/IMG/2.png)

这里用catkin_make编译工作空间的时候，有报错，这个报错应该是说这个版本的rgbdslam_2依赖的是qt4；

这里停以下进度，查看各个版本的软件依赖关系；

- indigo：pcl-1.8，opencv3.2.0，opencv_contrib-3.2.0,g2o

- Kinect：

`sudo apt-get install qt4*`  安装qt4；

![3](https://github.com/GRF-Sunomikp31/SLAM/blob/main/VSLAM/vslam_test/rgbdslam_v2/IMG/3.png)

这个错误的 原因是：

##### 使用kinetic包

![4](https://github.com/GRF-Sunomikp31/SLAM/blob/main/VSLAM/vslam_test/rgbdslam_v2/IMG/4.png)

同样报错；

这里利用`catkin_make`太卡了，利用`catkin_make -j2`加速；

![5](https://github.com/GRF-Sunomikp31/SLAM/blob/main/VSLAM/vslam_test/rgbdslam_v2/IMG/5.png)

这里编译报错都是 ） 的错误，涉及到的函数有： g2o::BlockSolverX(linearSolver)、g2o::OptimizationAlgorithmDogleg(solver_ptr)、SlamBlockSolver(linearSolver)、g2o::OptimizationAlgorithmLevenberg(solver)、round(float d)

分析：这里主要报错原因是：两个常见错误；

解决方法:

- home/gipsy/rgbdslam_k_catkin_ws/src/rgbdslam_v2-kinetic/src/misc.cpp：需要修改内联函数round()为函数名ROUND()，804、880、881、1015、1016；
- /home/gipsy/rgbdslam_k_catkin_ws/src/rgbdslam_v2-kinetic/src/graph_manager.cpp；g2o版本问题，需要使用作者提供的g2o库；

这里装一下作者的g2o：

```bash
#卸载ROS自带的g2o安装包
sudo apt-get purge ros-melodic-libg2o libqglviewer-dev
#安装依赖项
sudo apt-get install libsuitesparse-dev libeigen3-dev

#删除之前安装的g2o
sudo rm -rf /usr/local/include/g2o
sudo rm -rf /usr/local/lib/libg2o_*

#我这里在src文件夹下
cd rgbdslam_vs/src
git clone https://github.com/felixendres/g2o.git
cd ~/rgbdslam/src/g2o
mkdir build
cd build
cmake ..
make 
sudo make install
```

重新装好g2o之后，`catkin_make -j2`成功编译：

![6](https://github.com/GRF-Sunomikp31/SLAM/blob/main/VSLAM/vslam_test/rgbdslam_v2/IMG/6.png)

启动之前，先source以下：`source ~/rgbdslam_k_catkin_ws/devel/setup.bash` 

启动rgbdslam节点：`roslaunch rgbdslam rgbdslam.launch`

有如下报错：

![7](https://github.com/GRF-Sunomikp31/SLAM/blob/main/VSLAM/vslam_test/rgbdslam_v2/IMG/7.png)

主要报错信息： `error while loading shared libraries: libg2o_ext_freeglut_minimal.so: cannot open shared object file: No such file or directory`

报错原因：这里g2o安装好之后需要配置环境，操作流程如下：

```bash
#打开文件
sudo gedit /etc/ld.so.conf
#添加g2o库的位置, 默认位置如下: 直接在文本末尾添加即可
/usr/local/lib
#再敲入
sudo ldconfig
```

重新运行程序，成功启动：

![8](https://github.com/GRF-Sunomikp31/SLAM/blob/main/VSLAM/vslam_test/rgbdslam_v2/IMG/8.png)

#### 配置数据集

在下述链接下载数据集：http://vision.in.tum.de/data/datasets/rgbd-dataset/download#freiburg1_xyz

![9](https://github.com/GRF-Sunomikp31/SLAM/blob/main/VSLAM/vslam_test/rgbdslam_v2/IMG/9.png)

后面需要按照他的配置修改以下launch文件：https://blog.csdn.net/XindaBlack/article/details/102499364

配置成功：

![10](https://github.com/GRF-Sunomikp31/SLAM/blob/main/VSLAM/vslam_test/rgbdslam_v2/IMG/10.png)

### 利用D435i跑rgbdslam_v2

这里参考：https://blog.csdn.net/qq_42145185/article/details/105651527   

使用的launch文件为：rgbdslam_v1.launch

第一次运行的时候，有如下报错，意思就是深度图和rgbd图不匹配：

![11](https://github.com/GRF-Sunomikp31/SLAM/blob/main/VSLAM/vslam_test/rgbdslam_v2/IMG/11.png)

但是第二次重新运行就好了：

![12](https://github.com/GRF-Sunomikp31/SLAM/blob/main/VSLAM/vslam_test/rgbdslam_v2/IMG/12.png)

## 知识点总结

Associate.txt文件：匹配RGB图像和深度图

这里rgbdslam_v2用自己摄像头跑好卡，电脑还卡死了一次；

电脑提示root空间不足；

下一步工作：整理资料上传到github，研究数据集跑八叉树；

## 参考资料

数据集跑rgbdslam_v2

-  https://blog.csdn.net/zhuoyueljl/article/details/78536996

D435i跑rgbdslam_v2：

- https://blog.csdn.net/weixin_44350238/article/details/103630188

- 

- https://blog.csdn.net/qq_28467367/article/details/92832685
- https://blog.csdn.net/weixin_30556161/article/details/95807216

kinect跑rgbdslam_v2：

- https://blog.csdn.net/qq_37611824/article/details/84896718

Ubuntu18.04跑rgbdslam_2:

- https://blog.csdn.net/XindaBlack/article/details/102499364

## 其他思考

这里可以用那个乐视的摄像头跑rgbdslam_v2吗

RTAB-MAP：http://wiki.ros.org/rtabmap_ros  这个好玩 https://blog.csdn.net/yrc19950911/article/details/94574517

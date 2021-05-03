# 一般的点云或者稠密点云转为八叉树（OctoMap）

## 基本信息

什么是八叉树：

什么是八叉树地图：

![2](C:\Users\lenovo\Desktop\八叉树\IMG\2.png)

大搞意思就是将一个空间，很多次八等分，再细分进而让地图更加精确；最简单的， 八叉树的节点存储了它是否被占据的信息。可以用0表示空白，1表示占据，当然，现在很多方式都是通过多次观察用概率的方法给出这个点的占据信息。

八叉树地图能做什么：是一种在导航中比较常用的，本身有较好的压缩性能的地图形式。

这里有个问题，非正方形，比如矩形这种利用八叉树也是按照严格的八等分吗？

## 八叉树导航

如何使用八叉树地图导航：

栅格地图：

三维栅格地图：

点云地图：

## 现在任务

将点云地图转换为八叉树地图：实现的思路是将点云数据通过ROS发布到某个topic上面比如"/outputCloud",再启动 octomap 节点将数据读入该topic并发布到另一个新的topic 上面去。最后在rviz 里面接收这个新topic 达到实时显示的目的。

参考资料：

- https://blog.csdn.net/crp997576280/article/details/74605766
- https://www.freesion.com/article/8792431661/



## 实现过程记录

```bash
#安装octomap
sudo apt-get install ros-melodic-octomap-ros 
sudo apt-get install ros-melodic-octomap-msgs 
sudo apt-get install ros-melodic-octomap-server
#安装octomap 在 rviz 中的插件
 sudo apt-get install ros-melodic-octomap-rviz-plugins
```

安装好之后，启动rviz，这时候这个模块会多一个octo打头的模组.如下图所示:

![3](/home/gipsy/Desktop/八叉树/IMG/3.png)

这里现在的任务是查看，rgbdslam_v2发布的点云数据的通过什么话题发送的，然后在launch文件中修改相应的点云接受话题，转换显示即可；

利用rviz查看如下：输出点云话题节点为：/rgbdslam/online_clouds    ,图像话题为：/camera/rgb/image_color

![4](/home/gipsy/Desktop/八叉树/IMG/4.png)

launch文件中：文件里面有的frame_id 和 remap topic 的值必须和发布节点中的frame_id以及数据发布的topic一致。

frame_id 为传感器输出摄像头图像数据，remap topic为点云输出话题数据；

frame_id 为/camera， remap topic为/rgbdslam/online_clouds ，运行：roslaunch rgbdslam octomaptransform.launch，产生以下报错：

`[ WARN] [1619975632.888582540]: MessageFilter [target=/camera ]: Dropped 100.00% of messages so far.`

frame_id 应该是一个tf关系，这里查看tf关系如下所示：这是运行rgbdslam代码的关系

![5](/home/gipsy/Desktop/八叉树/IMG/5.png)

这是只运行bag的关系：

![6](/home/gipsy/Desktop/八叉树/IMG/6.png)

这里修改为kinect之后，产生了一个新的报错：

`Warning:` TF_OLD_DATA ignoring data from the past for frame kinect at time 1.30503e+09 according to authority unknown_publisher`
`Possible reasons are listed at http://wiki.ros.org/tf/Errors%20explained`
         at line 277 in /tmp/binarydeb/ros-melodic-tf2-0.6.5/src/buffer_core.cpp`

现在的可能原因：1、我这个地图不是点云地图，无法转换  2、frame_id还是有问题

kinect下是有数据的，但是rviz报错这个数据类型不对：

![7](/home/gipsy/Desktop/八叉树/IMG/7.png)

![8](/home/gipsy/Desktop/八叉树/IMG/8.png)

这哭还有一个问题，可能是这个点云是彩色的稠密点云rgb格式的，一般处理的点云是白色的点云

节点终端中：[ WARN] [1620024644.375118714]: Nothing to publish, octree is empty

rviz中：Wrong octomap type. Use a different display type.

类似解决方法：https://github.com/ivipsourcecode/DS-SLAM/issues/11

尝试解决方法：

sudo apt-get remove ros-melodic-octomap-server 

重新编译工作空间

这玩意不能卸载，卸了运行不了节点

出去和同学吃了个饭，代码跑通了哈哈哈，应该是重新卸载，再下载octomap-server 这个包的原因。不应该是我应该选occupancygrid节点；

![9](/home/gipsy/Desktop/八叉树/IMG/9.png)

运行成功的状态：

终端：[ WARN] [1620049156.759013305]: Nothing to publish, octree is empty

rviz：如上图所示。

## 参考资料

1、

## 总结

**点云地图**：

- 点云地图无法处理运动物体。因为我们的做法里只有“添加点”，而没有“当点消失时把它移除”的做法；
- 点云地图通常规模很大，所以pcd文件也会很大；
- 是否能直接用于导航：

**八叉树地图**：：

- 总述：八叉树地是一种灵活的、压缩的、又能随时更新的地图形式；

- 八叉树地图可以随时更新；
- 是否能直接用于导航：

**栅格地图**：

- 是否能直接用于导航：是



ROS：

- 你可以使用rostopic echo 来检查是否有数据输出

## 疑问

终端中如何用快捷键跳到行首和行末？

如何用快捷键全选

如何用快捷键删除当前内容；

这里还有一个问题，；launch文件更新之后到底要不要重新编译？


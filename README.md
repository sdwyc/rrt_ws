# SMP中文说明

#### 
这是一个工作空间，clone到本地后，直接在工作空间内运行命令
```bash
catkin_make
```

## 个别功能包说明
sim_environment_arranger 仿真环境

scout_simulator scout2机器人模型（可换成husky）

pose_publisher 直接从gazebo中获取准确的位姿（没有用到现在的系统里）

local_planner MPC局部规划器，输入waypoint，输出 local_path

path_follower 根据路径产生waypoint，（没有用到现在的系统里）

elevation_completion 高程值补全（没有用）

## 运行
首先source工作空间，启动仿真环境
```bash
roslaunch sim_environment_arranger launch_sim.launch world_name:=uneven1
```
其中，world_name为仿真环境的地皮模型，可填的模型：uneven0～5，normal

将机器人移动一段距离，待elevation_map形成一定范围后，运行SMP planner
```bash
roslaunch rrt_star_mesh run.launch
```
注意：节点运行后，直接在rviz里选择目标点，但是只能选择一次，多次选择，节点会掉（此问题是由于rrt树生长线程会崩，未解决）

注意：现在rrt节点已经可以发布subgoal和waypoint，在某些地皮环境下会失败，如uneven1

## 说明
src/rrt_star_mesh/include文件夹下带～的文件是未用到的文件

现在存在的问题：

1. 无法重规划，多次规划节点会died，树生长线程容易挂，多次规划需要重启节点

2. subgoal有时选取不可信，机器人移动过程中，grid_map里有空洞，这些空洞是有数值的，数值为NAN，很多叶子节点在空洞前就已经停止生长了，现在判断叶子是否位于边界的方法是取以叶子节点为中心的正方形submap，如果取出来宽高相同说明叶子位于map里面，如果宽高不相同说明叶子在边界上。

3. 寻线功能包不稳定，机器人有时尾部朝前移动，甚至是在小山丘附近摇摆，这是由于没有terrian_analysis的原因，目前的寻线功能包是从cmu团队开发的TAREplanner里改编的，他们有用terrian_analysis，但是不好融入到现有的仿真环境里。
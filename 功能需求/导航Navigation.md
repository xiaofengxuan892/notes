[TOC]



##### NavMeshAgent：

当使用该组件用于导航时，会自动从烘培后的场景中查找“路径最短”的一条。其包含以下参数：

1).stoppingDistance：代表停止导航的距离。当距离目标位置的距离小于stoppingDistance，则结束导航

2).agentRadius：角色可以距离墙壁，窗户或其他物体的最近距离

3).agentHeight:角色可以通过的最低高度

4).stepHeight:角色可以跨过的最高高度——台阶

5).dropHeight:从上往下跳的最大高度

6).jump distance:可以跳出的最远距离

7).min region area指agent可以在其上寻路的最小区域，若区域太小则会被直接剔除

8).remainingDistance：距离目标位置剩余的距离

**注意**：

1).navgation面板中Advanced 中的height mesh一般不需要勾选，否则会占用更多的资源，延长烘培时间

2).当修改导航的任何参数后，都需要重新Bake烘培得到新的地图后才能得到新的导航路径

###### agent.Resume() 和 agent.enabled 的区别：

agent.resume()是恢复之前的导航状态，接着之前的情况继续导航

**agent.enabled = false 后再重新设置** **agent.enabled = true，实际表现并不会继续之前导航，而是重新开始**

PS: agent.enabled=false 在false之后，以前的信息都会被丢失



##### `Off Mesh Link`组件的使用：

该组件专用于设置两个点之间的跳越。在寻路中存在从A-》B，但A，B之间有白色的不可直接通过的白色区域，且该区域通过navgation面板中的drop height, Jump distance等属性都无法达到，此时则需要借助OffMeshLink在A，B之间架接桥梁，使得A，B之间无论隔的多远都可以直接达到

1).该组件中的“cost override”表示“跳过这个的代价”，在导航时会自动比较各个路径的代价，选择“最低代价”的路进行导航

2).NavMeshAgent中包含参数“autoTraverseOffMeshLink”，当其数值为true时，则角色到达A点后会自动跳到B点；如果为false，则到达A点后，无法自动跳越。针对部分需要展示“一步步爬楼梯”过程的情况，则可以设置该参数为false



##### 导航中遇到的问题：

###### 1.对于在攻击状态下仍然可能存在的敌我之间有墙壁遮挡，但已然是最小的攻击距离

**解决方案**：先设置较小的攻击距离，另外在烘培时把障碍物墙壁等遮挡的，设置其下方的可通过区域很小，这样敌人能接近的地方就很少了。如此使敌我双方尽量没有墙壁遮挡



###### 2.当大量怪物导航到目标地点，出现“扎堆聚集、模型穿透”的情况

**解决方案**：引入“Nav Mesh Obstacle”组件，和“Nav Mesh Agent”组件挂载在同一个物体上。在导航时设置“navMeshAgent = true，navMeshObstacle = false”；当到达目标地点后，则设置“navMeshAgent = false, navMeshObstacle = true”，如此即可实现“挖坑”的效果，避免“扎堆聚集”

**PS**：1).在“nav mesh obstacle” 组件中“carve"一定要勾选，这个才是真正的挖坑实现

2).两个组件不可同时启用，同一时间只能使用其中一个

```c#
void Update ()
{
     if (Input.GetMouseButtonDown (0)) {
          Ray ray = Camera.main.ScreenPointToRay (Input.mousePosition);
          RaycastHit hit;
          if (Physics.Raycast(ray, out hit)) {
               agent.enabled = true;
               obstacle.enabled = false;
               agent.SetDestination (hit.point);  /*设置目标位置的方法，记住：不是直接的赋值 ： agent.destination =hit.point*/
               isStart = true;
          }
      }

      if (isStart) {
          if (agent.remainingDistance < 2.5f && agent.remainingDistance > 0.5f && agent.enabled){          
              /*agent.remainingDistance这个方法很重要，另外这里直接使用agent.enabled来作为条件标志位帮助判断*/
               agent.Stop(true);      /*这个代表停止巡逻*/
               agent.enabled = false;
               obstacle.enabled = true;
               isStart = false;
          }
      }
}
```










































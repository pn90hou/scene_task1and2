---

## SceneNavigation


简介： 带有 VR 摄像头的 3D 场景。

特殊Notes: 

由于之前使用的教程内容不再适用，现有两个办法，fork的同学可以选择更新并merge，如果git操作不够熟练的，可以保持原工程，但需拖入SteamVR->InteractionSystem->Core->Prefabs->Player.prefab，并删除场景中的[CameraRig]，如果喜欢之前Camera的特殊效果，找到[CameraRig]->Camera(Head)->Camera(eye)，将其上绑定的Image Effect（Fog,DepthField，Sunshaft等等）绑定到Player->SteamVRObjects->VRCamera。

对于Teleport Area，要求自制floor，可以选择GameObject->3D Objects->Plane，创建一个Plane，将高度y设为0.5，将scale（缩放）设为25，1，25，并将Teleport Area绑定其上即可当做自由传送区域使用。
Teleport Point要求特定传送点，Teleport Area则是一整个范围内都可以传送，可以自行选择，并作出有意思的漫游App。

当在“混合现实门户”中的头盔模拟器中调试时，右键点击Left Controller和Right Controller下的Touchpad圆盘，将会显示Teleport传送线，保持右键按住并向上滑动后松开可以传送至想到达的区域。

调试难度： ★

基于提供的场景，完成以下tasks：

1. 根据[教程](https://unity3d.college/2017/07/18/create-a-vr-teleport-system-in-your-unity-game-with-the-steamvr-interaction-system/)， 完成 *VR Teleport* （难度： ★★）
2. 自学并尝试使用 Unity Terrain 制作工具，绘制地形并种植植被，并在新构建的场景上完成 VR 漫游 （难度： ★★★）
3. 在场景里加入更多其他3D物件，并增加更多交互 （如VR射箭） （难度： ★★★★）

![SceneNavigation](https://bitbucket.org/blueprintrealityinc/vr_demo_instruction/raw/master/demo2.png "快来愉快地开始你的第一个VR App吧")

---

Acknowledge

https://assetstore.unity.com/packages/templates/systems/steamvr-plugin-32647

https://assetstore.unity.com/packages/3d/environments/nature-starter-kit-2-52977
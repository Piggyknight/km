## ScreenShot重构

### 需求整理

1. ScreenShot组件

   ​	支持内容选择

   1. 1. ui
      2. 不带UI
      3. gui

   2. 几种获得的方式

      1. 当前的
      2. 上一帧

   3. 支持editor版本检验场景截屏时是否有效

      1. 截取场景时当前是否存在场景与角色
      2. 截取ui时当前是否存在ui

      

   PS: 原代码支持blur, 属于贴图处理不输于screenshot, 暂时不用支持



2. CameraVisibleData, CamearVisibleCtrl放入RenderApi组建内, 通过外部消息推动相机状态机

   1. 3个参数, CMD类型: DisableRendering,  关闭的类型: 3d/ui, 申请者名字
   2. 关闭3d相机

   

3. (可选)相机配置的竞争机制(token加队列)

   1. DialogueCameraBridge
      1. 获取ui信息, 
      2. 获取token, 产生是否关闭相机
   2. screenshotBridge
      1. 获取从screen shot来的消息
      2. 获取token, 对相机进行操作

   

4. 单独的抽象逻辑相机状态位(和unity无关)

   1. 只开启3d
   2. 只开启2d
   3. 全开
   4. 全关闭



### 物理设计

1. 截图使用主相机, 因为后处理, 雾效暂时绑定主相机, 因此必须保存主相机之前状态,  截完图后再恢复

2. 内存中会保留一张截图, 外部需求截图时, 如果参数不付和, 则需要重新截图, 如果符合则直接使用

3. 外部获取只能通过异步方式call back方式获取相应贴图的句柄, 使用时通过句柄访问获得贴图, 并建立双向链接, 如果内部截图销毁, 则需要进行断链

4. 增加新的命令, 相机在关闭时, 会默认将最后一帧显示内容作为截图进行保存

![image-20201020153826503](image/SnapShop%E7%9A%84%E9%87%8D%E6%9E%84/image-20201020153826503.png)





### code review

350行代码

主要类

- SnapShotUtil

  - 主入口

  

- BlurTexture

  - 

- SnapShotUtilSub
  
- 包含一个
  
- SnapShotCameraComp
  
  - monobehaivour, onrenderimage来截图



和RenderApi中相机关系

- 通过目前的命令机制, 控制相机相机关闭消息



和CameraVisibleCtrl/CameraVisibleData的关系

	- 去除CameraVisibleCtrl, 相机关闭属于相机内部状态
	- 移除CameraVisibleData中的disable数据



和BookPageScene的关系
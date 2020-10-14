## Talk 系统

1.  从对话的数据来源分

   1. 行为树

   2. 策划的表格

2.  从类型分：
   1. 主对话, 主角参与的,有一定交互
   2. npc 自动对话,一般是头顶冒字,侧板栏,聊天,角色动画
   3. npc 对话,一般是头顶冒字,角色动画



  现在碰到的问题

1. 叶子节点不干净

   ​	* 角色的头顶冒字

   ​	* 相机节点

​	

2. 一些回调(ui点击,timer 等)会立即直接推动状态机变化

   ​	比如:分支选择,点击之后,直接播放下一个对话, 但是整个对话系统,只能有一个主对话在运行

   ​	要保证栈底是干净的.	

   ​	收到click 消息,发送到消息队列

   



talk的表现

	- npc的表现(面朝, 动画)
	- 相机表现
	- 近距离触发3d ui button





2. 支持根据性格筛选不同的对话选项





### Code Review

TalkClipFactoryWrap

 - 内部有所有TalkClip的子工厂, 根据ResDialogue中配置的类型找到对应的clip子工厂, 创建放入队列返回
 - Talk_BtFactory._create中会创建

TalkMsgHandler.PlayGuideTalk => TalkMsgHandler._play_talk_id => TalkEntryFactory.CreateSpecifyTalk => TalkEntryFactory._create_talk => TalkClipFactoryWrap.CreateClip

 

TalkClipFactory_QuestBranch

- 任务分支对话的工厂类
- TalkUIBehavior_Dialog 放入clip的behavior_comps, 

- ChatSender(发送聊天频道), sound, actor_display, gyro_camera都放在dialog中



​	- TalkClip_QuestBranch : I_TalkClip

TalkClipEntry

	- talkclip创建完后放入这里
	- 



DialogTalkMediator

​	- 接受消息, 启动关闭ui相关功能, 包括表情等



NpcBt

- 行为树的主入口



QuestBt

- 任务行为树的主入口



TalkBranchApi

	- 分支选项api
	- 灵宝分支与普通分支





TalkMsgHandler

- C_CMD_TALK
  - 没有状态防护, 纯被动相应外部事件, 
- 



BranchSelectEntityAction

	- 分支选择时的触发, 会发送网络消息,同时触发CMD_Talk, 没有状态保护


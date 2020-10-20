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



### 对话需求

- 对话类型决定对话的表现

  - **3D对话**
  - 底边对话
  - 电影字幕
  - boss对话(被用作公告, 目前dialog_type=6的公告类型为对话id非对话组id)
  - 引导对话
  - Q版侧边栏对话
  - **任务分支选择**
  - **灵宝分支选择**
  - 头顶气泡对话
  - 友好度气泡

  PS: 标黑的3中类型都是3d对话的表现, 在分类上原有代码不是很严谨



- 同一类对话类型dialog_type触发的对话队列播放，各类插入式表现（Cut Scene、loading切场景）不会打断对话，对话延迟至插入式表现结束后继续播放
  - 灵溪镇和王路3d 对话触发漫画cutscene, 结束后继续3d对话

- 不同类对话类型触发的对话互不干扰，正常播放
- 所有的对话类型都需要能应用打字机效果need_typer且文字正常显示
- 所有的对话类型都需要能应用显示时长duration，时长达到，对话自动结束
- 说话人姓名speaker_name改为int，支持富文本格式

- 对话触发源
  - 点击NPC，触发默认对话
  - 场景里面的角色，触发的 头顶气泡/侧边栏的 对话
  - 任务入口 进入的对话
  - 3渲2界面里面的交互对话
  - 服务器下发的对话（帮花)

- 提供策划在对话树(Behaivc Tree )中配置的功能:
  - 显示对话/分支剧情
  - 添加对话的选项
  - 获得用户交互的结果
    - 分支选项的结果
    - 按钮点击的事件
    - 某些特殊UI关闭的事件
  - 执行NPC Func
  - 弹出指定的UI Dialogue组件
  - 获得当前游戏的某些状态
    - 个人赛季的步骤
    - 公会寇岛的状态
    - 队友数量
    - 是否加入公会
    - 是否是队长
    - 心域卡牌



#### bug汇总

| 6731 | BUG  | z任务/主线 | 01-必须解决 | 1    | 系统 | 结构（需要分析结论） | 2    | 对话：【对话】分支后对话配置失效，在游戏内不播放             |
| ---- | ---- | ---------- | ----------- | ---- | ---- | -------------------- | ---- | ------------------------------------------------------------ |
| 6811 | BUG  | NPC        | 01-必须解决 | 1    | 系统 | 结构（需要分析结论） | 2    | 对话：【NPC】与NPC对话，报错“[Talk]正在talking 对话ID”，导致部分任务无法通过 |
| 6813 | BUG  | j基础功能  | 01-必须解决 | 1    | 系统 | 结构（需要分析结论） | 2    | 对话：dialog_type为2的对话在播的时候不能进行下一步任务       |
| 6827 | BUG  | NPC        | 01-必须解决 | 1    | 系统 | 结构（需要分析结论） | 2    | 对话：【NPC】【实际为对话功能报错】部分情况下，无法通过NPC接取环任务 |
| 6843 | BUG  | 0_基础功能 | 01-必须解决 | 1    | 系统 | 需要分析             | 2    | 对话： 物件本地事件-分支选择不会触发，游戏会卡死             |



### Code Review

TalkClipFactoryWrap

 - 内部有所有TalkClip的子工厂, 根据ResDialogue中配置的类型找到对应的clip子工厂, 创建放入队列返回
 - Talk_BtFactory._create中会创建

TalkMsgHandler.PlayGuideTalk => TalkMsgHandler._play_talk_id => TalkEntryFactory.CreateSpecifyTalk => TalkEntryFactory._create_talk => TalkClipFactoryWrap.CreateClip

 

TalkClipFactory_QuestBranch

- 任务分支对话的工厂类
- TalkUIBehavior_Dialog 放入clip的behavior_comps, 

- ChatSender(发送聊天频道), sound, actor_display, gyro_camera都放在dialog中



TalkClip_QuestBranch : I_TalkClip



TalkClipEntry

​	- talkclip创建完后放入这里 



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



talk的类型

```c#
public enum E_TalkType
{
    none = 0,//非法值
    dialogue_norm = 1,   //普通对话
    dialogue_bubble = 2,   //右侧小气泡
    dialogue_subtitle = 3,  //电影字幕
    dialogue_3_d = 4,       //3d对话
    dialogue_boss1 = 5,
    dialogue_boss2 = 6,
    dialogue_boss3 = 7,
    dialogue_guide = 8, //引导对话
    dialogue_quest = 9, //分支任务
    dialogue_pet_choose = 10, //灵宝选择
    dialogue_npc_bubble = 100, // npc头顶气泡
    dialogue_no_dialog = 101,//没有任何形式的弹窗，用来往聊天框发消息
    dialogue_friendliness_bubble = 200, //友好度气泡
}
```



任务相关的对话都在任务的excel中



ResDialogue

ResQuestFork



- behaivc tree中 => ShowBranchSelect => SMAgent.ShowBranchSelect => TalkBranchApi.ShowBranch=>BranchSelectImplDialog.on_start
  - 发送C_CMD_NPC_CLICK消息
  - TalkBranchOptionFactory.Create创建button
  - 不管是否成功, 直接执行后续队列, 创建相应的分支选择btn, 并发送消息添加C_CMD_TALK_DIALOG_ADD_BTN



- 无意义的action
  - BranchSelectNpcAction
  - BranchSelectEntityAction


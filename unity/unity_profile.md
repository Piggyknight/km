## Profile Unity 的工具汇总

[TOC]



#### 1. Unity Profile

 - Editor模式下的注意事项:

   	- 不同窗口的内容可能会重复渲染相同的内容
      	- 编辑器渲染本身的开销
   	- 所有Mesh在run模式下都会开启 read/write Enabled标记, 会导致mesh memory翻倍
   	- vertex compression会被关闭
   	- 一些Api会调用gc, 比如GetComponent如果为空也会产生gc(为了new exception扔出来)

   

- Deep Profiling模式

   - 2017.3 版本之后, Deep Profile模式能够支持mono scripting下android与desktop的

   - 2019.3版本之后, Allocation Call Stacks支持全平台, 并且Deep Profile支持全平台包括il2cpp

   - **Desktop** 模式在启动命令行后添加  *-deepprofiling*

   -  **Android**上使用如下adb命令:

     > $ adb shell am start -n com.company.game/com.unity3d.player.UnityPlayerActivity -e 'unity' '-deepprofiling'*





#### 2. Android Profile

##### 2.1. Snapdragon Profiler

##### 2.2 Simperf





#### 3. PC Profile

##### 3.1 VTune的使用

VTune是intel发布的pc上的性能优化套件, 包含较多功能. 使用

1. 配置unity导出symbol文件, 如下图

   ![img](image/unity_profile/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16007658984716.png)

2. 将pdb放到exe相同目录, vtune会自动找到, 详情参考[Vtune参考网址](https://software.intel.com/content/www/us/en/develop/documentation/advisor-user-guide/top/configuration/project-properties-dialog-box/binary-symbol-search-and-source-search-locations.html)






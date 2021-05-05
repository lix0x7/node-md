# Ref


- ZGC 原理是什么，它为什么能做到低延时？ - RednaxelaFX的回答 - 知乎 [https://www.zhihu.com/question/287945354/answer/458761494](https://www.zhihu.com/question/287945354/answer/458761494)
- ZGC原理与实现分析  [https://www.jianshu.com/p/4e4fd0dd5d25](https://www.jianshu.com/p/4e4fd0dd5d25)
- ZGC原始论文
    [![](https://cdn.nlark.com/yuque/0/2019/png/657413/1576207504554-2c267985-015b-4612-9731-b67aa292450f.png#align=left&display=inline&height=60&margin=%5Bobject%20Object%5D&originHeight=60&originWidth=300&status=done&style=none&width=300)](https://cdn.nlark.com/yuque/0/2019/pdf/657413/1576207504508-73399405-e3af-4a6d-92c3-228148a742f4.pdf)
- Jfokus 2018 演讲，视频及 PPT： [https://www.youtube.com/watch?v=tShc0dyFtgw](https://www.youtube.com/watch?v=tShc0dyFtgw)
    [![](https://cdn.nlark.com/yuque/0/2019/png/657413/1576207504615-75228d6e-b971-4615-be98-c77e75224132.png#align=left&display=inline&height=60&margin=%5Bobject%20Object%5D&originHeight=60&originWidth=300&status=done&style=none&width=300)](https://cdn.nlark.com/yuque/0/2019/pdf/657413/1576207504578-741c9f6d-0c28-4df2-af48-893ed4f7490e.pdf)


# 特点


## 染色指针


如下图，ZGC 使用了 64 位指针中的四位用作染色（记录对象的 GC 状态）。


- Marked：是否标记存活。Marked 标记有两个，是因为 ZGC 的相邻的 GC 循环之间有重叠（详情见下文），相邻的两个 GC 循环会轮流使用 Marked 0/1。
- Remapped：是否已经重映射过了，如果未完成重映射，则读取时需要从 Relocation Set 读取引用的映射信息
- Finalizable：是否需要触发 finalize 方法


![](https://cdn.nlark.com/yuque/0/2019/png/657413/1576207504632-27f9be3a-44da-45b0-b7ec-8f0e78558e6e.png#align=left&display=inline&height=198&margin=%5Bobject%20Object%5D&originHeight=198&originWidth=664&status=done&style=none&width=664)


## 加载屏障


在**加载对象引用**前，GC 通过 mask 操作判断当前对象所需要进行的处理，如果当前引用对象需要进行 GC 中的某个流程，会执行一段 GC 相关的操作，如果不需要特殊操作，则直接返回对象引用。流程图如下：


![](https://cdn.nlark.com/yuque/0/2019/jpg/657413/1576207504650-9484b314-f19d-480d-b661-6a3eb25959aa.jpg#align=left&display=inline&height=540&margin=%5Bobject%20Object%5D&originHeight=768&originWidth=1024&status=done&style=none&width=720)




## 低停顿，堆大小无关


ZGC 的停顿时间与堆大小无关，只和 GC Roots 中对象数量有关（和线程数量相关）。最大停顿时间 10ms，平均停顿时间 1ms。


## 单代


ZGC 不分代，整个堆统一管理。但这样其实不好，不能很好的区分冷热数据。


## Page Allocation


将堆分为 2M（small）, 32M（medium）, n*2M（large）三种大小的页面（Page）来管理，根据对象的大小来判断在那种页面分配


# ZGC 三阶段


1. 标记 Mark
    1.1 【**暂停**】初始标记 GC Roots


        ![](https://cdn.nlark.com/yuque/0/2019/png/657413/1576207504677-49071442-8583-49c2-94f9-a3f61dfa1d15.png#align=left&display=inline&height=251&margin=%5Bobject%20Object%5D&originHeight=309&originWidth=818&status=done&style=none&width=664)


**    **1.2 【并发】并发标记，遍历对象图
        ![](https://cdn.nlark.com/yuque/0/2019/png/657413/1576207504697-fff85d8b-3dc8-4dee-97a2-d1f667595c2e.png#align=left&display=inline&height=243&margin=%5Bobject%20Object%5D&originHeight=299&originWidth=813&status=done&style=none&width=662)


    1.3 【**暂停**】并发标记结束，处理边界情况，确认标记完成


2. 迁移 Relocate
    2.1 【并发】筛选需要回收的 Region（记录为 Relocation Set），即有垃圾的 Region
        ![](https://cdn.nlark.com/yuque/0/2019/png/657413/1576207504715-0bdf81bf-cd7a-4264-a322-397dc28df523.png#align=left&display=inline&height=295&margin=%5Bobject%20Object%5D&originHeight=361&originWidth=808&status=done&style=none&width=660)


    2.2 【并发】在可回收 Region 上构建映射表用于加载屏障（Load Barrier）进行引用映射
        ![](https://cdn.nlark.com/yuque/0/2019/png/657413/1576207504733-77e4653c-2015-4e9b-8364-8e5c56c615c1.png#align=left&display=inline&height=326&margin=%5Bobject%20Object%5D&originHeight=401&originWidth=811&status=done&style=none&width=659)


    2.3 【**暂停**】初始 Relocate，将 GC Roots 上的对象进行 Relocate，更新其所在 Region 的映射表
        ![](https://cdn.nlark.com/yuque/0/2019/png/657413/1576207504751-bbcf01b9-b204-4c43-a076-b9b621a7e824.png#align=left&display=inline&height=298&margin=%5Bobject%20Object%5D&originHeight=371&originWidth=811&status=done&style=none&width=651)


    2.4 【并发】并发 Relocate，遍历对象图；用户进程此时也可以通过读取对象引用的方式触发加载屏障完成 Relocate
        ![](https://cdn.nlark.com/yuque/0/2019/png/657413/1576207504772-573bfcfc-1449-4a0e-a53a-52b92e94eac8.png#align=left&display=inline&height=297&margin=%5Bobject%20Object%5D&originHeight=371&originWidth=814&status=done&style=none&width=652)
        ![](https://cdn.nlark.com/yuque/0/2019/png/657413/1576207504795-b8f29168-1aa9-45a5-9158-2dc2469aaecf.png#align=left&display=inline&height=289&margin=%5Bobject%20Object%5D&originHeight=362&originWidth=812&status=done&style=none&width=649)
        ![](https://cdn.nlark.com/yuque/0/2019/png/657413/1576207504818-7ce4af40-0090-4d96-8af7-21952c4808a9.png#align=left&display=inline&height=298&margin=%5Bobject%20Object%5D&originHeight=373&originWidth=806&status=done&style=none&width=645)
        ![](https://cdn.nlark.com/yuque/0/2019/png/657413/1576207504843-28b78733-4657-400e-a9e1-7a5e97d92601.png#align=left&display=inline&height=285&margin=%5Bobject%20Object%5D&originHeight=360&originWidth=811&status=done&style=none&width=642)




3. 重映射 Remap / 下一轮 Mark【也是下一轮 GC 的第一阶段】
    3.x 修正前一轮 GC 中的映射关系。在 Mark 过程中，如果发现需要经过映射表的映射，则将其修正为直接的对象引用。最终清空映射表。
        
        ![](https://cdn.nlark.com/yuque/0/2019/png/657413/1576207504873-fc15d588-7d51-4190-ad4c-7f3e6b57656d.png#align=left&display=inline&height=284&margin=%5Bobject%20Object%5D&originHeight=360&originWidth=817&status=done&style=none&width=645)



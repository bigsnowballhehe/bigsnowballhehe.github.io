---
title: Vue父子组件生命周期加载顺序/mutation和action
date: 2021-05-26 18:37:56
tags:
 -Vue
---
今天面试被问到一个问题,之前都没遇见过,问父子组件生命周期加载顺序,把我整懵了. 
## 答案 
**1.加载渲染过程**  
父beforeCreate->父created->父beforeMount->子beforeCreate->子created->子beforeMount- >子mounted->父mounted

**2.子组件更新过程**  
父beforeUpdate->子beforeUpdate->子updated->父updated

**3.父组件更新过程**  
父beforeUpdate->父updated

**4.销毁过程**  
父beforeDestroy->子beforeDestroy->子destroyed->父destroyed

## 理解  
组件的调用顺序都是先父后子,渲染完成的顺序肯定是先子后父

组件的销毁操作是先父后子，销毁完成的顺序是先子后父
## 原理
* 当dom渲染时，会createElm创建元素，创建元素后会进行初始化，初始化组件的时候内部还有组件，会不停的去渲染，所以它的渲染顺序是先父后子，完成的顺序是先子后父。

* dom渲染描述：先父组件要创建beforeCreate、created，父组件实例化完成后要挂载这个父组件beforeMount，挂载父组件的时候会调用父的render方法，渲染的时候发现里面有子组件，这时就会调用子组件的beforeCreate、created、beforeMount，当子组件都完成之后，会把子组件先存起来，这儿有队列，不是子完成就会调用子的mounted，因为子组件中可能还有子组件，这时会暂存一下，到最后子全完成了会按照子调父的调，mounted是先子后父。

* 源码：在dom渲染和更新时就会调用patch方法，在patch方法中有一个insertedVnodeQueue数组，会将所有的vnode存放在insertedVnodeQueue中，最后会在整个创建完之后会调用invokeInsertHook方法，依次调用收集的insert hook。在patch中会调用createElm方法来创建元素，createElm方法中会对元素进行判断：如果元素是组件，会调用createComponent方法创建组件，调用组件的init方法渲染当前组件的内容，通过initComponent方法将pendingInsert插入到自己的insertedVnodeQueue中，然后将它置为null，最后会调用invokeCreateHooks方法将vnode存放在insertedVnodeQueue中；如果元素不是组件，会调用createChildren方法递归遍历子节点（子组件），createChildren方法中会再次调用createElm方法，直到元素不是一个组件时。

* 当所有组件都完成后，会调用invokeInsertHook方法，里面会循环通过insert方法依次调用收集的hook，在insert方法中就会触发mounted。
![流程](https://img-blog.csdnimg.cn/20200903160917803.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcyMDg2,size_16,color_FFFFFF,t_70)

[原文链接](https://blog.csdn.net/qq_42072086/article/details/108385632)
### 为什么有mutation还要有action
* Mutation 必须是同步函数一条重要的原则就是要记住 

现在想象，我们正在 debug 一个 app 并且观察 devtool 中的 mutation 日志。每一条 mutation 被记录，devtools 都需要捕捉到前一状态和后一状态的快照。然而，在上面的例子中 mutation 中的异步函数中的回调让这不可能完成：因为当 mutation 触发的时候，回调函数还没有被调用，devtools 不知道什么时候回调函数实际上被调用——实质上任何在回调函数中进行的状态的改变都是不可追踪的。
今天被问得有点奇怪,说为什么有了action还要mutation...事实上action只是当一个延后提交mutation的方法,真正改数据的还是mutation.也就不存在,两者重复一说.
### 微前端简介
    1.将不同功能按照不同维度拆分成多个子应用，通过主应用来加载这些子应用
    2.解决的问题：1).技术栈不做限制，不同团队协同开发2).子应用可以独立部署3).在已有项目中重构，依然可以保留老的代码
### 微前端框架
    1.SingleSpa：2018年诞生，实现了路由劫持和应用加载问题，未解决父子或者平级应用之间css样式冲突以及父子或者兄弟应用同时使用window所产生的冲突问题
    2.qiankun：2019年诞生，基于single-spa实现了开箱即用的api(single-spa + sandbox + import-html-entry)，可以做到父子或者兄弟应用之间css隔离，js沙箱机制
    3.总结：子应用可以做到独立部署，技术栈无关，父子应用解耦，运行时动态加载，连接父子应用之间依靠的就是协议接入(子应用必须导出bootstrap unmount mount方法)
    4.iframe的问题：不能单独保存状态 eg，一个页面中嵌入多个iframe实现，每个iframe中都有自己的路由，当用户切换了其中一个iframe的路由，然后刷新了整个页面，那么iframe中的状态就丢掉了，所有的iframe都回到了最初的状态，包括刚刚切换了路由的那个iframe
    5.应用通信：1）基于URL传递，当传递消息很弱 2）基于浏览器提供的一些原生事件customEvent进行通信 3）基于props主子应用之间通信 4）使用全局变量，借助window对象，挂载一些属性进行同信，也可使用redux通信


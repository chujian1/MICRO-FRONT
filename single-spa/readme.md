### single-spa实战
    vue create xxx // 创建vue的父应用和子应用
### 缺陷
    1.css样式冲突问题: 1)子应用之间的样式冲突可以通过动态样式表，当应用切换时，移除老样式，添加新样式 2）父子应用样式冲突时，通过 BEM，约定项目前缀 or css-modules，打包生成不冲突的选择器名 or shadow DOM ，真正意义上的隔离
```
shadow dom
<body>
<div>
    <p>hello world</p>
    <div id='shadow'></div>
</div>
<script>
    // 创建一个shadow dom，closed 外界无法访问，open 外界可以访问
    const shadowDom = document.querySelector('#shadow').attachShadow({mode: 'closed'})
    const pEle = document.createElement('p')
    pEle.innerHTML = 'hello p'
    const styleEle = document.createElement('style')
    styleEle.textContent = 'p {color: red;}'
    shadowDom.appendChild(pEle)
    // 仅仅hello p 为红色
    // shadowDom.appendChild(styleEle)
    // 仅仅hello world 为红色
    document.body.appendChild(styleEle)
</script>
</body>
```
```
加载A应用，有window.a 切换到B应用，丢弃window.a，当再次切换到A应用，又恢复window.a属性
沙箱：创建一个干净的环境给子应用使用，当切换时，可以丢弃或者恢复属性
```
```
快照沙箱
// 快照沙箱：将change保存起来展示或者隐藏，只适合两个应用之间的切换
class Snapshot {
    constructor() {
    this.proxy = window
    this.modifyProps = {}
    this.active()
    }
    active() {
    this.currentProps = {}
    for(let prop in window) {
        if(window.hasOwnProperty(prop)) {
        this.currentProps[prop] = window[prop]
        }
    }
    Object.keys(this.modifyProps).forEach( prop => window[prop] = this.modifyProps[prop])
    }
    inactive() {
    for(let prop in window) {
        if(window.hasOwnProperty(prop)) {
        if(window[prop] !== this.currentProps[prop]) {
            this.modifyProps[prop] = window[prop]
            window[prop] = this.currentProps[prop]
        }
        }
    }
    }
}
const snapshot = new Snapshot();
// 立即执行函数前边的代码需要以;结尾，否则报错
(window => {
    snapshot.proxy.a = 'a'
    snapshot.proxy.b = 'b'
    console.log(snapshot.proxy.a, snapshot.proxy.b) // a b
    snapshot.inactive()
    console.log(snapshot.proxy.a, snapshot.proxy.b) // undefined undefined
    snapshot.active()
    console.log(snapshot.proxy.a, snapshot.proxy.b) // a b
})(snapshot.proxy)
```
```
// 多个应用的沙箱采用es6的proxy，可以创建多个proxy实现，不同的应用使用不同的代理实现
class SnapProxy {
    constructor() {
    const rawWindow = window
    const fakeWindow = {}
    const proxy = new Proxy(fakeWindow, {
        set(target, p, value) {
        target[p] = value
        return true
        },
        get(target, p) {
        return target[p] || rawWindow[p]
        }
    })
    this.proxy = proxy
    }
}
const box1 = new SnapProxy()
const box2 = new SnapProxy()
window.a = 1
console.log(window.a);
(window => {
    window.a = 'box1'
    console.log(window.a)
})(box1.proxy);
(window => {
    window.a = 'box2'
    console.log(window.a)
})(box2.proxy);
console.log(window.a)
```
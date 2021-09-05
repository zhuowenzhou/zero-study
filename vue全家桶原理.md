# vue全家桶

## vue-router

```javascript
let vue = null;
// hash
const getUrlPath = () => location.hash.slice(1) || '/';

class VueRouter {
    constructor(options) {
        const { routes } = options;
        this.routes = routes;
        // 监听浏览器url的#变化
        window.addEventListener('hashchange', this.hashchange.bind(this))
        // 监听页面加载完毕事件
        window.addEventListener('load', this.hashchange.bind(this))
        vue.util.defineReactive(this, 'matched', []);
    }

    hashchange() {
        // 获取当前路由
        this.current = getUrlPath();
        // 重置匹配路由
        this.matched = [];
        // 执行路由匹配
        this.match();
    }

    match(routes) {
        routes = routes || this.routes;
        for (const route of routes) {
            if (route.path === '/' && this.current === '/') {
                this.matched.push(route);
                return;
            }
            if (route.path !== '/' && this.current.includes(route.path)) {
                this.matched.push(route);
                if (route.children) {
                    this.match(route.children)
                }
            }

        }
    }

}

VueRouter.install = (_vue) => {
    vue = _vue;
    // 通过mixin的方式延迟router实例的挂载
    vue.mixin({
        beforeCreate() {
            const { router } = this.$options;
            //  只有根实例才有router
            if (router) {
                vue.prototype.$router = router;
            }

        }
    });
    // 跳转用全局组件
    vue.component('router-link', {
        props: {
            to: {
                type: String,
                require: true
            }
        },
        render(h) {
            return h('a', {
                attrs: {
                    href: '#' + this.to
                }
            }, this.$slots.default)
        }
    });

    // 视图占位符组件
    vue.component('router-view', {
        render(h) {
            // 标记当前的vnode是routerview组件
            this.$vnode.data.routerView = true;
            // 初始化深度
            let depth = 0;
            let parent = this.$parent;
            // 递归找出当前router-view实例在第几层
            while (parent) {
                // root实例没有$vnode
                const vnodeData = parent.$vnode && parent.$vnode.data;
                if (vnodeData && vnodeData.routerView) {
                    depth++;
                }
                parent = parent.$parent;
            }
            const { matched } = this.$router;
            // 根据深度找出对应的组件
            const route = matched[depth];
            if (!route) {
                return h('h1', '404')
            }
            return h(route.component);
        }
    });
}


export default VueRouter;

```

## vuex

```javascript
let vue = null;
class Store {
    constructor(options) {
        this._mutations = options.mutations;
        this._actions = options.actions;
        this.commit = this.commit.bind(this);
        this.dispatch = this.dispatch.bind(this);
        this._wrappedGetters = options.getters;
        this.getters = {};
        const computed = {};
        Object.keys(this._wrappedGetters).forEach(key => {
            computed[key] = () => this._wrappedGetters[key](this.state, this.getters);
            Object.defineProperty(this.getters, key, {
                get: () => {
                    return this._vm[key];
                },
                enumerable: true
            })
        });

        this._vm = new vue({
            data() {
                return {
                    $$state: options.state
                };
            },
            computed
        });
    }

    get state() {
        return this.vm._data.$$state;
    }

    set state(val) {
        console.error('state属性只读')
    }

    commit(type, payload) {
        const mutation = this._mutations[type];
        if (mutation) {
            mutation(this.state, payload)
        } else {
            console.error('无法从mutations找到', type);
        }
    }

    dispatch(type, payload) {
        const action = this._actions[type];
        if (action) {
            action(this, payload)
        } else {
            console.error('无法从actions找到', type);
        }
    }
}

const install = (_vue) => {
    vue = _vue;
    vue.mixin({
        beforeCreate() {
            const { store } = this.$options;
            //  只有根实例才有store
            if (store) {
                vue.prototype.$store = store;
            }

        }
    });
}

export default {
    Store,
    install
}
```


## vue

```javascript
// 劫持数据
function defineReactive(obj, key, val) {
    //如果val是object则递归
    observe(val);
    const dep = new Dep(key);
    // 创建关系
    val.__dep__ = dep;
    Object.defineProperty(obj, key, {
        get() {
            if (Dep.target) {
                console.log(key, '添加订阅者', Dep.target);
                dep.addDep(Dep.target);
            }
            return val;
        },

        set(v) {
            if (val === v) {
                return;
            }
            //如果val是object则递归
            observe(val);
            val = v;
            dep.notify();
        }
    })
}

/**
 * 把data里面的字段全部代理到this下，便于用户操作
 * @param {vm实例}} vm 
 */
function proxy(vm) {
    Object.keys(vm.$data).forEach((key) => {
        Object.defineProperty(vm, key, {
            get() {
                return vm.$data[key];
            },
            set(v) {
                vm.$data[key] = v;
            },
        });
    });
}

/**
 * 重写Array方法，用于实现通知
 */
const originPrototype = Array.prototype;
const arrayPrototype = Object.create(originPrototype);
['push',
    'pop',
    'shift',
    'unshift',
    'splice',
    'sort',
    'reverse'
].forEach(function (method) {
    arrayPrototype[method] = function () {
        console.log("重写", method)
        originPrototype[method].apply(this, arguments);
        this.__dep__.notify();
    }
})

// 递归对象劫持数据
function observe(obj) {
    if (typeof obj !== 'object' || obj === null) {
        return obj;
    }
    if (Array.isArray(obj)) {
        obj.__proto__ = arrayPrototype;
    } else {
        Object.keys(obj).forEach(key => {
            defineReactive(obj, key, obj[key]);
        })
    }

}
// 处理响应式数据
function set(obj, key, val) {
    defineReactive(obj, key, val)
}

// 框架文件
class ZVue {
    constructor(options) {
        const { el, data } = options;
        this.$el = el;
        this.$data = data;
        this.$methods = options.methods;
        // 数据响应式处理
        observe(this.$data);
        // 把data的字段代理到this当中，方便使用
        proxy(this)
        // 编译模版
        this.complie(document.querySelector(el));
    }

    // 编译模版
    complie(node) {
        Array.from(node.childNodes).forEach(node => {
            // element
            if (node.nodeType === 1) {
                this.complieElement(node);
                if (node.hasChildNodes) {
                    this.complie(node);
                }
            } else if (node.nodeType === 3) { //text
                if (this.isInter(node)) {
                    this.compileText(node)
                }
            }
        })
    }

    // 判断是否为插值表达式{{xxx}}
    isInter(node) {
        return node.nodeType === 3 && /\{\{(.*)\}\}/.test(node.textContent)
    }

    // 编译文本节点
    compileText(node) {
        this.update(node, RegExp.$1, 'text')
    }

    // 编译元素节点
    complieElement(node) {
        Array.from(node.attributes).forEach(attr => {
            const { name, value } = attr;
            if (name.startsWith('z-')) {
                const direct = name.substring(2);
                this['_' + direct] && this['_' + direct](node, value)
            }
            if (this.isEvent(name)) {
                const eventName = name.substring(1);
                this.addEventListener(node, eventName, value);
            }
        })

    }

    // v-text处理模块
    _text(node, exp) {
        this.update(node, exp, 'text')
    }

    // v-html处理模块
    _html(node, exp) {
        this.update(node, exp, 'html')
    }

    // v-model处理模块
    _model(node, exp) {
        this.update(node, exp, 'model');
        this.addEventListener(node, 'input', (event) => {
            this[exp] = event.currentTarget.value
        });
    }

    _show(node, exp) {
        this.update(node, exp, 'show')
    }

    _if(node, exp) {
        this.update(node, exp, 'if')
    }
    /**
     * 
     * @param {节点} node 
     * @param {表达式} exp 
     * @param {指令} directive 
     */
    update(node, exp, directive) {
        const fn = this[directive + 'Updater'];
        fn && fn.call(this, node, this[exp])

        // 更新函数
        new Watcher(this, exp, val => {
            fn && fn.call(this, node, val)
        })
    };

    // 文本指令更新
    textUpdater(node, val) {
        val = typeof val === 'object' ? JSON.stringify(val) : val
        node.textContent = val;
    }

    // html指令更新
    htmlUpdater(node, val) {
        node.innerHTML = val;
    }

    // model指令更新
    modelUpdater(node, val) {
        console.warn("modelUpdater")
        node.value = val;
    }

    // show指令更新
    showUpdater(node, val) {
        console.warn("showUpdater")
        if (val) {
            node.style.display = '';
        } else {
            node.style.display = 'none';
        }
    }

    // if指令更新
    ifUpdater(node, val, key) {
        console.warn("ifUpdater")
        if (!val) {
            node && node.parentNode && node.parentNode.removeChild(node)
        } else {
            // node.style.display = 'none';
        }
    }

    // 事件判断
    isEvent(attrName) {
        return attrName.startsWith('@');
    }

    /**
     * 添加事件
     * @param {监听的元素} node 
     * @param {事件名称} eventName 
     * @param {事件处理器，如果是字符串则调用methods里面的方式，否则当函数执行} handleName 
     */
    addEventListener(node, eventName, handleName) {
        const handleFunc = typeof handleName === 'string' ? this.$methods[handleName] : handleName;
        handleFunc && node.addEventListener(eventName, handleFunc.bind(this))
    }
}

/**
 * 发布者
 */
class Dep {
    constructor(key) {
        this.key = key;
        this.watchers = [];
    }

    addDep(dep) {
        this.watchers.push(dep)
    }

    notify() {
        this.watchers.forEach(watcher => watcher.update());
    }
}

/**
 * 订阅者
 */
class Watcher {
    constructor(vm, key, updateFn) {
        this.vm = vm;
        this.key = key;
        Dep.target = this;
        // 通过触发属性获取的方式，在defineProperty的时候进行依赖收集
        this.vm[key]
        // 收集依赖结束后及时清空
        Dep.target = null;
        this.updateFn = updateFn;
    }

    update() {
        this.updateFn.call(this.vm, this.vm[this.key])
    }
}



```
```HTML
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <div id="app">
        <p>{{counter}}</p>
        <p>{{counter}}</p>
        <p z-text="text"></p>
        <p z-html="desc"></p>
        <p @click="onclick">onclick</p>
        <input type="text" z-model="text">
        <p z-show="show1">z-show true</p>
        <p z-show="show2">z-show false</p>
        <p @click="switchShow">switch show</p>

        <p z-if="if1">z-if true</p>
        <p z-if="if2">z-if false</p>
        <p>{{items}}</p>
    </div>

    <!-- <script src="../node_modules/vue/dist/vue.js"></script> -->
    <script src="./zvue.js"></script>
    <script>
        var app = new ZVue({
            el: '#app',
            data: {
                counter: 1,
                text: '我是一个text',
                desc: '<p>mini vue<span style="color:red">666</span></p>',
                show1: true,
                show2: false,
                if1: true,
                if2: false,
                items: [1, 2, 3]
            },
            methods: {
                onclick() {
                    console.log(this);
                },
                switchShow() {
                    this.show1 = !this.show1;
                    this.show2 = !this.show2;
                }
            },
        })

        setInterval(() => {
            app.counter++
            app.text = new Date().toLocaleString();
            app.items.push(1);
        }, 1000);

    </script>
</body>

</html>

```
# Vue全家桶&原理

## 资源

1、[vue-router](https://router.vuejs.org/zh/guide/)

2、[vuex](https://vuex.vuejs.org/zh/guide/)

3、[vue-router源码](https://github.com/vuejs/vue-router)

4、[vuex源码](https://github.com/vuejs/vuex)

## 知识点

### vue-router

Vue-router是[Vue.js](http://cn.vuejs.org/)官方的路由管理器。它和 Vue.js 的核心深度集成，让构建单页面应用变得易如反

掌。

**安装**： ``vue add router``

**核心步骤**

- 步骤一：使用``vue-router``插件，``router.js``

  ```javascript
  import VueRouter from "vue-router";
  
  Vue.use(VueRouter);
  ```

- 步骤二：创建``Router``实例, ``router.js``

  ```javascript
  const router = new VueRouter({
    routes
  });
  
  export default router;
  ```

- 步骤三：在根组件上添加实例

  ```javascript
  import router from "./vrouter";
  
  new Vue({
    router,
  }).$mount("#app");
  ```

- 步骤四：添加路由视图，``App.vue``

  ```html
  <router-view />
  ```

- 导航

  ```html
  <div id="nav">
  	<router-link to="/">Home</router-link> |
  	<router-link to="/about">About</router-link>
  </div>
  ```

  ```javascript
  this.$router.push('/')
  this.$router.push('/about')
  ```

### vue-router源码实现

单页面应用程序，URL发生变化时，页面不能刷新，显示对应视图内容

**需求分析**

- spa页面不能刷新
  - hash #/about
  - History api /about

- 根据url显示对应的内容
  - router-view
  - 数据响应式：current变量持有当前URL地址，一旦发生变化，动态重新执行router-view的render方法

**任务**

- 实现一个插件

  - 实现VueRouter类

    - 处理路由选项

    - 监控URL变化，hashchange

    - 响应这个变化

  - 实现install方法
    - $router 全局注册
    - 两个全局组件

**实现一个插件：创建VueRouter类和install方法**



``VueRouter``基本结构，创建``vue-router.js``

```javascript
// 保存Vue构造函数，插件中要使用（不导入还能使用）
let Vue;

class VueRouter {
  constructor(options) {
    this.$options = options;
  }
}

// 插件：实现install方法，注册$router
// 参数一是Vue.use调用时传入的
VueRouter.install = _Vue => {
  // 引⽤构造函数，VueRouter中要使⽤
  Vue = _Vue;

  // 1、挂载$router属性
  // this.$router.push()
  // 全局混入(目的：延迟下面逻辑到router创建完毕并且附加到选项上时才执行)
  Vue.mixin({
    beforeCreate() {
      // 此钩子在每个组件创建实例时都会调用
      // 根实例才有该选项
      if (this.$options.router) {
        Vue.prototype.$router = this.$options.router;
      }
    }
  });

  // 注册并且实现两个组件:router-view,router-link
  Vue.component("router-link", {});

  Vue.component("router-view", {}});
};

export default VueRouter;
```

>为什么要用混入方式写？主要原因是use代码在前，Router实例创建在后，而install逻辑又需要用到该实例

**创建router-link和router-view**

```javascript
Vue.component("router-link", {
    props: {
      to: {
        type: String,
        required: true
      }
    },
    render(h) {
      return h(
        "a",
        {
          attrs: {
            href: `#${this.to}`
          }
        },
        this.$slots.default
      );
    }
  });

  Vue.component("router-view", {
    render(h) {
      let component = null;
			// 暂时先不渲染任何内容
      return h(component);
    }
  });
```

**监控URL变化**

定义响应式的current属性，监听hashchange事件

```javascript
class VueRouter {
  constructor(options) {
    this.$options = options;
    // 把current变为响应式数据，
    // 将来数据发生变化，router-view的render函数能够再次执行
    let initial = window.location.hash.slice("#") || "/";

    Vue.util.defineReactive(this, "current", initial);

    // 监控hash变化
    window.addEventListener("hashchange", this.onhashchange.bind(this));
    window.addEventListener("load", this.onhashchange.bind(this));
  }

  onhashchange() {
    this.current = window.location.hash.slice(1) || "/";
  }
}
```

动态获取对应的组件，router-view组件

```javascript
Vue.component("router-view", {
    render(h) {
      let component = null;
      // 获取当前路由所对应的组件
      const current = this.$router.current;
      const route = this.$router.$options.routes.find(
        route => route.path === current
      );

      if (route) {
        component = route.component;
      }

      return h(component);
    }
});
```

**优化点**

提前处理路由表避免每次都循环

```javascript
class VueRouter {
  constructor(options) {
      // 缓存path和route映射关系
      this.routeMap = {}
      this.$options.routes.forEach(route => {
        this.routeMap[route.path] = route
      });
   }
}
```

router-view组件

```javascript
render(h) {
  const {routeMap, current} = this.$router
  const component = routeMap[current] ? routeMap[current].component : null;
  return h(component);
}
```



### Vuex

``Vuex``集中式存储管理应用的所有组件的状态，并以相应的规则保证状态可预测的方式发生变化。

<img src="https://vuex.vuejs.org/vuex.png" alt="vuex" style="zoom:150%;" />

**整合Vuex**

```javascript
vue add vuex
```

**核心概念**

- ``state``状态、数据
- ``mutations``更改状态的函数
- ``actions``异步操作
- ``store``包含以上概念的容器

**状态 - state**

```javascript
export default new Vuex.Store({
	state: {
    counter: 0,
  },
})
```

**状态变更 - mutations**

mutations用于修改状态，store/index.js

```javascript
export default new Vuex.Store({
  mutations: {
    add(state) {
      // state从哪里来
      state.counter++;
    },
  },
});
```

**异步操作 - actions**

添加业务逻辑，类似于controller

```javascript
export default new Vuex.Store({
  actions: {
    add({ commit }) {
      // 参数是怎么来的
      setTimeout(() => {
        commit("add");
      }, 1000);
    },
  },
});
```

**派生状态 - getters**

从state派生出新状态，类似计算属性

```javascript
export default new Vuex.Store({
  getters: {
    doubleCounter(state) {
      return state.counter * 2;
    },
  },
});
```

**测试代码**

```html
<p @click="$store.commit('add')">counter:{{ $store.state.counter }}</p>
<p @click="$store.dispatch('add')">
  async counter:{{ $store.state.counter }}
</p>
<p>double counter:{{ $store.getters.doubleCounter }}</p>
```



### Vuex原理解析

**任务分析**

- 实现插件

  - 实现Store类
    - 维持一个响应式的状态state
    - 实现commit、dispatch、getters

  - 挂载 $store

**初始化**

Store声明、install实现，vuex.js

```javascript
// 1、挂载$store
// 2、实现Store

let Vue;

class Store {
  constructor(options) {
    // data响应式处理
    // Vue会把data里面的数据递归一下，全部变成响应式数据
    this._vm = new Vue({
      data: {
        $$state: options.state,
      },
    });
  }

  get state() {
    return this._vm.$data.$$state;
  }

  set state(value) {
    console.error("please use replaceState to reset state");
  }
}

const install = (_Vue) => {
  Vue = _Vue;

  Vue.mixin({
    beforeCreate() {
      if (this.$options.store) {
        Vue.prototype.$store = this.$options.store;
      }
    },
  });
};

export default { Store, install };

```

**实现commit**

根据用户传入的type获取并执行对应mutation

```javascript
class Store {
  constructor(options) {
    // 保存⽤户配置的mutations选项
    this._mutations = options.mutations || {};
    this.commit = this.commit.bind(this);
  }

  commit(type, payload) {
    // 获取type对应的mutation
    const entry = this._mutations[type];

    if (!entry) {
      console.error("unknow action type");
      return;
    }

		// 传递state给mutation
    entry(this.state, payload);
  }
}
```

**实现actions**

根据用户传入type获取并执行对应的action

```javascript
class Store {
  constructor(options) {
    this._actions = options.actions || {};

    this.dispatch = this.dispatch.bind(this);
  }

  dispatch(type, payload) {
    // 获取⽤户编写的type对应的action
    const entry = this._actions[type];
    if (!entry) {
      console.error("unknow action type:", type);
      return;
    }

    entry(this, payload);
  }
}
```

**实现getters** 

遍历用户传入的``getters``并将所有的值变为响应式，以方法名为``key``，存入``$store.getters``中

```javascript
class Store {
  constructor(options) {
    this.getters = {};

    // 1、遍历所有的getters放置到this.getters
    // 2、用户通过this.$store.getters.doubleCounter调用时，执行对应的doubleCounter方法
    options.getters && this.handleGetters(options.getters);
  }

  handleGetters(getters) {
    Object.keys(getters).forEach((key) => {
      Object.defineProperty(this.getters, key, {
        get: () => getters[key](this.state),
      });
    });
  }
}
```




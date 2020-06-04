## 创建项目

vue cli版本大于3.0

```js
vue create vue-three-test	

Please pick a preset:
  default (babel, eslint)
> Manually select features

 Check the features needed for your project:
 (*) Babel
 ( ) TypeScript
 ( ) Progressive Web App (PWA) Support
 (*) Router
 (*) Vuex
 (*) CSS Pre-processors
 (*) Linter / Formatter
 ( ) Unit Testing
 ( ) E2E Testing 
 
cd vue-three-test
vue add vue-next (添加vue3依赖)

```

## 新特性

### 开始

setup函数作为逻辑的主入口

```
setup () {}
```

### 上下文

通过getCurrentInstance函数获取上下文ctx

```js
import { getCurrentInstance } from 'vue'
export default {
  setup () {
    const { ctx } = getCurrentInstance()
  }
}
```

### data

ref函数创建数据

```js
import { ref } from 'vue'
export default {
  setup () {
    const count = ref(0)
  }
}
```

### 生命周期

```js
import { onBeforeMount, onBeforeUnmount, onActivated, onMounted, onBeforeUpdate, onUpdated, onUnmounted, onRenderTracked, onRenderTriggered, onErrorCaptured, onDeactivated } from 'vue'
export default {
  setup () {
    onActivated(() => {
      console.log('onActivated')
    })
    onBeforeMount(() => {
      console.log('onBeforeMount')
    })
    onMounted(() => {
      console.log('onMounted')
    })
    onBeforeUpdate(() => {
      console.log('onBeforeUpdate')
    })
    onUpdated(() => {
      console.log('onUpdated')
    })
    onBeforeUnmount(() => {
      console.log('onBeforeUnmount')
    })
    onUnmounted(() => {
      console.log('onUnmounted')
    })
    onDeactivated(() => {
      console.log('onDeactivated')
    })
    // 触发渲染
    onRenderTriggered(data => {
      console.log('onRenderTriggered', data)
    })
    // 追踪渲染
    onRenderTracked(data => {
      console.log('onRenderTracked', data)
    })
    onErrorCaptured(() => {
      console.log('onErrorCaptured')
    })
  }
}
```

### 事件

内部声明函数并导出即可

```js
import { ref, computed, watch, getCurrentInstance } from 'vue'
export default {
  setup () {
    const count = ref(0)
    const handleClick = () => {
      count.value++
    }
    return { count, handleClick }
  }
}
```

### watch监听

watch函数接收两个函数，第一个函数返回监听的值，第二个函数处理监听

```js
import { ref, watch, getCurrentInstance } from 'vue'
export default {
  setup () {
    const count = ref(0)
    const handleClick = () => {
      count.value++
    }
    const { ctx } = getCurrentInstance()
    watch(() => count.value, val => {
      console.log(val)
    })
    return { count, handleClick }
  }
}
```

### 计算属性

```js
import { ref, computed } from 'vue'
export default {
  setup () {
    const count = ref(0)
    const handleClick = () => {
      count.value++
    }
    const double = computed(() => count.value * 2)
    return { count, handleClick, double }
  }
}
```

### 路由

使用createRouter创建路由

```js
import { createRouter, createWebHashHistory } from "vue-router";
import Home from "../views/Home.vue";

const routes = [
  {
    path: "/",
    name: "Home",
    component: Home
  },
  {
    path: "/about",
    name: "About",
    component: () =>
      import(/* webpackChunkName: "about" */ "../views/About.vue")
  }
];
const router = createRouter({
  history: createWebHashHistory(),
  routes
});
export default router;

```

使用ctx.$route.currentRoute.value获取路由

```js
watch(() => count.value, val => {
      console.log(val, ctx.double, ctx.$router.currentRoute.value, ctx.$refs)
})
```

### Vuex状态管理

使用createStore创建

```js
import Vuex from "vuex";

export default Vuex.createStore({
  state: {
    userName: '李四'
  },
  mutations: {
    setUserName (state, value) {
      state.userName = value
    }
  },
  actions: {},
  modules: {}
});

```

使用ctx.$store触发

```js
const changeUser = () => {
      ctx.$store.commit('setUserName', '张三')
}
```




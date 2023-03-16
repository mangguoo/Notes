# Vue-Router

## 使用view组件挂载多个component

```ts
const router = new Router({
  mode: 'history',
  base: '/custody',
  routes: [
    {
      path: '/',
      redirect: '/login',
    },
    {
      path: '/login',
      name: 'home',
      component: () => import('../views/home/index.vue'),
      meta: {
          keepAlive: true,
      },
    },
    {
      path: '/wallet',
      components: {
        default: () => import('../views/wallet/wallet-list.vue'),
        header: Header,
      },
    }
  ]
}
```

```vue
<template>
  <div id="app">
    <!-- 挂载header组件 -->
    <router-view name="header"></router-view> 
    <!-- 挂载default组件 -->
    <keep-alive>
      <!-- 需要缓存的视图组件 -->
      <router-view v-if="$route.meta.keepAlive"></router-view>
    </keep-alive>
    <!-- 不需要缓存的视图组件 -->
    <router-view v-if="!$route.meta.keepAlive"></router-view>
  </div>
</template>
```


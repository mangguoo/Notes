# pinia

> pinia是2019 年 11 月对于新版本的vue提供的组合Api进行的尝试，它可以很好的集合vue新的api方法，且还很好的支持ts的写法，Pinia已经被纳入官方代码仓库中，也可以理解为Pinia为最新版本的vuex5

**安装：**

```bash
// 安装pinia
npm i pinia -S
// 持久性插件
npm i pinia-plugin-persistedstate@2 -S
```

## pinia与vuex

**vuex与pinia的api：**

- Vuex：namespaced、State、Gettes、Mutations(同步)、Actions(异步)

- Pinia： State、Gettes、Actions(同步异步都支持)

**版本区别：**

- Vuex 当前最新版是 4.x。Vuex4 用于 Vue3，而Vuex3 用于 Vue2

- Pinia 当前最新版是2.x，它即支持 Vue2 也支持 Vue3。就目前而言 Pinia 是我们在vue3项目中的首选，尤其是 TypeScript 的项目

## 创建模块

- **/src/store/user.js**

```js
import { defineStore } from 'pinia'
// defineStore的参数一是命名空间名称，而参数二是一个对象，用于写state.getters,actoins,persist
export const userStore = defineStore('user', {
    state: () => {
        return { 
            num: 1,
            arr: []
        }
    },
    getters: {},
    actions: {
        changeState(n){ // 不能用箭头函数
            this.num += n
        }
    },
    // 当persist为true时，为当前pinia模块中的所有数据进行数据持久化
    // persist: true
    // 也可以对数据持久化进行详细的配置
    persist: {
        key: 'countKey', // 保存在storage中key的名称
        storage: window.sessionStorage, // 存储在哪里
        paths: ['num'] // 存储哪些东西
    }
})
```

## 初始化配置

- **/src/main.js**

```js
import { createPinia } from 'pinia'
import persistedState from 'pinia-plugin-persistedstate'

const pinia = createPinia()
pinia.use(persistedState) // 给pinia引入持久化

createApp(App)
  .use(pinia)
  .mount('#app')
```

## 访问state

- **/src/pages/user.vue**

```vue
<template>
  <div>{{ user_store.count }}</div>
</template>

<script setup>
import { storeToRefs } from 'pinia'
import { userStore } from '@/store/user.js'
    
const user_store = userStore()
// 使用storeToRefs解构调用，如果直接使用user_store.num进行修改没有任何问题
// 但是解构会使其失去响应性，因此想要使其保持响应性就要通过storeToRefs进行包裹后解构
const { num } = storeToRefs(user_store)
</script>
```

## action更新数据

- **/src/pages/count.vue**

```vue
<template>
    <div>{{ user_store.count }}</div>
    <button @click="handleClick">按钮</button>
</template>

<script lang="ts" setup>
import { userStore } from '@/store/user.js'
    
const user_store = userStore()
const handleClick = () => {
    // 方法一
    user_store.num++
    // 方法二，需要修改多个数据，建议用 $patch 批量更新，传入一个对象
    user_store.$patch({
        count: user_store.num++,
        // 这种方法需要传入一个新数组，类似于redux，有点消耗性能
        arr: [ ...user_store.arr, 1 ] 
    })
    // 方法三，还是$patch，传入函数，第一个参数就是 state
    user_store.$patch( state => {
        state.num++
        state.arr.push(1)
    })
    // 方法四：调用action方法
    user_store.changeState(1)
}
</script>
```

## 异步action(TS)

- **/src/api/filmApi.ts**

```ts
import { get } from '@/utils/http'

export const getFilmsApi = <T>(page = 1) => {
  return get<T>('https://api.iynn.cn/film/api/v1/getNowPlayingFilmList?cors=T&cityId=110100&pageNum=1&pageSize=10')
}
```

- **/src/pages/film/index.vue**

```vue
<template>
  <div>
    <template v-if="store.films.length === 0">
      <div>加载中...</div>
    </template>
    <template v-else>
      <ul>
        <li v-for="item of store.films" :key="item.filmId">
          {{ item.name }}
        </li>
      </ul>
    </template>
  </div>
</template>

<script setup lang="ts">
import { onMounted } from "vue";
import useFilmStore from './store'
const store = useFilmStore()
onMounted(() => {
  store.setFilmsAction()
})
</script>
```

- **/src/pages/film/store.ts**

```ts
import { defineStore } from 'pinia'
import { getFilmsApi } from '@/api/filmApi'

interface Actor {
  name: string;
  role: string;
  avatarAddress: string;
}

interface FilmType {
  name: string;
  value: number;
}

interface Item {
  name: string;
  type: number;
}

interface Film {
  filmId: number;
  name: string;
  poster: string;
  actors: Actor[];
  director: string;
  category: string;
  synopsis: string;
  filmType: FilmType;
  nation: string;
  language: string;
  videoId: string;
  premiereAt: number;
  timeType: number;
  runtime: number;
  grade: string;
  item: Item;
  isPresale: boolean;
  isSale: boolean;
}

interface FilmDataType {
  films: Film[];
  total: number;
}

const useFilmStore = defineStore('film', {
  state: () => ({
    films: [] as Film[]
  }),
  actions: {
    async setFilmsAction() {
      let ret = await getFilmsApi<FilmDataType>()
      this.films = ret.data.films
    }
  }
})

export default useFilmStore
```

## 自定义插件

> Pinia 插件是一个函数，可以选择返回要添加到 store 的属性。 它需要一个可选参数，一个 *context*：

```js
export function myPiniaPlugin(context) {
  context.pinia // 使用 `createPinia()` 创建的 pinia
  context.app // 使用 `createApp()` 创建的当前应用程序（仅限 Vue 3）
  context.store // 插件正在扩充的 store
  context.options // 定义存储的选项对象传递给`defineStore()`
  // ...
}
```

然后使用 `pinia.use()` 将此函数传递给 `pinia`：

```js
pinia.use(myPiniaPlugin)
```
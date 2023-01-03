# Vue2+TS

> 在vue2中引入ts，是为了规范项目。vue文件并不需要修改为tsx，可以直接在vue扩展名中写ts语法，只是在script标签中加入声明一下。`<script lang="ts">`
>
> 要在vue2中使用ts，是借助了vue-property-decorator：基于vue-class-component扩展很多装饰器，让vue2支持ts。

**创建项目：**

```js
vue create myapp_ts
```

创建项目中，使用Manually select features，也就是自定义特性，在自定义特性时，只需要勾选上typescirpt即可。与js项目不同的是，他会增加几个选项:

- Use class-style component syntax：表示是否显示类组件
- Use Babel alongside Typescript(required for modern mode, auto-detected polyfills, transpiling JSX)：表示是否同时使用babel和typescitp进行编译，用于转换jsx

**目录文件区别:**

> vue_ts项目中，不仅增加了ts编译配置文件tsconfig.json，而且在src目录下增加了两个类型声明文件：

- /src/shims-tsx.d.ts(让项目支持tsx文件)

```ts
import Vue, { VNode } from "vue";

declare global {
  namespace JSX {
    interface Element extends VNode {}
    interface ElementClass extends Vue {}
    interface IntrinsicElements {
      [elem: string]: any;
    }
  }
}
```

- /src/shims-vue.d.ts(让项目支持vue文件)

```js
declare module "*.vue" {
  import Vue from "vue";
  export default Vue;
}
```

## props属性

```vue
<template>
  <div>
    <desn :title="'秋天，多喝这道冬瓜排骨汤，暖身暖胃，营养滋补降秋燥，真鲜美'" />
    <desn />
  </div>
</template>

<script lang="ts">
// 此库是vue2+ts实现的关键，每个组件都必须要使用
import { Component, Vue } from 'vue-property-decorator'
import desn from '@/components/Desn.vue'

// 注册局部组件
@Component({
  components: {
    desn,
  }
})
export default class Home extends Vue {}
</script>

<style scoped></style>
```

```vue
<template>
  <div>{{ title }}</div>
</template>

<script lang="ts">
// Prop 用于获取父组件传过来数据的声明
import { Component, Vue, Prop } from 'vue-property-decorator'

@Component
export default class Desn extends Vue {
  // 注意，这里使用非空断言，是因为这里不赋值会报错，但是又不能赋值，因为单向数据流
  @Prop() title!: string
  // 默认值
  @Prop({ default: '默认值' }) title!: string
  @Prop({ default: () => '默认值@@@' }) title!: string
  // 如果定义为私有，则template模板中无法访问中，只有在类中才能使用
  @Prop({ default: () => '默认值@@@' }) private title!: string
  // 必填属性
  @Prop({ required: true }) title!: string
}
</script>

<style scoped></style>
```

## tsx组件

> 计数器

```tsx
import { Component, Vue, Prop, Emit } from 'vue-property-decorator'

// 像react写法，但是和react还是有一些不同
@Component
export default class Count extends Vue {
  @Prop() num!: number
  @Prop() setNum!: (n: number) => void

  n = 1

  render() {
    return (
      <div>
        <h3>我是count组件，里面直接写jsx就可以 --- {this.num}</h3>
        <button
          onClick={() => {
            this.setNum(this.n)
          }}
        >
          +++++
        </button>
      </div>
    )
  }
}
```

```vue
<template>
  <div>
    <count :num="num" :setNum="setNum" />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'
import count from '@/components/Count.tsx'

@Component({
  components: {
    count
  }
})
export default class Home extends Vue {
  num = 100
  setNum(n: number) {
    this.num += n
  }
}
</script>

<style scoped></style>
```

## watch监听器

```vue
<template>
  <div>
    <input type="number" v-model.number="num" />
    <input type="number" v-model.number="obj.num" />
  </div>
</template>

<script lang="ts">
import { Component, Vue, Watch } from 'vue-property-decorator'

@Component
export default class AboutView extends Vue {
  num = 1
  // 初始化时，就执行
  @Watch('num', { immediate: true })
  watchNum(newValue: any, oldValue: any) {
    console.log(newValue)
  }

  obj = { num: 100 }
  // 如果是深度监听，则新和旧的值是一样的，因为vue并不会对旧值做一次拷贝
  @Watch('obj', { deep: true })
  watchObj(newValue: any, oldValue: any) {
    console.log({ ...newValue })
  }
}
</script>

<style scoped></style>
```

## computed计算属性

```vue
<template>
  <div>
    <h3>{{ showNum }}</h3>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Watch } from 'vue-property-decorator'

@Component
export default class AboutView extends Vue {
  num = 1
  // 计算属性，使用的就是类的访问器，与computed相比，失去了缓存功能
  get showNum() {
    return this.num > =3 ? '保密' : this.num
  }
}
</script>

<style scoped></style>
```

## mixin混入

```vue
<template>
  <div>
    <h3>{{ headerTitle }}</h3>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Mixins } from 'vue-property-decorator'
import MsgMixin from '@/mixins/msgMixin'

@Component
export default class User extends Mixins(MsgMixin) {}
</script>

<style scoped></style>
```

```ts
import { Component, Vue } from 'vue-property-decorator'

@Component
export default class MsgMixin extends Vue {
  headerTitle = '混入的内容'
}
```

## Provider Inject

> `@Provider @Inject`
>
> `@ProvideReactive @InjectReactive`
>
> 在vue2中，provider提供的数据不是响应式的，如果想要使其变成响应式，就需要借助函数的作用域来实现。但是vue_ts的装饰器解决方案中，提供了两个方法：@ProvideReactive @InjectReactive，他们使得provider的数据是响应式的

```vue
<template>
  <child />
</template>

<script lang="ts">
import { Component, Vue, Mixins, Provide, ProvideReactive } from 'vue-property-decorator'

import child from '@/components/Child.vue'

@Component({
  components: { child }
})
export default class User extends Vue {
  @ProvideReactive('num') num = 100
  @Provide('setNum')
  setNum() {
    this.num += 1
  }
}
</script>

<style scoped></style>
```

```vue
<template>
  <div>
    <h3>{{ num }}</h3>
    <button @click="setNum">++++</button>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Inject, InjectReactive, Ref } from 'vue-property-decorator'

@Component
export default class Child extends Vue {
  @InjectReactive('num') num!: number
  @Inject('setNum') setNum!: () => void
}
</script>

<style scoped></style>
```

## ref引用

```vue
<template>
  <div>
    <input type="text" ref="inputRef" />
    <son ref="usernameRef" />
    <button @click="getUserNameInput">get</button>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Ref } from 'vue-property-decorator'
import son from './Son.vue'

@Component({
  components: { son }
})
export default class Child extends Vue {
  // 引用dom元素
  @Ref('inputRef') iRef!: HTMLInputElement
  // 引用组件
  @Ref('usernameRef') uRef!: any
    
  getUserNameInput() {
    console.log(this.uRef.num)
    console.log(this.iRef.value)
  }
}
</script>

<style scoped></style>
```

- **./Son.vue**

```vue
<template></template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'

@Component
export default class Son extends Vue {
  num = 100000
}
</script>

<style scoped></style>
```

## .sync

```vue
<template>
  <son :count.sync="count" />
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'
import son from './Son.vue'

@Component({
  components: { son }
})
export default class Child extends Vue {}
</script>

<style scoped></style>
```

> @PropSync用于装饰.sync修饰符传过来的props属性，以简化双向绑定的语法，可以把它的作用理解为下面这种写法：
>
> ```js
> computed: {
>     msg: {
>        get() {
>          return this.msg
>        },
>        set(val) {
>       this.$emit('update:msg',val)
>        }
>     }
> }
> ```
> 

```vue
<template>
  <input :value.sync="linkMsg" />
</template>

<script lang="ts">
import { Component, Vue, PropSync } from 'vue-property-decorator'

@Component
export default class extends Vue {
  @PropSync('msg') linkMsg!: string
  setMsg() {
    this.linkMsg = Date.now() + ''
  }
}

</script>

<style scoped></style>
```

**模拟.sync:**

```vue
<template>
  // 注意：事件名如果是小驼峰写法，那么在模板中就必须使用横线分隔的写法
  <son :count="count" @set-count="onSetCount" />
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'
import son from './Son.vue'

@Component({
  components: { son }
})
export default class Child extends Vue {
  count = 100
  onSetCount(n: number) {
    this.count += n
  }
}
</script>

<style scoped></style>
```

```vue
<template>
  <div>
    <h3>{{ count }}</h3>
    <button @click="setCount(10)">+++</button>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Prop, Emit } from 'vue-property-decorator'

@Component
export default class Son extends Vue {
  @Prop() count!: number
  // 这里相当于写了this.$emit('setCount', n)，在父组件调用时，要注意小驼峰问题 @set-count
  @Emit()
  setCount(n: number) {}
}
</script>

<style scoped></style>
```

## v-model

```vue
<template>
  <h3>{{ msgtitle }}</h3>
  // 自定义组件上使用v-model，默认为value属性和input事件
  <son v-model="msgtitle" />
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'
import son from './Son.vue'

@Component({
  components: { son }
})
export default class Child extends Vue {
  msgtitle = 'aaaa'
}
</script>

<style scoped></style>
```

> 使用v-model的自定义props属性和事件来进行双向绑定操作

```vue
<template>
  <input :value="value" @input="$emit('input', $event.target.value)" />
</template>

<script lang="ts">
import { Component, Vue, Prop } from "vue-property-decorator";

@Component
export default class Son extends Vue {
  // 使用readonly修改，使得他只读，防止被修改，实现单向数据流
  @Prop() readonly value!: string
}
</script>

<style scoped></style>
```

> 通过@model来实现自定义v-model事件名和属性名，使用@model后就不需要使用@prop来接收属性了，它会帮我们来接收，并且对事件名和属性名进行自定义

```vue
<template>
  <input :value="value" @input="$emit('input', $event.target.value)" />
</template>

<script lang="ts">
import { Component, Vue, Model } from "vue-property-decorator";

@Component
export default class Son extends Vue {
  @Model('change') readonly titleMsg!: string
}
</script>

<style scoped></style>
```

## 生命周期钩子

```vue
<template></template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'

@Component
export default class Film extends Vue {
  // 如果要在created中做网络请求，优点是跨平台兼容比较好，缺点就是会增加白屏时间
  created() {
    console.log('created')
  }
  beforeMount() {
    console.log('beforeMount')
  }
  // 这个阶段，dom已经挂载到页面了，推荐在这里进行网络请求
  mounted() {
    console.log('我是mounted生命周期方法')
  }
  updated() {
    console.log('updated')
  }
  // 销毁处理
  beforeDestroy() {
    console.log('beforeDestory')
  }
}
</script>

<style scoped></style>
```

## 网络请求

- **/src/views/Films.vue**

```vue
<template>
  <ul>
    <li v-for="item in films" :key="item.filmId">
        {{ item.name }}
    </li>
  </ul>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'
import { getFilmListApi } from '@/api/filmApi'
import type { FilmInfoType } from '@/typings/filmTypes'

@Component
export default class Film extends Vue {
  films: FilmInfoType[] = []
  async created() {
    let ret = await getFilmListApi(1)
    this.films = ret.data.films
  }
}
</script>

<style scoped></style>
```

- **/src/typings/filmTypes**

```ts
export interface Actor {
  name: string;
  role: string;
  avatarAddress: string;
}

export interface FilmType {
  name: string;
  value: number;
}

export interface Item {
  name: string;
  type: number;
}

export interface FilmInfoType {
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
  timeType: number | string;
  runtime: number;
  grade: string;
  item: Item;
  isPresale: boolean;
  isSale: boolean;
}

export interface Data {
  films: FilmInfoType[];
  total: number;
}

export interface FilmRootObject {
  msg: string;
  status: number;
  cityId: number;
  data: Data;
}
```

- **/src/api/filmApi.ts**

```ts
import { get } from '@/utils/http'
import type { FilmRootObject } from '@/typings/filmTypes'

export const getFilmListApi = (page = 1) => {
  let url = `https://api.iynn.cn/film/api/v1/getNowPlayingFilmList?cors=T&cityId=110100&pageNum=${page}&pageSize=10`
  return get<FilmRootObject>(url)
}

```

- **/src/utils/http.ts**

```ts
import axios from "axios";
import type { AxiosRequestConfig } from 'axios'

const instance = axios.create({
  timeout: 10000
})

instance.interceptors.response.use(
  res => res.data,
  err => Promise.reject(err)
)

export const get = <T>(url: string, config: AxiosRequestConfig<any> = {}) => {
  // 直接覆盖第二个泛型参数，因为这里对res进行了解套
  return instance.get<any, T>(url, config)
}
```

## VueRouter

- **/src/router/index.ts**

```ts
import Vue from 'vue'
import VueRouter from 'vue-router'
import routes from './routes'
Vue.use(VueRouter)

const router = new VueRouter({
  mode: 'history',
  base: process.env.BASE_URL,
  routes
})

export default router
```

- **/src/router/routes.ts**

```ts
import type { RouteConfig } from 'vue-router'

const routes: RouteConfig[] = [
  {
    path: '/',
    component: () => import('../views/HomeView.vue')
  },
  {
    path: '/about',
    component: () => import('../views/AboutView.vue')
  },
  {
    path: '/user',
    component: () => import('../views/User.vue')
  },
  {
    path: '/film',
    component: () => import('../views/Film.vue')
  },
  {
    path: '/news',
    component: () => import('../views/News/index.vue')
  },
]

export default routes
```

## Vuex

- **/src/store/index.ts**

```ts
import Vue from 'vue'
import Vuex from 'vuex'
Vue.use(Vuex)
export default new Vuex.Store({})
```

- **/src/views/News/state.ts**

> 通过Module装饰器来动态创建模块并加载到vuex的状态节点上，因为要动态添加，所以需要导入store对象

```ts
import { Module, getModule, VuexModule, Mutation, Action } from 'vuex-module-decorators'
import store from '@/store'
import type { FilmInfoType } from '@/typings/filmTypes'
import { getFilmListApi } from '@/api/filmApi'

@Module({ store: store, namespaced: true, name: 'news', dynamic: true })
class NewsModule extends VuexModule {
  num = 100
  films = [] as FilmInfoType[]

  @Mutation
  setNum(payload: number) {
    this.num += payload
  }

  @Mutation
  setFilms(payload: FilmInfoType[]) {
    this.films = payload
  }

  @Action
  asyncSetNum(payload: number) {
    setTimeout(() => {
      this.context.setNum('setNum', payload)
    }, 1000);
  }

  @Action
  async fetchFilmsAction(page = 1) {
    let ret = await getFilmListApi(page)
    this.context.commit('setFilms', ret.data.films)
  }

}

export default getModule(NewsModule)
```

- **/src/views/index.vue**

```vue
<template>
  <div>
    <h3>{{ num }}</h3>
    <button @click="addNum">++++</button>
    <ul>
      <li v-for="item in films" :key="item.filmId">
        {{ item.name }}
      </li>
    </ul>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'
import newsModule from './state'

@Component
export default class News extends Vue {
  mounted() {
    newsModule.fetchFilmsAction(1)
  }

  addNum() {
    newsModule.asyncSetNum(1)
  }

  get num() {
    return newsModule.num
  }

  get films() {
    return newsModule.films
  }
}
</script>

<style scoped></style>
```


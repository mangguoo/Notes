# vue3+ts

## ref的类型

 ```vue
 <script setup lang="ts">
 import { ref } from "vue"
 import type { Ref } from "vue" // 在ts中类型，使用 type 来单独引入
 
 let num = ref<number>(100)
 let num = ref(100);
 let num: Ref<number> = ref(100)
 
 const addNum = () => {
   num.value++;
 };
 </script>
 ```

**ref获取组件实例：**

> InstanceType是用来获取类的实例的类型的
>
> ```ts
> type InstanceType<T extends new (...args: any[]) => any> = T extends new (...args: any[]) => infer R ? R : any;
> ```

```vue
<template>
  <div>
    <child ref="childRef" />
    <button @click="getChild">获取child组件</button>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import child from "./components/child.vue";

// typeof child用于获取child类的类型，而InstanceType表示获取child类实例的类型，因为ref其实获取到的是组件实例
const childRef = ref<InstanceType<typeof child>>();

const getChild = () => {
  // 通过ref获取自定义组件实例中的方法，一定要加一个 ？或 !
  // 这是因为childRef的类型其实是 Ref<InstanceType<typeof child>> | null，因为组件挂载之后childRef才有值，在setup中定义时它是没有值的
  // 但是我们在获取的时候是可以确定它一定是有值的，因此可以使用非空断言来告诉类型检查器不要报错
  console.log(childRef.value?.addNum);
  console.log(childRef.value && childRef.value.addNum);
  console.log((childRef.value as InstanceType<typeof child>).addNum);
};
</script>
```

## reactive的类型

```vue
<script setup lang="ts">
import { reactive } from 'vue'
    
const state = reactive({ num: 200 })
const state: { num: number } = reactive({ num: 200 })
const state = reactive<{ num: number }>({ num: 200 })
const state = reactive({ num: 200 }) as { num: number }
</script>
```

## 父子传值(props,emit,expose)

```vue
<template>
  <child ref="childRef" :title="title" @onChangeTitle="setTitle" />
</template>

<script setup lang="ts">
import { ref } from "vue";
import child from "./components/child.vue";

const title = ref("我父组件中的标题");
const setTitle = (msg: string) => {
  title.value = msg;
};
</script>
```

```vue
<template>
  <h3>{{ title }}</h3>
  <button @click="changeTitle">change title</button>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'
// 定义参数类型
interface IProps {
  title: string
}
defineProps<IProps>()
// 自定义事件类型
const emit = defineEmits<{
  (e: 'onChangeTitle', value: string): void
}>()
// 定义方法
const changeTitle = () => {
  emit('onChangeTitle', Date.now() + '')
}
// 暴露接口
const exposeParam = 1
defineExpose<{ exposeParam: number }>({ exposeParam })
</script>
```

## 路由类型

- **/src/router/index.ts**

```ts
import { createRouter, createWebHistory } from 'vue-router'
import routes from './rotues'
import useUserStore from '@/store/user'


const router = createRouter({
  history: createWebHistory(),
  routes
})

router.beforeEach((to, from) => {
  if (!to.meta.islogin) {
    return true
  }
  // 验证是否已经登录过
  if (!useUserStore().token) {
    return '/login'
  }
  return true;
})

export default router
```

- **/src/router/routes.ts**

```ts
import type { RouteRecordRaw } from 'vue-router'

const routes: RouteRecordRaw[] = [
  {
    path: '/login',
    component: () => import('../pages/login/index.vue')
  },
  {
    path: '/',
    component: () => import('../pages/admin/index.vue'),
    meta: {
      islogin: true // 标志访问该页面需不需要登录
    },
    children:[
      {
        path:'user/index',
        component: () => import('../pages/user/index.vue'),
      }
    ]
  }
]

export default routes
```

### 路由参数

```vue
<script setup lang="ts">
import { useRouter, useRoute } from 'vue-router'
const router = useRouter()
const route = useRoute()

// 建议使用一下 ? 来链式判断
const id = route.query?.id

const goHome = () => {
  router.push('/')
}
</script>
```

## store类型

- **/src/store/user.ts**

```ts
import { defineStore } from "pinia";

interface UserLoginType {
  uid: number;
  token: string;
  nickname: string;
}

const useUserStore = defineStore("user", {
  persist: {
    storage: window.sessionStorage,
  },
  state: () => ({
      uid: 0,
      token: "",
      nickname: "",
  } as UserLoginType),
  actions: {
    setUserInfo(userData: UserLoginType) {
      this.$patch(userData);
    },
  },
});

export default useUserStore;
```

## 网络请求

- **/src/utils/http.ts**

```ts
import axios from "axios";

export interface DataRootType<T> {
  msg: string;
  code: number;
  data: T;
}

const instance = axios.create({
  timeout: 10000,
});

instance.interceptors.response.use(
  (res) => res.data,
  (err) => Promise.reject(err)
);

export const get = <T>(url: string, config: any = {}) => {
  return instance.get<any, DataRootType<T>>(url, config);
};

export const post = <T>(url: string, data = {}, config: any = {}) => {
  return instance.post<any, DataRootType<T>>(url, data, config);
};
```

- **/src/api/userApi.ts**

```ts
import { post } from "../utils/http";

interface UserLoginType {
  uid: number;
  token: string;
  nickname: string;
}

interface FormUserDataType {
  username: string;
  password: string;
}

export const doLoginApi = (userData: FormUserDataType) => post<UserLoginType>("/api/login", userData);
```

## 登录组件逻辑

- **/src/main.ts**

> **安装elementplus**： `npm install element-plus`

```ts
import { createApp } from 'vue'
import App from '@/App.vue'
import router from '@/router'
import { createPinia } from 'pinia'
import persistedState from 'pinia-plugin-persistedstate'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'

createApp(App)
  .use(ElementPlus)
  .use(router)
  .use(createPinia().use(persistedState))
  .mount('#app')
```

- **/src/pages/login/index.vue**

```vue
<template>
  <el-form ref="ruleFormRef" :model="ruleForm" status-icon :rules="rules" label-width="120px">
    <el-form-item label="账号" prop="username">
      <el-input v-model="ruleForm.username" autocomplete="off" />
    </el-form-item>
    <el-form-item label="密码" prop="password">
      <el-input v-model="ruleForm.password" autocomplete="off" />
    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="submitForm(ruleFormRef)">登录</el-button>
    </el-form-item>
  </el-form>
</template>

<script setup lang="ts">
import { reactive, ref } from 'vue'
import { ElMessage } from 'element-plus'
import type { FormInstance, FormRules } from 'element-plus'
import { validateUsername } from './validate'
import { doLoginApi } from '@/api/userApi'
import useUserStore from '@/store/user'
import { useRouter } from 'vue-router'

const userStore = useUserStore()
const { replace } = useRouter()
const ruleFormRef = ref<FormInstance>() // 通过ref得到当前Form表单组件实例对象
const ruleForm = reactive({ // 表单项中的值
  username: '',
  password: '',
})
// 表单验证规则
export const validateUsername = (rule: any, value: string, callback: any) => {
  if (value.includes("z")) {
    return callback(new Error("账号中不能为z"));
  }
  return callback()
}
const rules = reactive<FormRules>({
  username: [
    { required: true, message: '账号不能为空' },
    { validator: validateUsername, trigger: 'blur' }
  ],
  password: [
    { required: true, message: '密码不能为空', whitespace: true }
  ]
})
// 表单提交
const submitForm = (formEl: FormInstance | undefined) => {
  if (!formEl) return
  formEl.validate(async (valid) => {
    if (valid) {
      let ret = await doLoginApi({ ...ruleForm })
      if (ret.code != 0) {
        ElMessage({
          message: '账号或密码不正确。',
          showClose: true,
          type: 'error'
        })
        return false;
      }
      userStore.setUserInfo(ret.data) // 需要把个人信息保存到pinia中,并且进行了持久化操作
      replace('/') // 跳转到后台
    } else {
      ElMessage({
        showClose: true,
        message: '账号或密码不正确。',
        type: 'error',
      })
      return false
    }
  })
}
</script>
```


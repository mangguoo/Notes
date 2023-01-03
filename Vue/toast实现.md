# toast实现

## vue2

```js
import Vue from 'vue';
import { i18n } from '@/locale';
import Main from './main.vue';

const UpLoadConstructor = Vue.extend(Main);

const UpLoad = (options = {}) => new Promise((resolve, reject) => {
  if (Vue.prototype.$isServer) return;
  // 和new Vue一样，可以传入一些参数
  // 比如用propsData来传入参数,用i18n来进行多语言适配,用el来决定挂载到哪里
  const instance = new UpLoadConstructor({
    propsData: options,
    i18n, 
    // el: document.body
  });
  instance.$mount(document.body);
  instance.dialogVisible = true;

  // 监听成功事件
  instance.$on('success', (re) => {
    resolve(re);
  });

  // 监听失败事件
  instance.$on('error', (e) => {
    reject(e);
  });
});

export default UpLoad;
```

## Vue3

> 在`vue3`中已经不再支持`vue.extend`，那用`vue.extend`实现的全局弹窗之类的功能那`vue3`的替代方案又是什么呢？这里推荐使用`createVNode`和`render`，`createVNode`用来创建虚拟`DOM`，而`render`可以将虚拟`DOM`转换成真实`DOM`并插入到第二个参数里。

- **toast.vue**

```vue
<template>
  <div :class="`notice ${toast_type}`" v-if="show_toast">
    <div class="icon">
      <WlPic :src="toast_icon" alt="" />
    </div>
    <p class="text">{{ message }}</p>
  </div>
</template>

<script setup lang="ts">
import { computed } from "vue";
import {
  toast_success_icon,
  toast_alert_icon,
  toast_warning_icon,
  toast_error_icon,
} from "@packages/utils/cdn";
import WlPic from "@packages/components/Common/WlPic/src/index.vue";

interface Props {
  show_toast: boolean;
  message: string;
  toast_type?: "success" | "alert" | "warning" | "error";
  duration?: number;
}
const props = withDefaults(defineProps<Props>(), {
  show_toast: true,
  message: "",
  toast_type: "success",
  duration: 1500
});

const toast_icon = computed(() => {
  const dict = {
    success: toast_success_icon,
    alert: toast_alert_icon,
    warning: toast_warning_icon,
    error: toast_error_icon,
  };
  return dict[props.toast_type];
})
</script>

<style lang="scss" scoped>
@keyframes slideInDown {
  from {
    -webkit-transform: translate3d(-50%, -100%, 0);
    transform: translate3d(-50%, -100%, 0);
    visibility: visible;
  }
  to {
    -webkit-transform: translate3d(-50%, 20px, 0);
    transform: translate3d(-50%, 20px, 0);
  }
}
.notice {
  box-sizing: border-box;
  padding: 8px 16px;
  color: $default-black-color;
  animation: slideInDown 1s;
  display: inline-flex;
  align-items: center;
  border-radius: 30px;
  position: fixed;
  top: 0;
  left: 50%;
  transform: translate3d(-50%, 20px, 0);
  z-index: 9999;
  font-size: 14px;
  .icon {
    width: 18px;
    height: 18px;
    flex-shrink: 0;
  }
  .text {
    flex: 1;
    word-break: break-all;
    margin-left: 5px;
  }
  &.success {
    border: 1px solid #70cda1;
    background-color: #f1faf5;
  }
  &.alert {
    border: 1px solid #1c9fff;
    background-color: #e8f6ff;
  }
  &.warning {
    border: 1px solid #ff9444;
    background-color: #fff5ed;
  }
  &.error {
    border: 1px solid #ff4545;
    background-color: #ffeded;
  }
}
</style>
```

- **toast.ts**

```ts
import { createVNode, render } from "vue";
import Notice from "./src/index.vue";
interface IOptions {
  message: string;
  toast_type?: "success" | "alert" | "warning" | "error";
  duration?: number;
}
let mountNode = null;
export default (options: IOptions) => {
  const duration = options.duration | 2000;
  //确保只存在一个弹框，如果前一个弹窗还在，就移除
  if (mountNode) {
    document.body.removeChild(mountNode);
    mountNode = null;
  }
  //将options参数传入，并将Notice组件转换成虚拟DOM，并赋值给app
  const app = createVNode(Notice, {
    ...options,
    duration,
    show_toast: true,
  });
  //创建定时器，duration时间后将mountNode移除
  let timer = setTimeout(() => {
    document.body.removeChild(mountNode);
    mountNode = null;
    clearTimeout(timer);
  }, duration);
  //创建一个空的div
  mountNode = document.createElement("div");
  //render函数的作用就是将Notice组件的虚拟DOM转换成真实DOM并插入到mountNode元素里
  render(app, mountNode);
  //然后把转换成真实DOM的Notice组件插入到body里
  document.body.appendChild(mountNode);
};
```

- **使用toast组件**

```js
Notice({ message: "hello world" })
```
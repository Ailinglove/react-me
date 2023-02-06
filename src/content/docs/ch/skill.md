---
title: "日常记录"
description: "网站大全"
---

1. **设置vscode保存代码时自动fix**  
然后在 VSCode 中 cmd + shift + P 后输入 Open Settings(JSON) 打开配置文件，加入以下配置
``` 
"editor.codeActionsOnSave": {
  "source.fixAll": true
}
```
2. **安全区域**  
针对ios手机的刘海屏、药丸屏
```
  padding-top: env(safe-area-inset-top);
  padding-right: env(safe-area-inset-right);
  padding-bottom: 50px;  /* 兼容不支持 env( ) 的设备  */
  padding-bottom: calc(env(safe-area-inset-bottom) + 50px); /* 在 iphone x + 中本句才会生效 */
  padding-left: env(safe-area-inset-left);
```

### 一些问题
#### 1. history模式和hash模式有啥区别？
前端路由：history和hash模式 https://juejin.cn/post/7127143415879303204  
单页面应用，说白了只有一个index.html静态文件，后期一系列跳转都是在这一个唯一的页面中完成的  
hash模式，#/   
hash模式的所有的工作都是在前端完成的，完全不需要后端服务的配合  
hash模式的实现方式就是通过监听URL中的hash部分的变化，从而作出对应的渲染逻辑window.onhashchange=()=>{console.log('aaa')}  
hash模式下，url中会带有#，看起来不太美观  
history模式要归功于H5的history全局对象  

#### 2. v-if和v-show的区别

v-if会调用addIfCondition方法，生成vnode的时候会忽略对应节点，render的时候就不会渲染；
v-show会生成vnode，render的时候也会渲染成真实节点，只是在render过程中会在节点的属性中修改show属性值，也就是常说的display；
频繁切换用v-show，不频繁切换用v-if，因为v-if是添加和删除节点，开销更大

v-for和v-if不能一起使用的原因
某元素被5循环了5次，v-if也分别执行了5次

#### 3. computed和watch
- computed
  + 支持缓存，只有当依赖的数据发生变化后才重新计算
  + 不支持异步
  + 如果一个属性由其他属性计算而来，这个属性依赖其他的属性，一般会用computed
- watch
  + 不支持缓存，监听的属性只要一变，就会触发相应的操作
  + 异步监听
#### 4. 组件可以直接改变父组件的数据吗  
不可以，为了保证父子组件的单向数据流，防止意外的改变父组件状态，使得应用的数据流变得难以理解，导致数据流混乱。因此只能通过$emit派发一个自定义事件，伏组件接收到后，由父组件修改

### 组件
- 倒计时组件
```
<!--倒计时组件-->
<script setup="">

import dayjs from 'dayjs';

const props = defineProps(['timeStamp']);
const diffTime = reactive({
  days: '-',
  hours: '-',
  minutes: '-',
  seconds: '-',
});

watch(() => props.timeStamp, (newVal) => {
  if (newVal) {
    formatDiff(newVal);
  }
});
// 格式化距离结束时间的差
const formatDiff = (timestamp) => {
  const days = Math.floor(timestamp / (24 * 3600 * 1000));
  const leave1 = timestamp % (24 * 3600 * 1000);    // 计算天数后剩余的毫秒数
  const hours = Math.floor(leave1 / (3600 * 1000));
  const leave2 = leave1 % (3600 * 1000);        // 计算小时数后剩余的毫秒数
  const minutes = Math.floor(leave2 / (60 * 1000));
  const leave3 = leave2 % (60 * 1000);      // 计算分钟数后剩余的毫秒数
  const seconds = Math.round(leave3 / 1000);
  diffTime.days = days;
  diffTime.hours = hours;
  diffTime.minutes = minutes;
  diffTime.seconds = seconds;
};

const computeTime = () => {
  const diffData = Math.abs(dayjs(new Date()).diff(dayjs.unix(props.timeStamp)));
  formatDiff(diffData);
};


onMounted(() => {
  const now = dayjs(new Date()).unix();
  if (dayjs(now).isAfter(props.timeStamp)) {
    // 今天是券使用到期时间之后
    diffTime.minutes = 0;
    return;
  }
  setInterval(() => computeTime(), 1000);
});
</script>

<template>
  <div class="time-wrap">
    <div class="time-up" v-if="diffTime.minutes==='-'"></div>
    <div class="time" v-else-if="diffTime.minutes">
      限免剩余 <span>{{ diffTime.days }}</span>天<span>{{ diffTime.hours }}</span>时<span>{{ diffTime.minutes }}分</span><span>{{ diffTime.seconds
      }}</span>秒
    </div>
    <div class="time-up" v-else>限免已结束</div>
  </div>
</template>

<style lang="less" scoped>
.time-wrap {
  font-size: 12px;

  span {
    color: #000028;
  }
}
</style>

```




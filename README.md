
# vue-lm-iframe
---
### 力码算费带出 iframe 


### 安装与基本用法
```
$ npm install  --save  vue-lm-iframe

```
全局注册方式
```
import Vue from 'vue'
import VueLmIframe from 'vue-lm-iframe'
Vue.component('vue-lm-iframe', VueLmIframe)
```
你现在就可以使用该组件了(示例代码)
```
<template>
  <div id="app">
    <Vue-Lm-Iframe
      v-draggable="draggableValue"
      v-if="iframeVisible"
      @closeIframeData="iframeVisible = false"
      :iframeUrl="newUrl"
      @listenChildEvent="setFormData"
    ></Vue-Lm-Iframe>
  </div>
</template>

<script>
import VueLmIframe from 'vue-lm-iframe'
export default {
  name: 'app',
  components:{
    VueLmIframe
  },
  data () {
    return {
      iframeVisible: false,
      newUrl:''
    }
  },
  methods:{
	setFormData(Data) {
	     //组件传回来的数据 
    }
 }
}
</script>
```
### props 说明： 

`iframeUrl`  ：   产品url 

### 方法说明： 

 `@listenChildEvent  ` :   组件回传的参数

- 返回参数说明

![参数说明](https://shop.zhongbaounion.com/pic/hza/cssm.jpg)







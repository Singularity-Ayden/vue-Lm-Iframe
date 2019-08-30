
# vue-Lm-Iframe
---
>力码算费带出 iframe 




### 安装与基本用法
```
$ npm install --save vue-Lm-Iframe
```
全局注册
```
import Vue from 'vue'
import VueRulerTool from 'vue-ruler-tool'

Vue.component('vue-Lm-Iframe', VueLmIframe)
```
你现在就可以使用该组件了
```
<template>
  <div id="app">
    <Vue-Lm-Iframe
      v-if="iframeVisible"
      @closeIframeData="iframeVisible = false"
      :iframeUrl="newUrl"
      @listenChildEvent="setFormData"
    ></Vue-Lm-Iframe>
  </div>
</template>

<script>
import VueLmIframe from 'vue-Lm-Iframe'
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
  }
}
</script>



# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build
```


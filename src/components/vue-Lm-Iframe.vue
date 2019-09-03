<template>
  <div class="proIframe" v-draggable="draggableValue">
    <div class="proIframeHeader">
      <img src="image/count_16x16.svg" class="icon_count_16x16" alt />
      <span>保费测算</span>
      <i class="el-icon-close" @click="$emit('closeIframeData')"></i>
    </div>
    <iframe :src="iframeUrl" frameborder="0" class="iframeWebView" width="100%" height="552px"></iframe>
  </div>
</template>
<script>
import { Draggable } from "draggable-vue-directive";
export default {
  directives: {
    Draggable
  },
  props: {
    iframeUrl: {
      type: String,
      default: ""
    }
  },
  data() {
    return {
      draggableValue: {}
    };
  },
  methods: {
    getFormData(data) {
      if (data && data.source === "lima") {
        this.$emit("listenChildEvent", this.formatLiMaDataHandle(data.res));
        return;
      }
      // 小帮齐欣项目
      if (data && data.source === "xbqx") {
        let sourceData = JSON.parse(data.res.qxData);
        let soruceObj = {
          fee: data.res.price,
          baseFactors: [],
          additionalFactors: []
        };
        sourceData.map(v => {
          console.log(v);
          let obj = {
            factorName: v.name,
            factorCode: v.key,
            factorValueName: v.value,
            factorValueCode: v.value
          };
          if (
            v.key == "malignantTumorSecondaryCompensation" ||
            v.key == "additionalInsureAgeLimit"
          ) {
            soruceObj.additionalFactors.push(obj);
          } else {
            soruceObj.baseFactors.push(obj);
          }
        });
        this.$emit("listenChildEvent", soruceObj);
        return;
      }
      return;
    },

    getPostMessage() {
      let that = this;
      window.addEventListener(
        "message",
        e => {
          if (e.origin.indexOf("zhongbaounion.com") === -1) return;
          if (e.data === "close") {
            that.$emit("closeIframeData");
            return;
          }

          let str = e.data;
          console.log(str);
          this.getFormData(str);
          that.$emit("closeIframeData");
          return;
        },
        false
      );
    }
  },
  mounted() {
    this.getPostMessage();
    // 拖动 初始化参数
    this.draggableValue.boundingElement = document.getElementsByTagName(
      "body"
    )[0];
    this.draggableValue.boundingRectMargin = {
      top: 2,
      bottom: 2,
      left: 2,
      right: 2
    };
  }
};
</script>
<style lang="stylus" scoped>
.proIframe {
  position: fixed;
  width: 376px;
  height: 608px;
  background-color: #f6f6f6;
  z-index: 10000;
  left: calc(50vw + 304px);
  top: 15vh;

  .proIframeHeader {
    box-sizing: border-box;
    padding: 0 16px;
    height: 55px;
    width: 100%;
    display: flex;
    align-items: center;
    background-color: #ffffff;
    border-bottom: 1px solid #e8e8e8;

    .icon_count_16x16 {
      width: 14px;
      height: 16px;
    }

    span {
      margin-left: 8px;
      font-family: Microsoft YaHei;
      font-size: 16px;
      font-weight: normal;
      font-stretch: normal;
      letter-spacing: 0;
      color: #333333;
    }

    .el-icon-close {
      margin-left: 240px;
    }
  }
}
</style>

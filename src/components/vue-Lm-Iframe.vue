<template>
  <div class="proIframe">
    <div class="proIframeHeader">
      <img src="image/count_16x16.svg" class="icon_count_16x16" alt />
      <span>保费测算</span>
      <i class="el-icon-close" @click="$emit('closeIframeData')"></i>
    </div>
    <iframe :src="iframeUrl" frameborder="0" class="iframeWebView" width="100%" height="552px"></iframe>
  </div>
</template>
<script>
export default {
  props: {
    iframeUrl: {
      type: String,
      default: ""
    }
  },
  data() {
    return {};
  },
  methods: {
    getFormData(data) {
      if (data && data.source === "qixin") {
        this.$emit("listenChildEvent", this.formatQixinDataHandle(data.res));
        return;
      }
      if (data && data.source === "lima") {
        this.$emit("listenChildEvent", this.formatLiMaDataHandle(data.res));
        return;
      }
      if (data && data.source === "qx1") {
        this.$emit("listenChildEvent", this.formatQiXinDataHandle1(data.res));
        return;
      }
      return;
    },
    // 齐欣数据格式化
    formatQixinDataHandle(data) {
      if (data.indexOf("#|#") === -1) {
        alert("该产品暂不支持算费自动带入,请算费后手动输入");
        return;
      }
      let basicArr = data.split("#|#");
      let paramsArr = JSON.parse(basicArr[0])["genes"];
      let price = parseFloat(basicArr[1]) / 100;
      let formData = {};
      formData.yearlyAmount = price.toFixed("2"); //保费
      let insuredAmountFlag = true;
      paramsArr.forEach(item => {
        if (item.key === "" && item.protectItemId && insuredAmountFlag)
          (formData.insuredAmount = item.value), (insuredAmountFlag = false);
        // 保额
        if (item.key === "insureAgeLimit") formData.paymentPeriod = item.value;
        // 缴费年限
        if (item.key === "insurantDateLimit")
          formData.guaranteePeriod = item.value;
        //保障期间
      });
      return formData;
    },
    // 齐欣1
    formatQiXinDataHandle1(data) {
      let res = JSON.stringify(data);
      res = JSON.parse(res);
      return res;
    },

    // 力码数据格式化
    formatLiMaDataHandle(data) {
      let res = JSON.stringify(data);
      res = JSON.parse(res);
      let guaranteePeriod = res.guaranteePeriod;
      let insuredAmount = res.insuredAmount;
      let paymentPeriod = res.paymentPeriod;
      let insuredGender = res.insuredGender;
      if (guaranteePeriod.substr(0, 1) === "O") {
        res.guaranteePeriod = "保终身";
      } else if (guaranteePeriod.substr(0, 1) === "Y") {
        res.guaranteePeriod = "保" + guaranteePeriod.replace("Y", "") + "年";
      } else if (guaranteePeriod.substr(0, 1) === "A") {
        res.guaranteePeriod =
          "保至" + guaranteePeriod.replace("A", "") + "周岁";
      } else if (guaranteePeriod.substr(0, 1) === "D") {
        res.guaranteePeriod = "保" + guaranteePeriod.replace("D", "") + "天";
      } else if (guaranteePeriod.substr(0, 1) === "M") {
        res.guaranteePeriod = "保" + guaranteePeriod.replace("M", "") + "个月";
      } else {
        res.guaranteePeriod = "";
      }

      if (paymentPeriod.substr(0, 1) === "Y") {
        res.paymentPeriod = paymentPeriod.replace("Y", "") + "年交";
      } else if (paymentPeriod.substr(0, 1) === "A") {
        res.paymentPeriod = "交至" + paymentPeriod.replace("A", "") + "周岁";
      } else {
        res.paymentPeriod = "一次性缴纳";
      }

      if (insuredAmount > 10000) {
        res.insuredAmount = insuredAmount / 10000 + "万元";
      } else {
        if ((insuredAmount + "").indexOf("元") > -1) {
          res.insuredAmount = insuredAmount;
        } else {
          res.insuredAmount = insuredAmount + "元";
        }
      }

      if (insuredGender == 1) {
        res.insuredGender = "男";
      } else {
        res.insuredGender = "女";
      }
      return res;
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
          this.getFormData(str);
        },
        false
      );
    }
  },
  mounted() {
    this.getPostMessage();
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

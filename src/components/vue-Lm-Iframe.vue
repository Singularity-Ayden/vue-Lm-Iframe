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


      // 相伴一生 格式化   
      if (data && data.source === "xbhk") {
        let Data = JSON.parse(data.res.hkData);
        let insuredPayDateCode = "";
        let insuredPayDate = "";
        if (Data.insuredPayDate == "1000") {
          insuredPayDateCode = "SP";
          insuredPayDate = "终身";
        } else {
          insuredPayDateCode = "Y" + Data.insuredPayDate;
          insuredPayDate = Data.insuredPayDate + "年";
        }

        let insuredGender = "";
        let insuredGenderCode = "";

        if (Data.insuredSex == "1") {
          insuredGender = "女";
          insuredGenderCode = 0;
        } else {
          insuredGender = "男";
          insuredGenderCode = 1;
        }

        let insuredReceive = Data.insuredReceive + "年领";

        let hkData = {
          fee: parseInt(Data.insureNumber) * 1000,
          baseFactors: [
            {
              factorName: "保障期限",
              factorCode: "insurantDateLimit",
              factorValueName: "终生",
              factorValueCode: "O"
            },

            {
              factorName: "缴费年限",
              factorCode: "insureAgeLimit",
              factorValueName: insuredPayDate,
              factorValueCode: insuredPayDateCode
            },
            {
              factorName: "被保险人出生日期",
              factorCode: "insurantDate",
              factorValueName: Data.insuredBirth,
              factorValueCode: Data.insuredBirth
            },
            {
              factorName: "被保险人性别",
              factorCode: "sex",
              factorValueName: insuredGender,
              factorValueCode: insuredGenderCode
            },
            {
              factorName: "投保份数",
              factorCode: "insureNumber",
              factorValueName: Data.insureNumber,
              factorValueCode: Data.insureNumber
            },
            {
              factorName: "领取方式",
              factorCode: "insuredReceive",
              factorValueName: insuredReceive,
              factorValueCode: Data.insuredReceive
            }
          ],
          additionalFactors: []
        };

        this.$emit("listenChildEvent", hkData);
        return;
      }






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

        // code  字典

        let codeDic = [
          {
            id: "vesterSex",
            sel: [{ name: "男", value: 1 }, { name: "女", value: 0 }],
            des: "1:男,2:女"
          },
          {
            id: "sex",
            sel: [{ name: "男", value: 1 }, { name: "女", value: 0 }],
            des: "1:男,2:女"
          }
        ];

        sourceData.map(v => {
          if (v.name == "基本保额") {
            v.key = "insureAmount";
          }

          let code = codeDic.find(a => {
            return a.id == v.key;
          });
          let valuecode = "";
          let des = "";
          if (code != undefined) {
            des = code.des;
            let selBox = codeDic
              .find(a => {
                return a.id == v.key;
              })
              .sel.find((b, c) => {
                return b.name == v.value;
              });
            if (selBox != undefined) {
              valuecode = selBox.value;
            }
          } else {
            valuecode = v.value;
            des = "---";
          }

          // 保障期间 格式化

          if (v.key == "insurantDateLimit") {
            if (valuecode.substr(0, 1) == "至") {
              valuecode = valuecode.replace(/至/g, "A").replace(/岁/g, "");
            } else if (valuecode == "终身") {
              valuecode = "O";
            }

            des = "O:终身,A70:至70岁";
          }

          if (v.key == "paymentType") {
            if (valuecode == "年交") {
              valuecode = "Y";
            }
            des = "D  按天缴费,Y  年缴,M  月缴,SP 趸缴";
          }

          if (v.key == "insureAgeLimit") {
            valuecode = valuecode.replace(/年/g, "");
            des = "单位：年";
          }

          if (v.key == "insureAmount") {
            valuecode = valuecode.replace(/万元/g, "");
            des = "单位：万元";
          }

          if (v.key == "additionalInsureAgeLimit") {
            if (valuecode == "投保") {
              valuecode = 1;
            } else {
              valuecode = 0;
            }

            des = "1 投保，0 不投保";
          }

          if (v.key == "insureForSelf") {
            if (valuecode == "是") {
              valuecode = 1;
            } else {
              valuecode = 0;
            }

            des = "1 是，0 否";
          }

          if (
            v.key == "premiumExemption" ||
            v.key == "malignantTumorSecondaryCompensation"
          ) {
            if (valuecode == "含") {
              valuecode = 1;
            } else {
              valuecode = 0;
            }

            des = "1 含，0 不含";
          }

          let obj = {
            factorName: v.name,
            factorCode: v.key,
            factorValueName: v.value,
            factorValueCode: valuecode,
            des: des
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

    // 力码数据格式化
    formatLiMaDataHandle(data) {
      let res = JSON.stringify(data);
      res = JSON.parse(res);
      let guaranteePeriod = res.guaranteePeriod;
      let insuredAmount = res.insuredAmount;
      let paymentPeriod = res.paymentPeriod;
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


      let  paymentPeriodValue = ''
      if(res.paymentPeriod == '一次性缴纳'){
        paymentPeriodValue = 'SP'

      }else{
         paymentPeriodValue = res.paymentPeriod.replace(/年交/g,'')
      }


      let guaranteePeriodCode = ''
      if(res.guaranteePeriod=='保终身'){
        guaranteePeriodCode = 'O'
      }

      let insureAmountCode = res.insuredAmount.replace(/元/g,'')

      let insuredGenderCode = '' 
      let insuredGenderValue = ''
      if(res.insuredGender == 1){
        insuredGenderValue = '男'
        insuredGenderCode = 1
      }else{
         insuredGenderValue = '女'
        insuredGenderCode = 0
      }
     
        

      let xbObj = {
        fee: res.yearlyAmount,
        baseFactors: [
          {
            factorName: "保障期限",
            factorCode: "insurantDateLimit",
            factorValueName: res.guaranteePeriod,
            factorValueCode: guaranteePeriodCode
          },
          {
            factorName: "基本保额",
            factorCode: "insureAmount",
            factorValueName: res.insuredAmount,
            factorValueCode: insureAmountCode
          },
          {
            factorName: "缴费年限",
            factorCode: "insureAgeLimit",
            factorValueName: res.paymentPeriod,
            factorValueCode: paymentPeriodValue
          },
          {
            factorName: "被保险人出生日期",
            factorCode: "insurantDate",
            factorValueName: res.insuredBirthday,
            factorValueCode: res.insuredBirthday
          },
          {
            factorName: "被保险人性别",
            factorCode: "sex",
            factorValueName: insuredGenderValue,
            factorValueCode: insuredGenderCode
          }
        ],
        additionalFactors: []
      };

      return xbObj;
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

          if (str.source == "xbqx") {
            that.$emit("closeIframeData");
          }

          if (str.source == "xbhk") {
            that.$emit("closeIframeData");
          }

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

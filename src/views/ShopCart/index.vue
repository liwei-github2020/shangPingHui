<template>
  <div class="cart">
    <h4>全部商品</h4>
    <div class="cart-main">
      <div class="cart-th">
        <div class="cart-th1">全部</div>
        <div class="cart-th2">商品</div>
        <div class="cart-th3">单价（元）</div>
        <div class="cart-th4">数量</div>
        <div class="cart-th5">小计（元）</div>
        <div class="cart-th6">操作</div>
      </div>
      <div class="cart-body">
        <ul class="cart-list" v-for="item in cartInfoList" :key="item.id">
          <li class="cart-list-con1">
            <input type="checkbox" name="chk_list" id="" value="" :checked="item.isChecked == 1"
              @change="handleChecked(item.skuId, $event.target.checked)" />
          </li>
          <li class="cart-list-con2">
            <img :src="item.imgUrl" />
            <div class="item-msg">
              {{ item.skuName }}
            </div>
          </li>

          <li class="cart-list-con4">
            <span class="price">{{ item.cartPrice }}.00</span>
          </li>
          <li class="cart-list-con5">
            <a href="javascript:void(0)" class="mins" @click="handler('dec', -1, item)">-</a>
            <input autocomplete="off" type="text" :value="item.skuNum" minnum="1" class="itxt"
              @change="handler('change', $event.target.value, item)" />
            <a href="javascript:void(0)" class="plus" @click="handler('add', 1, item)">+</a>
          </li>
          <li class="cart-list-con6">
            <span class="sum">{{ item.skuNum * item.skuPrice }}.00</span>
          </li>
          <li class="cart-list-con7">
            <a href="#none" class="sindelet" @click="deleteShopCartList(item)">删除</a>
            <br />
            <a href="#none">移到收藏</a>
          </li>
        </ul>
      </div>
    </div>
    <div class="cart-tool">
      <div class="select-all">
        <input class="chooseAll" type="checkbox" :checked="isAllChecked && cartInfoListLength > 0"
          @change='checkAll($event.target.checked)' />
        <span>全选</span>
      </div>
      <div class="option">
        <a @click="deleteAllCheckedCart">删除选中的商品</a>
        <a href="#none">移到我的关注</a>
        <a href="#none">清除下柜商品</a>
      </div>
      <div class="money-box">
        <div class="chosed">
          已选择 <span>{{ selectedShops }}</span>件商品
        </div>
        <div class="sumprice">
          <em>总价（不含运费） ：</em>
          <i class="summoney">{{ summoney }}</i>
        </div>
        <div class="sumbtn">
          <a class="sum-btn" href="###" target="_blank">结算</a>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { throttle } from "@/utils/func";
import { mapGetters } from "vuex";
export default {
  name: "ShopCart",
  data() {
    return {};
  },
  beforeMount() {
    this.getShopCartList();
  },
  computed: {
    ...mapGetters(["cartInfoList"]),

    cartInfoListLength() {
      return this.$store.getters.cartInfoList.length;
    },
    isAllChecked() {
      return this.cartInfoList.every((item) => item.isChecked === 1);
    },
    summoney() {
      // let sum = 0;
      // this.cartInfoList.forEach((item) => {
      //   if (item.isChecked) {
      //     sum += item.skuNum * item.skuPrice;
      //   }
      // });
      // return sum;
      let sum = this.cartInfoList.reduce((pre, current) => {
        // console.log(current);
        if (current.isChecked) {
          return pre + current.skuNum * current.cartPrice;
        }
        return pre;
      }, 0);
      return sum;
    },
    selectedShops() {
      return this.cartInfoList.reduce(
        (pre, current) => (current.isChecked ? pre + 1 : pre),
        0
      );
    },
  },
  methods: {
    getShopCartList() {
      this.$store.dispatch("getShopCartList");
    },
    handler: throttle(async function (type, disNum, cart) {
      switch (type) {
        case "add":
          disNum = 1;
          break;
        case "dec":
          disNum = cart.skuNum > 1 ? -1 : 0;
          break;
        case "change":
          if (isNaN(disNum) || disNum <= 1) {
            disNum = 0;
          } else {
            disNum = parseInt(disNum) - cart.skuNum;
          }
          break;
      }
      try {
        await this.$store.dispatch("addShopCart", {
          skuId: cart.skuId,
          skuNum: disNum,
        });
        this.getShopCartList();
      } catch (error) {
        console.log(error);
      }
    }, 500),
    async deleteShopCartList(cart) {
      try {
        await this.$store.dispatch("deleteShopCartList", cart);
        this.getShopCartList();
      } catch (error) {
        console.log(error);
      }
    },
    async handleChecked(skuId, event) {
      console.log(skuId);
      let checked = event ? 1 : 0;
      try {
        await this.$store.dispatch("updateShopCartList", {
          skuId,
          checked,
        });
        this.getShopCartList();
      } catch (error) {
        alert(error);
      }

    },
    async deleteAllCheckedCart() {
      try {
        await this.$store.dispatch("deleteAllCheckedCart");
        this.getShopCartList();
      } catch (error) {
        console.log(error)
      }
    },
    async checkAll(checked) {
      let isChecked = checked ? 1 : 0;
      try {
        await this.$store.dispatch("checkAll", isChecked);
        this.getShopCartList();
      } catch (error) {
        console.log(error);
      }
    }
  },
};
</script>

<style lang="less" scoped>
.cart {
  width: 1200px;
  margin: 0 auto;

  h4 {
    margin: 9px 0;
    font-size: 14px;
    line-height: 21px;
  }

  .cart-main {
    .cart-th {
      background: #f5f5f5;
      border: 1px solid #ddd;
      padding: 10px;
      overflow: hidden;

      &>div {
        float: left;
      }

      .cart-th1 {
        width: 25%;

        input {
          vertical-align: middle;
        }

        span {
          vertical-align: middle;
        }
      }

      .cart-th2 {
        width: 25%;
      }

      .cart-th3,
      .cart-th4,
      .cart-th5,
      .cart-th6 {
        width: 12.5%;
      }
    }

    .cart-body {
      margin: 15px 0;
      border: 1px solid #ddd;

      .cart-list {
        padding: 10px;
        border-bottom: 1px solid #ddd;
        overflow: hidden;

        &>li {
          float: left;
        }

        .cart-list-con1 {
          width: 15%;
        }

        .cart-list-con2 {
          width: 35%;

          img {
            width: 82px;
            height: 82px;
            float: left;
          }

          .item-msg {
            float: left;
            width: 150px;
            margin: 0 10px;
            line-height: 18px;
          }
        }

        .cart-list-con4 {
          width: 10%;
        }

        .cart-list-con5 {
          width: 15%;

          .mins {
            border: 1px solid #ddd;
            border-right: 0;
            float: left;
            color: #666;
            width: 6px;
            text-align: center;
            padding: 8px;
          }

          input {
            border: 1px solid #ddd;
            width: 40px;
            height: 33px;
            float: left;
            text-align: center;
            font-size: 14px;
          }

          .plus {
            border: 1px solid #ddd;
            border-left: 0;
            float: left;
            color: #666;
            width: 6px;
            text-align: center;
            padding: 8px;
          }
        }

        .cart-list-con6 {
          width: 10%;

          .sum {
            font-size: 16px;
          }
        }

        .cart-list-con7 {
          width: 15%;

          a {
            color: #666;
          }
        }
      }
    }
  }

  .cart-tool {
    overflow: hidden;
    margin-bottom: 20px;
    border: 1px solid #ddd;

    .select-all {
      padding: 10px;
      overflow: hidden;
      float: left;

      span {
        vertical-align: middle;
      }

      input {
        vertical-align: middle;
      }
    }

    .option {
      padding: 10px;
      overflow: hidden;
      float: left;

      a {
        float: left;
        padding: 0 10px;
        color: #666;
      }
    }

    .money-box {
      float: right;

      .chosed {
        line-height: 26px;
        float: left;
        padding: 0 10px;
      }

      .sumprice {
        width: 200px;
        line-height: 22px;
        float: left;
        padding: 0 10px;

        .summoney {
          color: #c81623;
          font-size: 16px;
        }
      }

      .sumbtn {
        float: right;

        a {
          display: block;
          position: relative;
          width: 96px;
          height: 52px;
          line-height: 52px;
          color: #fff;
          text-align: center;
          font-size: 18px;
          font-family: "Microsoft YaHei";
          background: #e1251b;
          overflow: hidden;
        }
      }
    }
  }
}
</style>
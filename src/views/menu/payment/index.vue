<template>
  <div class="payment-page">
    <v-row no-gutters v-if="selected != null">
      <v-col cols="12" class="d-flex align-start">
        <span class="ml-2">
          <h2>ซื้อเหรียญ</h2>
        </span>
      </v-col>
    </v-row>
    <v-row no-gutters justify="center" class="px-3">
      <v-col cols="12" xs="12" sm="8" md="6" lg="4">
        <v-row dense class="mt-3">
          <v-col cols="12">
            <v-menu bottom offset-y>
              <template v-slot:activator="{ on, attrs }">
                <div class="d-flex align-center">
                  <v-btn text class="pl-0" v-bind="attrs" v-on="on">
                    <v-img
                      height="30px"
                      max-width="30px"
                      contain
                      :src="selected.logoURI"
                    ></v-img>
                    <span class="ml-2">{{ selected.symbol }}</span>
                    <v-icon>arrow_drop_down</v-icon>
                  </v-btn>
                  <v-spacer></v-spacer>
                  <div class="text-right d-block mt-1">
                    <span class="sub-text">
                      1 {{ selected.symbol }} = {{ selected.price }} THB
                    </span>
                  </div>
                </div>
              </template>
              <v-list>
                <v-list-item
                  v-for="(token, i) in tokenListWithoutSelect"
                  :key="i"
                  link
                  @click="selected = token"
                >
                  <v-list-item-title>
                    <div class="d-flex align-center">
                      <v-img
                        height="25px"
                        max-width="25px"
                        contain
                        :src="token.logoURI"
                      ></v-img>
                      <span class="ml-3" style="font-size: 13px">{{
                        token.symbol
                      }}</span>
                    </div>
                  </v-list-item-title>
                </v-list-item>
              </v-list>
            </v-menu>
          </v-col>
          <v-col cols="12">
            <v-text-field
              background-color="#c71e2b"
              dark
              class="amount-input"
              hide-details
              type="number"
              outlined
              v-model.number="amount"
            >
            </v-text-field>
            <div class="text-center mt-2">
              <h3>จำนวน {{ sum_price }} THB</h3>
              <span class="error--text" style="font-size: 12px"
                >ซื้อขั้นต่ำ 20 บาท</span
              >
            </div>
          </v-col>
          <v-col cols="12" class="mt-3">
            <div>
              <h4>Payment Method</h4>
            </div>
          </v-col>
          <v-col cols="12">
            <v-expansion-panels>
              <v-expansion-panel @change="channelPayment('payplus_kbank')">
                <v-expansion-panel-header>
                  <div class="d-flex align-center pr-3">
                    <v-img
                      :src="'/image/kplus.png'"
                      max-width="30px"
                      height="30px"
                      contain
                    ></v-img>
                    <v-spacer></v-spacer>
                    <span> Mobile Payment </span>
                  </div>
                </v-expansion-panel-header>
                <v-expansion-panel-content>
                  <v-row no-gutters class="pa-3" justify="center">
                    <v-form
                      ref="formKplus"
                      v-model="valid"
                      lazy-validation
                      class="text-center"
                    >
                      <v-text-field
                        required
                        :rules="phoneRules"
                        type="number"
                        v-model.trim="form.PhoneNumber"
                        color="none"
                        outlined
                        dense
                        height="40px"
                        label="เบอร์โทรศัพท์ K PLUS"
                        :counter="10"
                      ></v-text-field>
                      <v-btn
                        color="#c71e2b"
                        class="mt-3"
                        :disabled="sum_price < 20"
                        @click="pay"
                      >
                        <span class="white--text"> จ่ายด้วย PayPlus </span>
                      </v-btn>
                    </v-form>
                  </v-row>
                </v-expansion-panel-content>
              </v-expansion-panel>

              <v-expansion-panel @change="channelPayment('bank_qrcode')">
                <v-expansion-panel-header>
                  <div class="d-flex align-center pr-3">
                    <v-icon>qr_code</v-icon>
                    <v-spacer></v-spacer>
                    <span> QR Code Payment </span>
                  </div>
                </v-expansion-panel-header>
                <v-expansion-panel-content>
                  <v-row no-gutters class="pa-3" justify="center">
                    <v-btn
                      color="#c71e2b"
                      :disabled="sum_price < 20"
                      @click="pay"
                    >
                      <span class="white--text"> จ่ายด้วย QR Code </span>
                    </v-btn>
                  </v-row>
                </v-expansion-panel-content>
              </v-expansion-panel>
            </v-expansion-panels>
            <div>
              <form
                ref="paymentMobile"
                method="POST"
                action="https://sandbox-cdnv3.chillpay.co/Payment"
              >
                <input
                  type="hidden"
                  name="MerchantCode"
                  v-model="form.MerchantCode"
                />
                <input type="hidden" name="OrderNo" v-model="form.OrderNo" />
                <input
                  type="hidden"
                  name="CustomerId"
                  v-model="form.CustomerId"
                />
                <input type="hidden" name="Amount" v-model="form.Amount" />
                <input type="hidden" name="PhoneNumber" v-model="form.PhoneNumber" />
                <input
                  type="hidden"
                  name="Description"
                  v-model="form.Description"
                />
                <input
                  type="hidden"
                  name="ChannelCode"
                  v-model="form.ChannelCode"
                />
                <input type="hidden" name="Currency" v-model="form.Currency" />
                <input type="hidden" name="LangCode" v-model="form.LangCode" />
                <input type="hidden" name="RouteNo" v-model="form.RouteNo" />
                <input
                  type="hidden"
                  name="IPAddress"
                  v-model="form.IPAddress"
                />
                <input type="hidden" name="APIKey" v-model="form.ApiKey" />
                <input
                  name="ProductImageUrl"
                  type="hidden"
                  v-model="form.ProductImageUrl"
                />
                <input type="hidden" name="CheckSum" v-model="form.CheckSum" />
              </form>

              <form
                ref="paymentQR"
                method="POST"
                action="https://sandbox-cdnv3.chillpay.co/Payment"
              >
                <input
                  type="hidden"
                  name="MerchantCode"
                  v-model="form.MerchantCode"
                />
                <input type="hidden" name="OrderNo" v-model="form.OrderNo" />
                <input
                  type="hidden"
                  name="CustomerId"
                  v-model="form.CustomerId"
                />
                <input type="hidden" name="Amount" v-model="form.Amount" />
                <input
                  type="hidden"
                  name="Description"
                  v-model="form.Description"
                />
                <input
                  type="hidden"
                  name="ChannelCode"
                  v-model="form.ChannelCode"
                />
                <input type="hidden" name="Currency" v-model="form.Currency" />
                <input type="hidden" name="LangCode" v-model="form.LangCode" />
                <input type="hidden" name="RouteNo" v-model="form.RouteNo" />
                <input
                  type="hidden"
                  name="IPAddress"
                  v-model="form.IPAddress"
                />
                <input type="hidden" name="APIKey" v-model="form.ApiKey" />
                <input
                  name="ProductImageUrl"
                  type="hidden"
                  v-model="form.ProductImageUrl"
                />
                <input type="hidden" name="CheckSum" v-model="form.CheckSum" />
              </form>
            </div>
          </v-col>
        </v-row>
      </v-col>
    </v-row>
    <bottom-nav />
  </div>
</template>

<script>
import layout from "../../../components/layout.vue";
import bottomNav from "../../../components/BottomNav.vue";
export default {
  name: "Payment",
  components: {
    layout: layout,
    "bottom-nav": bottomNav,
  },
  computed: {
    sum_price() {
      try {
        return this.amount * this.selected.price;
      } catch (err) {
        return 0;
      }
    },
    tokenList() {
      return this.$store.state.auth.tokenList;
    },
    tokenListWithoutSelect() {
      try {
        let index = this.tokenList.findIndex((q) => q == this.selected);
        if (index >= 0) {
          let array = [];
          for (let i in this.tokenList) {
            if (i != index) {
              array.push(this.tokenList[i]);
            }
          }
          return array;
        } else {
          return this.tokenList;
        }
      } catch (err) {
        console.log(err);
        return this.tokenList;
      }
    },
    me() {
      return this.$store.state.auth.me;
    },
  },
  data() {
    return {
      phoneRules: [
        (v) => !!v || "กรุณากรอกเบอร์โทรศํพท์",
        (v) => (v && v.length <= 10) || "เบอร์โทรศัพท์ 10 หลัก",
      ],
      valid: true,
      amount: 0,
      selected: null,
      form: {
        MerchantCode: "M031732",
        OrderNo: "00001",
        CustomerId: null,
        Amount: null,
        PhoneNumber: null,
        Description: null,
        ChannelCode: null,
        Currency: 764,
        LangCode: "TH",
        RouteNo: "1",
        IPAddress: "192.168.1.7:8080",
        ApiKey:
          "AqdoHRkWVIAi27yrsUu8WANXtWIM026PL0Mz8F9Vg7NCLEp1RsxRSv4yhV4RZgSO",
        ProductImageUrl: null,
        CheckSum: null,
      },
    };
  },
  created() {
    this.selected = this.tokenList[0];
    this.form.CustomerId = this.me.uid;
  },
  methods: {
    channelPayment(ch) {
      this.form.ChannelCode = ch;
    },
    async checkSumEncryp() {
      let md5key =
        "Z5zZf38usZKeeVFjwvbwshr7Aq7t18QWeZI9vLMzYcK2FG5vOMI47o36O7ppBxIXKbUyWjjG8q9y3a8QtpM1GRWQnGBwaxPob0cGMhxAQgU1Q5inMp8cRujQKbiFDnejKMFf2qb44ywzABH5ESUAhWBSgT1JJiXZdzT5k";
      var key = Object.keys(this.form);
      let txt = "";
      key.forEach((element) => {
        if (element != "CheckSum") {
          txt += this.form[element];
        }
      });
      txt += md5key;
      let cryp = await CryptoJS.MD5(txt);
      this.form.CheckSum = await cryp.toString();
    },
    async pay() {
      this.form.Amount = this.sum_price + "00";
      this.form.Description = this.selected.address;
      this.form.ProductImageUrl = this.selected.logoURI;
      if (this.form.ChannelCode == "payplus_kbank") {
        if (this.$refs.formKplus) {
          await this.checkSumEncryp();
          await this.$refs.paymentMobile.submit();
        }
      } else if (this.form.ChannelCode == "bank_qrcode") {
        delete this.form.PhoneNumber;
        await this.checkSumEncryp();
        await this.$refs.paymentQR.submit();
      }
    },
  },
};
</script>

<style lang="scss">
.payment-page {
  min-height: 100vh;
  height: 100%;
  background-color: white;
  padding: 24px 8px 50px 8px;

  .sub-text {
    color: #adadad;
  }

  .amount-input {
    font-size: 21px;
  }
}
</style>
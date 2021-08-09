<template>
  <div class="transfer-page">
    <v-row class="pt-8">
      <v-col cols="12" class="pa-0 d-flex" align-self="center">
          <v-img
            src="../../../assets/wallet-icon.svg"
            height="60px"
            width="60px"
            contain
          ></v-img>
      </v-col>
    </v-row>
    <layout
      :header="tokenByName.name"
      @onBack="$router.push('/')"
      :class="{ blur: showPin }"
      :receive="true"
      @receive="dialog = true"
    >
      <template v-slot:content>
        <v-col cols="12" xs="12" sm="8" md="6" lg="4">
          <v-row no-gutters class="pb-5">
            <v-col cols="12">
              <v-card class="box-detail" elevation="0">
                <v-toolbar class="custom-toolbar" color="#FFFFFF">
                  <v-row>
                    <v-col cols="6" class="text-right" align-self="center">
                      <v-row>
                        <v-col
                          cols="4"
                          class="ma-0 pa-0 pl-5"
                          align-self="center"
                        >
                          <div class="text-right">
                            <v-img
                              :src="tokenByName.logoURI"
                              height="40px"
                              width="40px"
                              contain
                            ></v-img>
                          </div>
                        </v-col>
                        <v-col
                          cols="8"
                          class="text-start ma-0 pa-3 pl-5"
                          align-self="center"
                        >
                          <v-row>
                            {{ tokenByName.name }}
                          </v-row>
                          <v-row style="font-size: 10px" class="grey--text">
                            1 {{ tokenByName.name }} = {{ bathRate }} ฿
                          </v-row>
                        </v-col>
                      </v-row>
                    </v-col>
                    <v-col cols="6" class="text-right" align-self="center">
                      <v-row class="text-right">
                        <v-col class="ma-0 pa-0 pr-4">
                          {{ parseUtillETHtoShow(tokenByName.balance) }}
                        </v-col>
                      </v-row>
                      <v-row class="text-right" style="font-size: 10px">
                        <v-col class="ma-0 pa-0 pr-4 grey--text">
                          = {{ bathRate * Number(parseUtillETHtoShow(tokenByName.balance)) }} ฿
                        </v-col>
                      </v-row>
                    </v-col>
                  </v-row>
                </v-toolbar>
                <div class="pa-4 mt-4">
                  <v-form
                    ref="form"
                    v-model="valid"
                    lazy-validation
                    class="text-center"
                  >
                    <v-row
                      style="background-color: white"
                      class="mr-1 ml-1 mb-5 pl-1 pr-3 pt-2 pb-2 rounded-lg font-weight-bold custom-text-field"
                    >
                      <v-col cols="10" class="pa-0 ma-0">
                        <v-text-field
                          solo
                          flat
                          background-color="white"
                          v-model="form.to"
                          :rules="itemRules"
                          label="หมายเลขที่อยู่กระเป๋า"
                          hide-details="auto"
                          required
                        >
                        </v-text-field
                      ></v-col>
                      <v-col cols="2" align-self="center" class="pa-0">
                        <v-btn text
                          ><v-img
                            src="../../../assets/qr-code-button-icon.svg"
                          ></v-img
                        ></v-btn>
                      </v-col>
                    </v-row>

                    <!-- Number Coin -->
                    <v-row
                      style="background-color: white"
                      class="mr-1 ml-1 mb-5 pl-1 pr-3 pt-2 pb-2 rounded-lg font-weight-bold custom-text-field"
                    >
                      <v-col cols="10" class="ma-0 pa-0">
                        <v-text-field
                          type="number"
                          solo
                          flat
                          background-color="white"
                          v-model="form.amount"
                          :rules="[
                            (v) => !!v || 'กรุณากรอกข้อมูล',
                            (v) => parseFloat(v) > 0 || 'จำนวนต้องมากกว่า 0',
                            (v) =>
                              parseFloat(v) <=
                                parseFloat(
                                  parseUtillETH(tokenByName.balance)
                                ) || 'จำนวนไม่ถูกต้อง',
                          ]"
                          hide-details="auto"
                          label="จำนวนเหรียญ"
                          required
                        >
                        </v-text-field>
                      </v-col>
                      <v-col cols="2" class="ma-0 pa-0 grey--text" align-self="center">
                        {{ tokenByName.name }}</v-col
                      >
                    </v-row>
                    <v-row>
                      <v-col cols="6">
                        <v-btn
                          color="#000000"
                          class="mr-2 rounded-xl pa-6"
                          width="100%"
                          @click="reset"
                        >
                          <span class="grey--text text--lighten-1"
                            >เคลียร์</span
                          >
                        </v-btn>
                      </v-col>
                      <v-col cols="6">
                        <v-btn
                          :disabled="!valid"
                          color="#000000"
                          class="ml-2 rounded-xl pa-6"
                          width="100%"
                          @click="validate"
                        >
                          <span class="white--text"> โอนเหรียญ </span>
                        </v-btn>
                      </v-col>
                    </v-row>
                  </v-form>
                </div>
              </v-card>
            </v-col>
            <!-- History Zone -->
            <v-col cols="12" class="mt-16" v-show="showPage">
              <div class="d-flex align-center pl-2 font-weight-bold">
                <v-icon large> history </v-icon>
                <span class="ml-1 ">ประวัติการทำรายการ</span>
              </div>
              <v-card class="box-detail-history mt-2" elevation="0">
                <div v-if="historyByToken.length > 0">
                  <history-item
                    v-for="(history, i) in historyByToken"
                    :key="'historyByToken' + i"
                    :hash="history.hash"
                    :from="history.from"
                    :timeStamp="history.timeStamp"
                    :tokenSymbol="history.tokenSymbol"
                    :value="history.value"
                    :lastChild="i >= historyByToken.length - 1"
                  />
                </div>
                <div
                  v-else
                  class="d-flex align-center justify-center"
                  style="height: 100px"
                >
                  ไม่มีข้อมูล...
                </div>
              </v-card>
            </v-col>
          </v-row>
        </v-col>
      </template>
    </layout>
    <pin-pad
      v-if="showPin"
      header="กรุณาใส่รหัส"
      :verify="true"
      :backward="true"
      @goback="showPin = false"
      @verifyPin="sendToken"
    />
    <v-dialog max-width="350" v-model="dialog">
      <my-qrcode
        :value="
          JSON.stringify({
            type: 'transfer',
            to: ethereumAddress,
            contractAddress: tokenByName.address,
          })
        "
      />
    </v-dialog>

    <!-- <v-btn fab color="#c71e2b" large style="position:fixed;right:10px;bottom:10px;">
      <v-img :src="'/image/buyicon.png'" height="50px" width="50px" contain></v-img>
    </v-btn> -->
  </div>
</template>

<script>
import layout from "../../../components/layout.vue";
import pidPad from "../../../components/pinPad.vue";
import qrcode from "../../../components/qrcode.vue";
import historyItem from "../../../components/historyItem.vue";

export default {
  name: "Token",
  components: {
    layout: layout,
    "pin-pad": pidPad,
    "my-qrcode": qrcode,
    "history-item": historyItem,
  },
  data() {
    return {
      dialog: false,
      showPin: false,
      showPage: false,
      valid: true,
      itemRules: [
        (v) => !!v || "กรุณากรอกข้อมูล",
        (v) => /^0x[a-fA-F0-9]{40}$/g.test(v) || "กรุณากรอกข้อมูลให้ถูกต้อง",
      ],
      form: {
        to: null,
        amount: null,
        text: null,
      },
      transferInstance: null,
      wsProvider: null,
      contract: null,
      bathRate: 10,
    };
  },
  computed: {
    tokenByName() {
      return this.$store.getters.tokenByName(this.$route.params.token);
    },
    historyByToken() {
      return this.$store.getters.historyByToken(this.tokenByName.address);
    },
  },
  methods: {
    async sendToken(verify) {
      if (verify.status) {
        this.showPin = false;
        try {
          await this.app_loading(true);
          let wallet = await decrypt(
            localStorage.getItem("encypt_string_mpv"),
            verify.password
          );
          let wallet_json = JSON.parse(wallet);
          let privateKey = await wallet_json.privateKey;
          const provider = await new this.$ethers.providers.JsonRpcProvider(
            "https://rpc.tbwg.io"
          );
          const walletSigner = await new this.$ethers.Wallet(
            privateKey,
            provider
          );
          let amount = await this.$ethers.utils.parseUnits(
            this.form.amount,
            18
          );
          let to_address = await this.form.to;
          let gas_limit = "0x100000";
          await this.app_loading(true);
          let currentGasPrice = await provider.getGasPrice();
          let gas_price = await this.$ethers.utils.hexlify(
            parseInt(currentGasPrice)
          );
          if (this.tokenByName.address != "mainnet") {
            try {
              // general token send
              let contract = await new this.$ethers.Contract(
                this.tokenByName.address,
                this.$abi,
                walletSigner
              );
              let transaction = await contract.transfer(to_address, amount);
              await transaction.wait();
              await this.transferSuccess();
            } catch (err) {
              throw err;
            }
          } else {
            // ether send
            try {
              const tx = {
                from: this.ethereumAddress,
                to: to_address,
                value: amount,
                nonce: provider.getTransactionCount(
                  this.ethereumAddress,
                  "latest"
                ),
                gasLimit: this.$ethers.utils.hexlify(gas_limit), // 100000
                gasPrice: gas_price,
              };
              let transaction = await walletSigner.sendTransaction(tx);
              await transaction.wait();
              await this.transferSuccess();
            } catch (err) {
              throw err;
            }
          }
          this.writeCookies(verify.password);
        } catch (err) {
          console.log(err);
          this.transferError();
        }
      }
    },
    async validate() {
      let valid = await this.$refs.form.validate();
      let pin = this.$cookies.get("pin_mpv");
      if (valid) {
        if (pin == null) {
          this.showPin = true;
        } else {
          const vm = this;
          this.alert_show({
            type: "confirm",
            header: "กรุณายืนยัน",
            title: "ท่านต้องการที่จะทำรายการใช่หรือไหม ?",
          }).then((res) => {
            if (res) {
              let verify = {
                status: true,
                password: pin,
              };
              vm.sendToken(verify);
            }
          });
        }
      }
    },
    async writeCookies(pin) {
      this.$cookies.set("pin_mpv", pin, "15min");
    },
    async transferSuccess() {
      // await this.$store.dispatch("getHistory");
      // await this.$store.dispatch("getBalance");
      await this.app_loading(false);
      this.reset();
      await this.alert_show({
        type: "success",
        header: "สำเร็จ",
        title: "โอนสำเร็จ",
      });
    },
    async transferError() {
      this.app_loading(false);
      this.alert_show({
        type: "error",
        title: "การทำรายการล้มเหลว",
        header: "ล้มเหลว !",
      });
    },
    reset() {
      this.$refs.form.reset();
      this.$refs.form.resetValidation();
    },
  },
  async mounted() {
    if (this.$route.query.to != null) {
      this.form.to = this.$route.query.to;
    }
    this.$nextTick(() => {
      const vm = this;
      var init = function () {
        vm.wsProvider = new vm.$ethers.providers.WebSocketProvider(
          "wss://ws.xchain.asia"
        );

        if (vm.tokenByName.address != "mainnet") {
          vm.contract = new vm.$ethers.Contract(
            vm.tokenByName.address,
            vm.$abi,
            vm.wsProvider
          );
          vm.contract.on("*", async (res) => {
            console.log(res);
            let to = await res.args.to;
            let amount = await res.args.tokens.toString();
            setTimeout(() => {
              vm.$store.dispatch("getHistory");
              vm.$store.dispatch("getBalance");
            }, 2000);
            if (
              res.event == "Transfer" &&
              String(to).toLowerCase() ==
                String(vm.ethereumAddress).toLowerCase()
            ) {
              await vm.$store.commit("SET_ALERT_TOAST", {
                status: true,
                tokenName: vm.tokenByName.name,
                tokenSymbol: vm.tokenByName.symbol,
                amount: amount,
              });
            }
          });
        } else {
          vm.wsProvider.on("pending", async (tx) => {
            let getTx = await vm.wsProvider.getTransaction(tx);
            await vm.wsProvider.once(tx, async (transaction) => {
              if (
                String(transaction.to).toLowerCase() ==
                  String(vm.ethereumAddress).toLowerCase() ||
                String(transaction.from).toLowerCase() ==
                  String(vm.ethereumAddress).toLowerCase()
              ) {
                setTimeout(() => {
                  vm.$store.dispatch("getHistory");
                  vm.$store.dispatch("getBalance");
                }, 2000);

                if (
                  String(transaction.to).toLowerCase() ==
                  String(vm.ethereumAddress).toLowerCase()
                ) {
                  let amount = await getTx.value.toString();
                  await vm.$store.commit("SET_ALERT_TOAST", {
                    status: true,
                    tokenName: "XTH",
                    tokenSymbol: "XTH",
                    amount: amount,
                  });
                }
              }
            });
          });
        }

        vm.wsProvider.on("error", async () => {
          console.log(`Unable to connect to ${ep.subdomain} retrying in 3s...`);
          setTimeout(init, 3000);
        });
        vm.wsProvider.on("close", async (code) => {
          console.log(
            `Connection lost with code ${code}! Attempting reconnect in 3s...`
          );
          vm.wsProvider.terminate();
          setTimeout(init, 3000);
        });
      };
      init();
    });
  },
  async created() {
    this.app_loading(true);
    await this.$store.dispatch("getHistory");
    this.app_loading(false);
    this.showPage = true;
  },
  beforeDestroy() {
    this.wsProvider.off();
  },
};

function decrypt(message = "", key = "") {
  var code = CryptoJS.AES.decrypt(message, key);
  var decryptedMessage = code.toString(CryptoJS.enc.Utf8);

  return decryptedMessage;
}
</script>

<style lang="scss">
  .transfer-page {
    .box-detail {
      box-shadow: 0px 0px 5px #CDE2F0 !important;
      background: radial-gradient(at left 50%,rgba(254,254,254,1) 0%, rgba(233,240,245,1) 74%, rgba(226,235,242,1) 100%);
    }
    .box-detail-history {
      box-shadow: 0px 0px 5px #CDE2F0 !important;
    }
    .custom-toolbar {
      /* background-color: #fff !important; */
      box-shadow: 0px 0px 5px #CDE2F0 !important;
    }
    .custom-text-field {
      border-radius: 9px;
      box-shadow: 0px 0px 5px #CDE2F0;
    }
  }
</style>

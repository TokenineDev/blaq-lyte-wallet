<template>
  <div class="bar-me">
    <v-app-bar class="mt-3" color="transparent" max-width="500px" flat height="64px">
      <v-img
        :src="require('/src/assets/wallet-icon.svg')"
        height="2.5rem"
        contain
      ></v-img>
      <div class="address">
        <input
          type="text"
          class="input-pvkey pa-4 pr-5"
          ref="ethWallet"
          :value="ethereumAddress"
          @click="CopyEth"
          readonly
        />
      </div>
      <v-menu bottom offset-y>
        <template v-slot:activator="{ on, attrs }">
          <v-avatar class="avatar" color="indigo" v-bind="attrs" v-on="on">
            <v-img
              :src="require('/src/assets/default-user-icon.svg')"
            ></v-img>
          </v-avatar>
        </template>
        <v-list>
          <v-list-item>
            <div
              class="pl-2"
              style="font-size: 11px; text-transform: uppercase"
            >
              {{ $store.state.auth.me.email || "" }}
            </div>
          </v-list-item>
          <v-list-item>
            <v-list-item-title @click="exportKey">
              <v-icon medium> outbox </v-icon>
              <span
                class="pl-2"
                style="font-size: 11px; text-transform: uppercase"
                >export privateKey</span
              >
            </v-list-item-title>
          </v-list-item>
          <v-list-item>
            <v-list-item-title @click="logout">
              <v-icon medium> logout </v-icon>
              <span
                class="pl-2"
                style="font-size: 11px; text-transform: uppercase"
                >ออกจากระบบ</span
              >
            </v-list-item-title>
          </v-list-item>
        </v-list>
      </v-menu>
      <v-spacer></v-spacer>
    </v-app-bar>
  </div>
</template>

<script>
import { firebaseAuth } from "../plugins/firebase";
import pinPad from "../components/pinPad.vue";
export default {
  name: "HomeAppBar",
  components: {
    "pin-pad": pinPad,
  },
  methods: {
    CopyEth() {
      try {
        this.$refs.ethWallet.select();
        document.execCommand("copy");
        this.alert_show({
          type: "success",
          title: "Copied on clipboard !",
        });
      } catch (err) {}
    },
    logout() {
      const vm = this;
      vm.alert_show({
        header: "กรุณายืนยัน",
        type: "confirm",
        title: "ท่านต้องการออกจากระบบ ?",
      }).then(async (res) => {
        if (res) {
          const vm = this;
          try {
            firebaseAuth
              .signOut()
              .then(async () => {
                await vm.$store.commit("SET_LOGGEDIN", false);
                await vm.$store.commit("SET_ME", null);
                await vm.$store.commit("SET_TOKENLIST", []);
                vm.$cookies
                  .keys()
                  .forEach((cookie) => vm.$cookies.remove(cookie));
                localStorage.removeItem("email_account_mpv");
                localStorage.removeItem("encypt_string_mpv");
                await vm.$router.push("/OTP/termService");
              })
              .catch((error) => {
                vm.alert_show({
                  header: "ล้มเหลว",
                  type: "error",
                  title: error.message,
                });
              });
          } catch (err) {}
        }
      });
    },
    exportKey() {
      this.$emit("exportKey");
    },
  },
};
</script>

<style lang="scss">
.bar-me {
  background-color: transparent;
  > header {
    margin: 0 auto;
    padding: 0;
  }
}
.address {
  justify-content: center;
}
</style>

<template>
  <div class="layout-page">
    <v-row no-gutters>
      <v-col cols="12" class="d-flex align-start">
        <v-icon large color="#6F6F6F" @click="$emit('onBack')"
          >keyboard_arrow_left</v-icon
        >
        <span class="ml-2 mt-2">
          <h4 v-html="header" class="grey--text text--darken-1"></h4>
        </span>
        <v-spacer> </v-spacer>
        <v-btn
          v-if="receive"
          rounded
          large
          @click="$emit('receive')"
          class="text-center white--text"
          color="black"
        >
          <v-row>
            <v-col
              ><div style="font-size:14px;" class="font-weight-bold">
                รับเหรียญ
              </div></v-col
            >
            <v-col class="pa-0 pt-3 pb-3"
              ><v-img
                src="../assets/buyicon.svg"
                width="23px"
                height="20px"
              ></v-img
            ></v-col>
          </v-row>
        </v-btn>
      </v-col>
      <v-col cols="12" class="mt-5 px-3">
        <v-row no-gutters justify="center">
          <slot name="content"></slot>
        </v-row>
      </v-col>
    </v-row>
  </div>
</template>

<script>
export default {
  name: "Layout",
  props: {
    header: {
      type: String,
      default: "",
    },
    receive: {
      type: Boolean,
      default: false,
    },
  },
};
</script>

<style lang="scss">
.layout-page {
  min-height: 500px;
  padding: 24px 8px;
}
</style>

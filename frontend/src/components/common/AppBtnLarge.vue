<template>
  <v-btn
    class="btnStyle"
    width="95%"
    height="55px"
    :color="btnColor"
    :disabled="isDisabled"
  >
    <h3 :style="'color:' + btnNameColor">{{ btnName }}</h3>
  </v-btn>
</template>

<script>
export default {
  props: ['btnColor', 'btnName', 'btnNameColor', 'isDisabled'],
};
</script>

<style>
.btnStyle {
  border-radius: 10px !important;
  padding: 0;
  margin: 0;
}
</style>

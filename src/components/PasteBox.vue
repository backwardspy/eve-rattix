<template>
  <textarea
    v-model="walletHistory"
    placeholder="Paste wallet transactions here"
  ></textarea>
</template>

<script>
import _ from "lodash";

export default {
  name: "PasteBox",

  data: function () {
    return {
      walletHistory: "",
    };
  },

  computed: {
    transactionsData: function () {
      if (!this.walletHistory) return;

      let entries = this.walletHistory.split("\n");

      let typePayouts = new Proxy(
        {},
        {
          get: (target, name) => (name in target ? target[name] : 0),
        }
      );

      for (let i = 0; i < entries.length; ++i) {
        try {
          /* eslint-disable no-unused-vars */
          let [date, type, amount, total, desc] = entries[i].split("\t");
          typePayouts[type] += this.parseNumber(amount.slice(0, -4));
        } catch {
          // we just ignore bad lines
        }
      }

      console.log(typePayouts, _.isEmpty(typePayouts));
      return _.isEmpty(typePayouts) ? undefined : typePayouts;
    },
  },

  methods: {
    parseNumber: function (value, locales = navigator.languages) {
      const example = Intl.NumberFormat(locales).format("1.1");
      const cleanPattern = new RegExp(`[^-+0-9${example.charAt(1)}]`, "g");
      const cleaned = value.replace(cleanPattern, "");
      const normalized = cleaned.replace(example.charAt(1), ".");
      return parseFloat(normalized);
    },

    formatNumber: function (number, locales = navigator.languages) {
      return new Intl.NumberFormat(locales).format(number);
    },
  },
};
</script>

<style scoped>
textarea {
  width: 80%;
  height: 12em;
}
</style>

<template>
  <div>
    <div class="stack-precision">
      <span class="order-book-title">{{ $t("common.orderbook") }}</span>
      <div class="grouping-orders">
        <span class="mr-1">{{ $t("common.grouping") }}</span>
        <select
          id="select-stack-precision"
          v-model="selectedPrecision"
          class="form-control"
          @change="changePrecision"
        >
          <option
            v-for="(value, index) in precisions"
            :key="index"
            :value="value"
          >
            {{ value }}
          </option>
        </select>
      </div>
    </div>
    <orders-header
      :base-currency="baseCurrency"
      :quote-currency="quoteCurrency"
    ></orders-header>
    <orders-table
      :type="'sell'"
      :sell-orders="sellOrders"
      :is-not-full="sellOrders && sellOrders.length < 14"
      :class="{ 'sell-not-full': sellOrders && sellOrders.length < 14 }"
    >
      <orders-table-row
        v-for="order in sellOrders"
        :key="order.id"
        class="orange"
        :round="decimals"
        :max-depth="sellOrders[0].depth"
        :order="order"
        :is-circle="order.owner"
        color="rgba(255, 93, 85, .3)"
        @select="sellRowClick"
      ></orders-table-row>
    </orders-table>

    <orders-sell-buy></orders-sell-buy>

    <orders-table :type="'buy'">
      <orders-table-row
        v-for="order in buyOrders"
        :key="order.id"
        class="green"
        :round="decimals"
        :max-depth="buyOrders[buyOrders.length - 1].depth"
        :order="order"
        :is-circle="order.owner"
        color="rgba(31, 167, 114, .3)"
        @select="buyRowClick"
      ></orders-table-row>
    </orders-table>

    <orders-header
      :base-currency="baseCurrency"
      :quote-currency="quoteCurrency"
    ></orders-header>
  </div>
</template>

<script>
import OrdersHeader from "./Orders-header.vue";
import OrdersTable from "./Orders-table.vue";
import OrdersSellBuy from "./Orders-sell-buy.vue";
import OrdersTableRow from "./Orders-table-row.vue";
import { mapGetters } from "vuex";
import getFixedDecimal from "~/mixins/getFixedDecimal";
export default {
  name: "OrdersComp",
  components: {
    OrdersTableRow,
    OrdersSellBuy,
    OrdersTable,
    OrdersHeader,
  },
  mixins: [getFixedDecimal],
  props: [
    // eslint-disable-next-line vue/require-prop-types
    "baseCurrency",
    // eslint-disable-next-line vue/require-prop-types
    "quoteCurrency",
    // eslint-disable-next-line vue/require-prop-types
    "stackPrecision",
    // eslint-disable-next-line vue/require-prop-types
    "precision",
    // eslint-disable-next-line vue/require-prop-types
    "decimals",
  ],
  emits: ["sellrowclick", "buyrowclick", "changePrecision"],
  data() {
    return {
      selectedPrecision: this.precision,
    };
  },
  computed: {
    ...mapGetters({ stack: "core/stack" }),
    sellOrders() {
      return this.getGroupSellsForStakan(this.stack.sells);
    },
    buyOrders() {
      return this.getGroupBuysForStakan(this.stack.buys);
    },
    precisions() {
      return this.stackPrecision;
    },
    setPrecision() {
      let toFixed = 0;
      if (this.selectedPrecision) {
        const decimalPlaces = this.selectedPrecision.split(".")[1];
        if (decimalPlaces) {
          toFixed = decimalPlaces.length;
        } else {
          // If there are no decimal places specified, default to 0
          toFixed = 0;
        }
        
        // Check if the number is less than 1, and set precision to display the whole number
        if (parseFloat(this.selectedPrecision) < 1) {
          toFixed = 0;
        }
      }
      return toFixed;
    }
  },
  watch: {
    precision(newPrecision) {
      this.selectedPrecision = newPrecision;
    },
    selectedPrecision(newPrecision, oldPrecision) {
      this.$emit("changePrecision", { old: oldPrecision, new: newPrecision });
    },
  },
  methods: {
    sellRowClick(price, quantity) {
      this.$emit("sellrowclick", { price, quantity });
    },
    buyRowClick(price, quantity) {
      this.$emit("buyrowclick", { price, quantity });
    },
    getGroupSellsForStakan(orders) {
      if (orders && orders.length) {
        orders.reverse();
        let groupdepth = 0;
        orders.forEach((order) => {
          order.price = this.setPrecision
            ? order.price
            : order.price;
          groupdepth += order.quantity;
          order.depth = groupdepth;
        });
      }
      return orders ? orders.reverse() : orders;
    },
    getGroupBuysForStakan(orders) {
      if (orders && orders.length) {
        let groupdepth = 0;
        orders.forEach((order) => {
          order.price = this.setPrecision
            ? parseFloat(order.price).toFixed(this.setPrecision)
            : order.price;
          groupdepth += order.quantity;
          order.depth = groupdepth;
        });
      }
      return orders;
    },
    changePrecision() {
      localStorage.setItem(
        `precision_${this.baseCurrency}-${this.quoteCurrency}`,
        this.selectedPrecision
      );
    },
  },
};
</script>

<style lang="scss">
.stack-precision {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 8px;
  background: #fff;
  border-bottom: 1px solid #e0e0e0;
  span.order-book-title {
    font-weight: 700;
    font-size: 14px;
    display: block;
    text-transform: uppercase;
  }

  .grouping-orders {
    span {
      font-size: 14px;
    }
  }
  #select-stack-precision {
    display: inline-block;
    border-radius: 0;
    padding: 0 0 0 5px;
    color: #4a4a4a;
    font-size: 12px;
    width: 90px;
    height: 28px;
  }
}
</style>

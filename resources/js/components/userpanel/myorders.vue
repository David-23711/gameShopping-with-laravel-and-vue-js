<template>
  <div>
    <div class="Wishlist" style="margin-top: 90px">
      <h4>My Orders</h4>
      <v-breadcrumbs
        class="d-block"
        dark
        :items="items"
        divider=">"
      ></v-breadcrumbs>
    </div>
    <v-container>
      <v-row>
        <v-col cols="12" md="8">
          <v-simple-table dark>
            <thead>
              <tr>
                <td>Name</td>
                <td>Unit Price</td>
                <td>Quantity</td>
                <td>Discount</td>
                <td>Price</td>
              </tr>
            </thead>
            <tbody>
              <tr v-for="data in orderDatas" :key="data.id">
                <td>{{ data.name }}</td>
                <td>{{ data.price.toLocaleString() }} MMK</td>
                <td>{{ data.cartqty }}</td>
                <td>{{ data.discount ? data.discount : 0 }}%</td>
                <td>
                  {{ data.discountPrice.toLocaleString() }}
                  MMK
                </td>
              </tr>
              <tr>
                <td colspan="4" align="end">Total Price</td>
                <td>{{ totalPrice.toLocaleString() }} MMK</td>
              </tr>
            </tbody>
          </v-simple-table>
        </v-col>
        <v-col cols="12" md="4">
          <v-sheet dark>
            <v-row>
              <v-col cols="12" md="12">
                <v-text-field
                  v-model="user_name"
                  outlined
                  placeholder="Enter Your Name"
                />

                <v-select
                  outlined
                  v-model="payment_method"
                  :items="selectItems"
                  label="Choose Payment Method"
                ></v-select>
                <v-text-field
                  v-model="phone"
                  outlined
                  placeholder="Enter Your Phone"
                />
                <v-textarea
                  v-model="address"
                  outlined
                  placeholder="Enter Your Address"
                />
                <v-btn @click="submitOrder">Submit Order</v-btn>
              </v-col>
            </v-row>
          </v-sheet>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import { exampleMixin } from "../userpanel/utils/reuseable";
import { mapGetters } from "vuex";
import axios from "axios";
import Swal from "sweetalert2";
import QuickView from "../userpanel/path/quickView.vue";
import "../../../css/app.css";
import { eventBus } from "../../app";
export default {
  mixins: [exampleMixin],
  data() {
    return {
      user_name: "",
      payment_method: "",
      phone: "",
      address: "",
      orderDatas: [],
      postOrders: [],
      qtyonhand: [],
      allPrice: [],
      totalPrice: 0,
      user_id: "",
      selectItems: ["Pay On Delivery"],
      items: [
        { text: "Home", disabled: false, href: "/user/index" },
        { text: "Shopping Cart", disabled: false, href: "/user/addtocart" },
        { text: "Order", disabled: true, href: "/user/whishlist" },
      ],
    };
  },
  components: {
    QuickView,
  },

  methods: {
    scrollTop() {
      window.scrollTo(0, 0);
    },
    async getAddToCartGame() {
      await axios
        .get(`/user/addtocart/getCartGames/${this.user_id}`)
        .then((resp) => {
          this.orderDatas = resp.data;
          for (let i = 0; i < resp.data.length; i++) {
            this.allPrice.push(
              resp.data[i].cartqty * resp.data[i].price -
                (resp.data[i].cartqty *
                  resp.data[i].price *
                  resp.data[i].discount) /
                  100
            );
            this.qtyonhand.push(resp.data[i].qty - resp.data[i].cartqty);
          }
          for (let j = 0; j < this.allPrice.length; j++) {
            this.totalPrice += this.allPrice[j];
          }
          for (let h = 0; h < this.allPrice.length; h++) {
            this.orderDatas[h] = {
              ...this.orderDatas[h],
              discountPrice: this.allPrice[h],
              qoh: this.qtyonhand[h],
            };
          }
        });
    },
    async submitOrder() {
      for (let i = 0; i < this.orderDatas.length; i++) {
        this.orderDatas[i] = {
          ...this.orderDatas[i],
          user_name: this.user_name,
          payment_method: this.payment_method,
          phone: this.phone,
          address: this.address,
          user_id: this.user_id,
        };
      }
      await axios
        .post(`/user/myorders/insert`, this.orderDatas)
        .then((resp) => {
          alert("success");
          eventBus.$emit("submitOrder");
          this.$router.push("/user/myorderlists");
        });
    },
  },
  computed: {
    ...mapGetters(["userData"]),
  },
  mounted() {
    if (this.userData) {
      window.axios.defaults.headers.common[
        "Authorization"
      ] = `Bearer ${this.userData}`;
      axios.get("/api/user").then((resp) => {
        this.user_id = resp.data.id;
        this.getAddToCartGame();
      });
    }
    this.scrollTop();
  },
};
</script>

<style>
</style>
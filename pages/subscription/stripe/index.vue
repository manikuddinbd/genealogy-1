<template>
  <div>
    <div class="columns is-gapless is-multiline is-mobile">
      <div class="column is-12">
        <h1 class="is-size-4 has-text-black">
          <span class="has-text-weight-medium">Payment</span>
        </h1>
      </div>
      <div class="column is-12">
        <nav class="breadcrumb mt-1 mb-0" aria-label="breadcrumbs">
          <ul>
            <li><a class="is-size-7 has-text-weight-medium has-text-link"
                   href="subscription">Subscription</a></li>
            <li class="is-size-7 has-text-weight-medium is-active"><a href="/dashboard"
                                                                      aria-current="page">Payment</a></li>
          </ul>
        </nav>
      </div>
    </div>
    <div class="columns is-variable is-3">
      <div class="column is-3">
        <div class="card has-background-white has-text-black">
          <div class="card-content payment_block" v-if="clientSecret">
            <label> Card holder name</label>
            <div class='mb-2'>
                <input id="card-holder-name" type="text" placeholder="eg. Jon Doe" class="input">
            </div>
            <label>Card Number</label>
            <div class='mb-2 mt-1'>
                <!-- Stripe Elements Placeholder -->
                <div id="card-element"></div>
            </div>
             
            <button id="card-button" :dataSecret="clientSecret" @click="createPaymentMethod" class="button is-size-7 is-uppercase has-text-white has-background-primary has-text-weight-medium is-light mt-4">
                Process
            </button>
          </div>
        </div>
      </div>
      <div class="column is-6">
      </div>
      <div class="column is-3">
        <div class="columns is-gapless is-multiline">
          <div class="card plan_info_block has-background-primary has-text-black">
            <img src="~assets/images/plan_info_img.svg" alt="">
            <div class="card-content">
              <div class="has-text-black has-text-weight-medium is-flex plans_info mb-5">
                <i class="fas fa-check mr-2 mt-1"></i>
                <p class="is-size-7">Family Tree 365 is a secure online website
                  which you can use to create your own family tree(s) with.</p>
              </div>
              <div class="has-text-black has-text-weight-medium is-flex plans_info mb-5">
                <i class="fas fa-check mr-2 mt-1"></i>
                <p class="is-size-7">It has a tree viewer and DNA support
                  more features are planned such as the inclusion of archive databases and
                  collections.</p>
              </div>
              <div class="has-text-black has-text-weight-medium is-flex plans_info">
                <i class="fas fa-check mr-2 mt-1"></i>
                <p class="is-size-7">Set up your first family tree free of charge.
                  We offer different pricing levels with optional subscriptions if you need to create extra trees.</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>


<script>
import Vue from 'vue'


const stripe = Stripe(process.env.STRIPE_PK);
let elements = null;
let cardElement = null;

export default {
  layout: 'default',
  head: {
    title: 'Stripe'
  },
  data () {
    return {
      clientSecret: "wtfffffff",
    };
  },
  mounted () {
    this.$axios
        .$get("/api/stripe/intent", this.payment)
        .then((data) => {
            this.clientSecret = data.intent.client_secret
         
            elements = stripe.elements();
            cardElement = elements.create('card');
         
            cardElement.mount('#card-element');


           }).catch(error => {
          console.log(error)
        });
  },
  beforeDestroy () {

  },
  methods: {
    async createPaymentMethod() {
        const cardHolderName = document.getElementById('card-holder-name');
        const cardButton = document.getElementById('card-button');

        const { setupIntent, error } = await stripe.confirmCardSetup(
            this.clientSecret, {
                payment_method: {
                    card: cardElement,
                    billing_details: { name: cardHolderName.value }
                }
            }
        );
        if (error)
            return alert("Sorry there was an error creating your subscription intent");


        if (!error) {
            this.$axios
                .$post("/api/stripe/subscribe", {
                    payment_method: setupIntent.payment_method,
                    plan_id: new URLSearchParams(window.location.href.split('?')[1]).get('name')
                }).then(() => {
                    this.$router.push('/subscription');
                })
                .catch(error => {
                    alert("Sorry there was an error creating your subscription");
                });
        }
    }
  }
}

</script>
<router>
{
    name: 'subscription.stripe.index'
}
</router>

<style scoped>
.card-content input {
    width: 100%;
}
#custom-button {
  height: 30px;
  outline: 1px solid grey;
  background-color: green;
  padding: 5px;
  color: white;
}

#card-error {
  color: red;
}
</style>

<template>
  <div style="background-color: #fff">
    <apple-pay-button
      buttonstyle="black"
      type="buy"
      locale="en-GB"
      @click="onApplePayButtonClicked"
      
    ></apple-pay-button>
  </div>
</template>
<!-- v-if="showButton()" -->
<script>
import Axios from "axios";
const serverUrl = 'https://dev.acmedao.com';
const accessToken = window.vm.config.globalProperties.$cookie.getCookie('session');

const headers = {
    'Content-Type': 'application/json',
    Accept: 'application/json',
    Authorization: 'Bearer ' + accessToken,
    withCredentials: true,
};

function loginToAcmeBackend() {
    try {
        Axios.post(`${serverUrl}/user/login`, {}, { headers });
        console.log('Successfully logged in with Apple Pay');
    } catch (err) {
        console.log('Failed to log in ' + err);
    }
}

export default {
  data() {
    return {
      responseObject: null,
      loading: true,
      errored: false,
      // These need to be set
      nftPrice: 15.59,
      estimatedTransactionCostUSD: 49.59,
      amount: 14.55,
      token_id: 1,
      contract_id: '0x0eAfA761ce742Fb9C9Dc99190cEC726BE85E6Ca4',
      walletId: null,
      token: null,
      ipaddress: '0.0.0.0',
    };
  },
  mounted() {
    let recaptchaScript = document.createElement("script");
    recaptchaScript.setAttribute(
      "src",
      "https://applepay.cdn-apple.com/jsapi/v1/apple-pay-sdk.js"
    );
    document.head.appendChild(recaptchaScript);
    console.log('Apple Pay mounted');

    Axios.defaults.withCredentials = true;

    loginToAcmeBackend();
  },
  methods: {
    showButton() {
      return window.safari !== undefined;
    },
    //Invoked Method
    onApplePayButtonClicked() {
      console.log('onApplePayButtonClicked');
      if (!ApplePaySession) {
        return;
      }
      // Define ApplePayPaymentRequest
      this.getApplePayRequest();
      //Get Request Based Wyre Quote
      // Create ApplePaySession
      const session = new ApplePaySession(3, this.responseObject);
      session.onvalidatemerchant = async (event) => {
        console.log('onvalidatemerchant')
        // Call your own server to request a new merchant session.
        const merchantSession = this.authapplepay();
        session.completeMerchantValidation(merchantSession);
      };
      session.onpaymentauthorized = (event) => {
        console.log('onpaymentauthorized')
        // Define ApplePayPaymentAuthorizationResult
        const result = {
          status: ApplePaySession.STATUS_SUCCESS,
        };
        session.completePayment(result);
        handleApplePayOrder(event);
      };
      session.oncancel = (event) => {
        console.log("Payment cancelled by WebKit");
        // Payment cancelled by WebKit
      };
      session.begin();
    },
    getApplePayRequest() {
      console.log('posting to createApplePayRequest');
      const serverUrl = "https://dev.acmedao.com";
      this.amount = this.nftPrice + this.estimatedTransactionCostUSD;
      const raw = JSON.stringify({
        amount: this.amount.toString(),
      });
      Axios.post(`${serverUrl}/user/createApplePayRequest`, raw, {
        headers,
      })
        .then((response) => {
          console.log('createApplePayRequest: ' + response)
          this.responseObject = response;
        })
        .catch((error) => {
          console.log(error);
          this.errored = true;
        })
        .finally(() => (this.loading = false));
    },
    authapplepay() {
      console.log('posting to createApplePaySession');
      Axios.post(`${serverUrl}/user/createApplePaySession`, { }, { headers })
        .then((response) => {
          console.log('createApplePaySession: ' + response)
          this.responseObject = response;
        })
        .catch((error) => {
          console.log(error);
          this.errored = true;
        })
        .finally(() => (this.loading = false));
    },
    handleApplePayOrder(e) {
      console.log('handleApplePayOrder');
      // variables
      const orderParams = {
        price: this.nftPrice,
        usdPrice: this.amount,
        fees: this.estimatedTransactionCostUSD,
        token_id: this.tokenId,
        contract_id: this.contractId,
        walletId: this.walletId,
        token: e.payment.token,
        ipaddress: this.ipAddress,
      };
      Axios.post(`${serverUrl}/user/createApplePayOrder`, orderParams, {
        headers,
      })
        .then((response) => {
          console.log('createApplePayOrder: ' + response)
          this.responseObject = response;
        })
        .catch((error) => {
          console.log(error);
          this.errored = true;
        })
        .finally(() => (this.loading = false));
    },
  },
};
</script>

<style>
apple-pay-button {
  --apple-pay-button-width: 140px;
  --apple-pay-button-height: 30px;
  --apple-pay-button-border-radius: 5px;
  --apple-pay-button-padding: 5px 0px;
  display: initial;
}
</style>
<template>
  <div class="login-form">
    <img style="height: 25%" src="@/assets/images/greenwich/greenwich.svg" alt="Logo">
    <notifications group="default" />
    <vs-input
        v-validate="'required|min:3'"
        data-vv-validate-on="blur"
        name="email"
        icon-no-border
        icon="icon icon-user"
        icon-pack="feather"
        v-model="email"
        placeholder="Email"
        color="#28a745"
        class="w-full"/>
    <span class="text-info text-sm">{{ errors.first('email') }}</span>

    <vs-input
        data-vv-validate-on="blur"
        v-validate="'required|min:6'"
        type="password"
        name="password"
        icon-no-border
        icon="icon icon-lock"
        icon-pack="feather"
        v-model="password"
        placeholder="Password"
        color="#28a745"
        class="w-full mt-6" />
    <span class="text-info text-sm">{{ errors.first('password') }}</span>
    <br>
    <div class="d-block text-center">
      <b-btn variant="info" @click="login()" size="lg">Login</b-btn>
    </div>
    <Loading :active.sync="isLoading"
             :can-cancel="true"
             :on-cancel="onCancel"
             :is-full-page="fullPage"></Loading>

  </div>
</template>

<style type="text/css" scoped>
  button:hover {
    transform: rotate(10deg);
    border: 3px solid #f1f1f1;
  }
  .login-form {
    padding: 5px;
    position: fixed; top: 35%; left: 38%;
    width: 25%;
    height: 29%;
    border-radius: 20px;
    background-color: rgb(0,0,0); /* Fallback color */
    background-color: rgba(0,0,0, 0.4); /* Black w/opacity/see-through */
    border: 3px solid #f1f1f1;
    font-weight: bold;
  }
</style>

<script>
import Service from "@/domain/services/api"
import commonHelper from "@/infrastructures/common-helpers"
import Loading from 'vue-loading-overlay';
// Import stylesheet
import 'vue-loading-overlay/dist/vue-loading.css';
import {BACKEND_BASE_URL, ERROR_COMMON} from "../../../configs/environment";
const axios = require('axios').default;

export default {
  components: {Loading},
  comments: {
    Service,
    Loading
  },
  data() {
    return {
      email: '',
      password: '',
      isLoading: false,
      fullPage: true
    }
  },
  computed: {},
  created() {
    this.checkLogin();
  },
  methods: {
    checkLogin() {
      const accessToken = localStorage.getItem('access_token');
      axios.get(`${BACKEND_BASE_URL}/user/info`, {
        headers: { Authorization: `Bearer ${accessToken}`}
      })
        .then(() => {
          return window.location.href = '/adm'
        })
        .catch((error) => {
          if (error.response.status === 401) return commonHelper.showMessage('Vui Lòng Đăng Nhập', 'warning')
          return commonHelper.showMessage(ERROR_COMMON, 'warning')
        })
    },
    onCancel() {},
    login() {
      this.isLoading = true;
      let dataSend = {
        username: this.email,
        password: this.password
      }
      axios.post(`${BACKEND_BASE_URL}/auth/login`, dataSend)
        .then(res => {
          if (res.data.access_token) {
            localStorage.setItem('access_token', res.data.access_token);
            return window.location.href = '/adm'
          }
          return commonHelper.showMessage('Sai Tài Khoản', 'warning')
        })
        .catch((error) => {
          if (error.response.status === 401) return commonHelper.showMessage('Sai Tài Khoản', 'warning')
          commonHelper.showMessage(ERROR_COMMON, 'warning')
        })
        .finally(() => {
          this.isLoading = false;
        })
    }
  }
}

</script>


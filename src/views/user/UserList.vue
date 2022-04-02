<template>

  <div id="page-user-list">
    <header-fptschool></header-fptschool>
    <b-row class="ml-3">
      <b-col>
        <b-form-select v-model="roleSearch" :options="listGroup"></b-form-select>
      </b-col>
      <b-col>
        <b-input :placeholder="`Email`" type="email" v-model="emailSearch"></b-input>
      </b-col>
      <b-col>
        <b-input :placeholder="`Full name`" v-model="fullNameSearch"></b-input>
      </b-col>
      <b-col>
        <b-input :placeholder="`Phone Number`" v-model="phoneNumberSearch"></b-input>
      </b-col>
      <b-col>
        <b-btn class="mr-3" variant="outline-info" @click="getUser()">Search</b-btn>
      </b-col>
    </b-row>
    <br>
    <br>
    <b-row class="ml-3">
      <b-col>
        <div class="w-50">
          <b-form-input
            class="mb-5"
            id="filter-input"
            v-model="filter"
            type="search"
            placeholder="Type to Search"
          ></b-form-input>
        </div>
      </b-col>
    </b-row>
    <b-row class="ml-3">
      <b-table
        :filter="filter"
        class="ml-5 mr-5"
        hover
        responsive
        striped
        :fields="fieldsDataUsers"
        :items="dataUsers"
        :per-page="perPage"
        :current-page="currentPage"
      >
        <template v-slot:cell(manage)="row">
          <b-btn class="mr-3" variant="outline-success" @click="modifyUser('1', row.item.id)">
            <feather-icon icon="UnlockIcon" svgClasses="h-4 w-4"/>
          </b-btn>
          <b-btn class="mr-3" variant="outline-warning" @click="modifyUser('2', row.item.id)">
            <feather-icon icon="LockIcon" svgClasses="h-4 w-4"/>
          </b-btn>
          <b-btn variant="outline-danger" @click="modifyUser('3', row.item.id)">
            <feather-icon icon="TrashIcon" svgClasses="h-4 w-4"/>
          </b-btn>
        </template>
      </b-table>
    </b-row>
    <br>
    <b-row>
      <div class="d-flex justify-content-center w-100">
        <b-pagination
          align="center"
          v-model="currentPage"
          :total-rows="rows"
          :per-page="perPage"
          aria-controls="my-table"
        >
          <template #first-text><span class="text-success">First</span></template>
          <template #prev-text><span class="text-danger">Prev</span></template>
          <template #next-text><span class="text-warning">Next</span></template>
          <template #last-text><span class="text-info">Last</span></template>
        </b-pagination>
      </div>
    </b-row>
  </div>

</template>

<script>
import '@/assets/scss/vuexy/extraComponents/agGridStyleOverride.scss'
import Multiselect from 'vue-multiselect'
import commonHelper from "@/infrastructures/common-helpers"
import FacultySelect from "@/views/components/component/FacultySelect";
import HeaderFptschool from "@/views/Header";
import {BACKEND_BASE_URL, ERROR_COMMON, SUCCESS_COMMON} from "../../configs/environment";
const axios = require('axios').default;

export default {
  components: {
    HeaderFptschool,
    Multiselect,
    FacultySelect
  },
  data() {
    return {
      accessToken: null,
      perPage: 9,
      currentPage: 1,
      fieldsDataUsers: [
        {key: 'id', label: 'Id', sortable: true},
        {key: 'email', label: 'Email', sortable: true},
        {key: 'role', label: 'Chức Năng', sortable: true},
        {key: 'full_name', label: 'Họ Tên', sortable: true},
        {key: 'address', label: 'Địa Chỉ', sortable: true},
        {key: 'status', label: 'Trạng Thái', sortable: true},
        {key: 'manage', label: 'Manage'}
      ],
      dataUsers: [],
      filter: null,

      roleSearch: null,
      fullNameSearch: '',
      phoneNumberSearch: '',
      emailSearch: '',

      optionsRole: [
        { value: null, text: 'Please select role' },
        { value: 1, text: 'Admin' },
        { value: 2, text: 'Marketing Coordinator' },
        { value: 3, text: 'Student' },
        { value: 4, text: 'Marketing Manager' },
        { value: 5, text: 'Guest' },
      ],
      listGroup: [],
    }
  },
  watch: {
  },
  computed: {
    rows() {
      return this.dataUsers.length
    }

  },
  methods: {
    getListGroup () {
      axios.get(`${BACKEND_BASE_URL}/user/list-group`, {
        headers: { Authorization: `Bearer ${this.accessToken}`}
      })
      .then((res) => {
        this.listGroup = Object.values(res.data.data).map(e => {
          return {
            value: e,
            text: e
          }
        })
        this.listGroup.push({
          value: null,
          text: 'Select role...'
        })
        return commonHelper.showMessage(SUCCESS_COMMON, 'success')
      })
      .catch((error) => {
        if (error.response.status === 401) {
          commonHelper.showMessage('Vui Lòng Đăng Nhập', 'warning')
          return window.location.href = '/adm/login'
        }
        commonHelper.showMessage(ERROR_COMMON, 'warning')
      })
    },
    getUser () {
      let data = {
        full_name: this.fullNameSearch,
        phone_number: this.phoneNumberSearch,
        email: this.emailSearch,
        group_id: this.roleSearch
      }
      this.dataUsers = []
      axios.get(`${BACKEND_BASE_URL}/user/user-list`, {
        'params': data,
        headers: { Authorization: `Bearer ${this.accessToken}`}
      })
        .then(res => {
          this.dataUsers = res.data.data
          return commonHelper.showMessage(SUCCESS_COMMON, 'success')
        })
        .catch((error) => {
          if (error.response.status === 401) {
            commonHelper.showMessage('Vui Lòng Đăng Nhập', 'warning')
            return window.location.href = '/adm/login'
          }
          commonHelper.showMessage(ERROR_COMMON, 'warning')
        })
    },

    modifyUser (modifyType, idUser) {
      if (!confirm('Bạn có chắc chắn thực hiện hành động này?')) return 1
      let param = {
        modifyType,
        idUser
      }
      axios.get(`${BACKEND_BASE_URL}/user/modify-user`, {
        params: param,
        headers: { Authorization: `Bearer ${this.accessToken}`}
      })
        .then(res => {
          if (res.data.data) {
            this.getUser()
            return commonHelper.showMessage('Thao tác thành công', 'success')
          }
          commonHelper.showMessage(ERROR_COMMON, 'warning')
        })
        .catch((error) => {
          if (error.response.status === 401) {
            commonHelper.showMessage('Vui Lòng Đăng Nhập', 'warning')
            return window.location.href = '/adm/login'
          }
          commonHelper.showMessage(ERROR_COMMON, 'warning')
        })
    }
  },
  mounted() {
  },
  created() {
    this.accessToken = localStorage.getItem('access_token');
    this.getUser()
    this.getListGroup()
  }
}

</script>

<style lang="scss">
#page-user-list {
  .user-list-filters {
    .vs__actions {
      position: absolute;
      right: 0;
      top: 50%;
      transform: translateY(-58%);
    }
  }
}
</style>

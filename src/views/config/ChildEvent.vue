<template>

  <div id="page-user-list">
    <header-fptschool></header-fptschool>
    <b-row class="ml-3">
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
        <b-btn class="mr-3" variant="outline-info" @click="getListChildEvent()">Search</b-btn>
      </b-col>
    </b-row>
    <b-row class="ml-3 mt-3">
      <b-col>
        <b-input v-model="childEventName" :placeholder="`Tên Gợi Nhớ`"></b-input>
      </b-col>
      <b-col>
        <b-input v-model="childEventTitle" :placeholder="`Tiêu Đề`"></b-input>
      </b-col>
      <b-col>
        <b-input v-model="childEventContent" :placeholder="`Nội Dung`"></b-input>
      </b-col>
      <b-col>
        <b-form-file class="mb-3" :accept="`image/*`" v-model="childImg" @change="loadFile(childImg)"></b-form-file>
        <img class="w-100" id="output"/>
      </b-col>
      <b-col>
        <b-btn variant="outline-success" @click="createChildEvent">Tạo Child Event</b-btn>
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
        :fields="fieldsDataChildEvents"
        :items="dataChildEvents"
        :per-page="perPage"
        :current-page="currentPage"
      >
        <template v-slot:cell(img)="row">
          <img class="w-25" :src="getImg(row.item.img)" alt="">
        </template>
        <template v-slot:cell(manage)="row">
          <b-btn class="mr-3" variant="outline-warning" @click="modifyUser('2', row.item.id)">
            <feather-icon icon="EditIcon" svgClasses="h-4 w-4"/>
          </b-btn>
          <b-btn variant="outline-danger" @click="deleteEvent(row.item.id)">
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
      childEventName: null,
      childEventTitle: null,
      childEventContent: null,
      childImg: null,
      accessToken: null,
      perPage: 9,
      currentPage: 1,
      fieldsDataChildEvents: [
        {key: 'id', label: 'Id', sortable: true},
        {key: 'name', label: 'Tên', sortable: true},
        {key: 'title', label: 'Tiêu Đề', sortable: true},
        {key: 'content', label: 'Nội Dung', sortable: true},
        {key: 'img', label: 'Ảnh', sortable: true},
        {key: 'manage', label: 'Quản Lý ', sortable: true},
      ],
      dataChildEvents: [],
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
    }
  },
  watch: {
  },
  computed: {
    rows() {
      return this.dataChildEvents.length
    }

  },
  methods: {
    createChildEvent() {
      let formData = new FormData(); // Currently empty
      formData.append('file', this.childImg);
      formData.append('name', this.childEventName);
      formData.append('title', this.childEventTitle);
      formData.append('content', this.childEventContent);
      axios.post(`${BACKEND_BASE_URL}/event/child`, formData, {
        headers: { Authorization: `Bearer ${this.accessToken}`}
      })
      .then(res => {
        if (res.data.data) {
          this.getListChildEvent();
          return commonHelper.showMessage('Tạo Child Event Thành Công', 'success')
        }
      })
      .catch(error => {
        if (error.response.status === 401) {
          commonHelper.showMessage('Vui Lòng Đăng Nhập', 'warning')
          return window.location.href = '/adm/login'
        }
        if (error.response.status === 406) {
          return commonHelper.showMessage('Tài Khoản Không Có Quyền', 'warning')
        }
        commonHelper.showMessage(ERROR_COMMON, 'warning')
      })
    },
    deleteEvent(eventId) {
      if (!confirm('Bạn có chắc chắn thực hiện hành động này?')) return 1
      axios.get(`${BACKEND_BASE_URL}/event/deleteChildEvent?eventId=${eventId}`, {
        headers: { Authorization: `Bearer ${this.accessToken}`}
      })
        .then(res => {
          if (res.data.data) {
            this.getListChildEvent();
            return commonHelper.showMessage('Xóa Event Thành Công', 'success')
          }
        })
        .catch(error => {
          if (error.response.status === 401) {
            commonHelper.showMessage('Vui Lòng Đăng Nhập', 'warning')
            return window.location.href = '/adm/login'
          }
          if (error.response.status === 406) {
            return commonHelper.showMessage('Tài Khoản Không Có Quyền', 'warning')
          }
          commonHelper.showMessage(ERROR_COMMON, 'warning')
        })
    },
    loadFile () {
      setTimeout(() => {
        const output = document.getElementById('output');
        output.src = URL.createObjectURL(this.childImg);
        output.onload = function() {
          URL.revokeObjectURL(output.src) // free memory
        }
      }, 200)
    },
    getImg (imgPath) {
      return `${BACKEND_BASE_URL}/event/get-image?image_path=${imgPath}`;
    },
    getListChildEvent () {
      axios.get(`${BACKEND_BASE_URL}/event/child`, {
        headers: { Authorization: `Bearer ${this.accessToken}`}
      })
        .then((res) => {
          if (res.data.data) {
            this.dataChildEvents = res.data.data;
            return commonHelper.showMessage(SUCCESS_COMMON, 'success')
          }
          commonHelper.showMessage(ERROR_COMMON, 'warning')
        })
        .catch((error) => {
          if (error.response.status === 401) {
            commonHelper.showMessage('Vui Lòng Đăng Nhập', 'warning')
            return window.location.href = '/adm/login'
          }
          if (error.response.status === 406) {
            return commonHelper.showMessage('Tài Khoản Không Có Quyền', 'warning')
          }
          commonHelper.showMessage(ERROR_COMMON, 'warning')
        })
    },
  },
  mounted() {
  },
  created() {
    this.accessToken = localStorage.getItem('access_token');
    this.getListChildEvent()
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

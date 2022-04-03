<template>
<div>
  <b-row class="mb-3">
    <b-col>
      <h3>Tên Child Event</h3>
    </b-col>
    <b-col>
      <b-input v-model="childEvent.name"></b-input>
    </b-col>
  </b-row>
  <b-row class="mb-3">
    <b-col>
      <h3>Tiêu Đề</h3>
    </b-col>
    <b-col>
      <b-input v-model="childEvent.title"></b-input>
    </b-col>
  </b-row>
  <b-row class="mb-3">
    <b-col>
      <h3>Nội Dung</h3>
    </b-col>
    <b-col>
      <b-input v-model="childEvent.content"></b-input>
    </b-col>
  </b-row>
  <b-row>
    <b-col>
      <b-form-file class="mb-3" :accept="`image/*`" v-model="childImg" @change="loadFile(childImg)"></b-form-file>
      <img :src="getImg(childEvent.img)" class="w-100" id="outputEdit"/>
    </b-col>
  </b-row>
  <br>
  <b-row class="justify-content-end">
    <b-btn class="mr-3 mb-3" variant="outline-success" @click="updateChildEvent">Cập Nhật</b-btn>
  </b-row>
</div>
</template>

<style></style>
<script>

import commonHelper from "@/infrastructures/common-helpers"
const axios = require('axios').default;
import {BACKEND_BASE_URL, ERROR_COMMON} from "../../../configs/environment";

export default {
  name: 'child-event-edit',
  data() {
    return {
      childImg: null,
    }
  },
  props: {
    childEvent: null,
    accessToken: null,
  },
  components: {
  },
  mounted() {
  },
  created() {
  },
  methods: {
    updateChildEvent() {
      let formData = new FormData(); // Currently empty
      formData.append('file', this.childImg);
      formData.append('name', this.childEvent.name);
      formData.append('title', this.childEvent.title);
      formData.append('content', this.childEvent.content);
      formData.append('id', this.childEvent.id);
      axios.post(`${BACKEND_BASE_URL}/event/updateChildEvent`, formData, {
        headers: { Authorization: `Bearer ${this.accessToken}`},
      })
        .then(res => {
          if (res.data.data) {
            this.$emit('updateSuccess');
            return commonHelper.showMessage('Update Event Thành Công', 'success')
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
    getImg (imgPath) {
      return `${BACKEND_BASE_URL}/event/get-image?image_path=${imgPath}`;
    },
    loadFile () {
      setTimeout(() => {
        const output = document.getElementById('outputEdit');
        output.src = URL.createObjectURL(this.childImg);
        output.onload = function() {
          URL.revokeObjectURL(output.src) // free memory
        }
      }, 200)
    },
    setEdit () {
      this.isUpdate = true;
    },
    closeModel () {
      this.$bvModal.hide('childEventEdit')
    },
  },
  watch: {
  }
}
</script>

<template>
  <div>
    <h2>Make verification request</h2>
    <b-form-select v-model="selected" :options="options" style="width: 50%; height: 5%" ></b-form-select>
    <hr>
    <form enctype="multipart/form-data" novalidate v-if="isInitial || isSaving">
      <div class="dropbox">
        <input type="file" multiple :name="uploadFieldName" :disabled="isSaving"
               @change="upload($event.target.name, $event.target.files); fileCount = $event.target.files.length"
               accept="image/*" class="input-file">
        <p v-if="isSaving">
          Uploading {{ fileCount }} files...
        </p>
      </div>
    </form>
    <hr>
    <b-button @click="uploadContent()">Send</b-button>

    <!--SUCCESS-->
    <div v-if="isSuccess">
      <h2>Uploaded {{ uploadedFiles.length }} file(s) successfully.</h2>
      <p>
        <a href="javascript:void(0)" @click="reset()">Upload again</a>
      </p>
      <ul class="list-unstyled">
        <li v-for="(item, index) in uploadedFiles" v-bind:key="index">
          <img :src="item.url" class="img-responsive img-thumbnail" :alt="item.originalName">
        </li>
      </ul>
    </div>

    <!--FAILED-->
    <div v-if="isFailed">
      <h2>Uploaded failed.</h2>
      <p>
        <a href="javascript:void(0)" @click="reset()">Try again</a>
      </p>
      <pre>{{ uploadError }}</pre>
    </div>

  </div>
</template>

<script>
import {upload} from "@/components/ContentUpload/file-upload.service";
import {wait} from "@/components/ContentUpload/utils";
const STATUS_INITIAL = 0, STATUS_SAVING = 1, STATUS_SUCCESS = 2, STATUS_FAILED = 3;

export default {

  name: "VerificationForm",
  data() {
    return {
      selected: 'INFLUENCER',
      options: [
        {value: 'INFLUENCER', text: 'Influencer'},
        {value: 'SPORTS', text: 'Sports'},
        {value: 'NEW_MEDIA', text: 'New media'},
        {value: 'BUSINESS', text: 'Business'},
        {value: 'BRAND', text: 'Brand'},
        {value: 'ORGANIZATION', text: 'Organization'},
      ],
      formData: null,
      uploadFieldName: 'photos',
      uploadedFiles: [],
      uploadError: null,
      currentStatus: null,
    }
  },
  mounted() {
    this.reset();
  },
  methods:{
    uploadContent() {
      this.$http
          .post(process.env.VUE_APP_CONTENT_URL + 'image/save', this.formData)
          .then(response => {
            this.sendVerificationRequest(response.data)
          })
    },
    sendVerificationRequest(officialDocumentImageName){
      let dto = {'category':this.selected,'officialDocumentImageName':officialDocumentImageName, 'name':null, 'surname':null}
      this.$http
          .post(process.env.VUE_APP_BACKEND_URL + 'verification/', dto)
          .then(response => {
            alert(response.data);
            this.$router.push('/')
          })
    },
    reset() {
      // reset form to initial state
      this.currentStatus = STATUS_INITIAL;
      this.uploadedFiles = [];
      this.uploadError = null;
    },

    save(formData) {
      // upload data to the server
      this.currentStatus = STATUS_SAVING;
      console.log(formData);
      upload(formData)
          .then(wait(1500)) // DEV ONLY: wait for 1.5s
          .then(x => {
            this.uploadedFiles = [].concat(x);
            this.currentStatus = STATUS_SUCCESS;
            this.formData = formData;
          })
          .catch(err => {
            this.uploadError = err.response;
            this.currentStatus = STATUS_FAILED;
          });
    },

    upload(fieldName, fileList) {
      // handle file changes
      const formData = new FormData();

      if (!fileList.length) return;

      // append the files to FormData
      Array
          .from(Array(fileList.length).keys())
          .map(x => {
            formData.append(fieldName, fileList[x], fileList[x].name);
          });

      // save it
      this.save(formData);
    },
  },
  computed: {
    isInitial() {
      return this.currentStatus === STATUS_INITIAL;
    },

    isSaving() {
      return this.currentStatus === STATUS_SAVING;
    },

    isSuccess() {
      return this.currentStatus === STATUS_SUCCESS;
    },

    isFailed() {
      return this.currentStatus === STATUS_FAILED;
    }
  }
}
</script>

<style scoped>

</style>

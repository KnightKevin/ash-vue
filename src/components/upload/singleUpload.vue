<template>
  <div>
    <el-upload
      class="upload-demo"
      multiple
      :limit="10"
      :file-list="fileList"
      :http-request="httpRequest"
      :on-preview="onPreview"
      :show-file-list="showFileList"
      action=""
    >
      <el-button size="small" type="primary">点击上传</el-button>
      <div slot="tip" class="el-upload__tip">
        只能上传jpg/png文件，且不超过500kb
      </div>
    </el-upload>
    <el-dialog :visible.sync="dialogVisible" :append-to-body="true">
      <img width="100%" :src="fileList[0].url" alt="" />
    </el-dialog>
  </div>
</template>

<script>
import { getUUID } from "@/utils";
export default {
  name: 'singleUpload',
  props: {
    value: String,
  },
  computed: {
    imageUrl() {
      console.log('fasdfasdfasdfasdfasdfasdf', this.value)
      return this.value;
    },
    imageName() {
      if (this.value != null && this.value !== "") {
        return this.value.substr(this.value.lastIndexOf("/") + 1);
      } else {
        return "";
      }
    },
    fileList() {
      console.log('fileList', this.imageUrl)
      return [
        {
          name: this.imageName,
          url: this.imageUrl,
        },
      ];
    },
    showFileList: {
      get: function() {
        return this.value != ''
      },
      set: function (newValue) {

      }
    }
  },
  data() {
    return {
      dialogVisible:false
    };
  },
  methods: {
    onPreview(){
      this.dialogVisible = true
    },
    httpRequest(data) {
      let { file } = data;

      console.log("file", data);

      // 请求服务器，获取minio的请求的地址
      let uploadName = getUUID() + file.name;
      this.$http
        .get(`/third-service/minio/getPresignedObjectUrl?name=${uploadName}`)
        .then(({ data }) => {
          // 获取到minio返回的上传地址
          let uploadUrl = data.data.url;

          // 解析出上传成功后文件的地址
          let url = uploadUrl.substr(0, uploadUrl.indexOf("?"));

          this.$http.put(uploadUrl, file).then((data) => {
            this.fileList.pop();
            this.fileList.push({ name: uploadName, url: url });
            this.$emit("input", url)
            this.showFileList = true
          });
        });
    },
  },
};
</script>
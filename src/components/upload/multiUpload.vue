<template>
  <div>
    <el-upload
      class="upload-demo"
      multiple
      :list-type="listType"
      :file-list="fileList"
      :http-request="httpRequest"
      :on-remove="onRemove"
      :on-preview="onPreview"
      :limit="maxCount"
      :show-file-list="showFileList"
      :on-exceed="onExceed"
      action=""
    >
      <i slot="default" class="el-icon-plus"></i>
      <div slot="file" slot-scope="{file}">
        <img
          class="el-upload-list__item-thumbnail"
          :src="file.url" alt=""
        >
        <span class="el-upload-list__item-actions">
          <span
            class="el-upload-list__item-preview"
            @click="handlePictureCardPreview(file)"
          >
            <i class="el-icon-zoom-in"></i>
          </span>
          <span
            v-if="!disabled"
            class="el-upload-list__item-delete"
            @click="handleDownload(file)"
          >
            <i class="el-icon-download"></i>
          </span>
          <span
            v-if="!disabled"
            class="el-upload-list__item-delete"
            @click="handleRemove(file)"
          >
            <i class="el-icon-delete"></i>
          </span>
        </span>
      </div>
    </el-upload>
    <el-dialog :visible.sync="dialogVisible">
      <img width="100%" :src="dialogImageUrl" alt="">
    </el-dialog>
  </div>
</template>

<script>
import { getUUID } from "@/utils";
export default {
  name: 'multiUpload',
  props: {
    value: Array,
    maxCount: {
      type: Number,
      default: 30
    },
    listType: {
      type: String,
      default: "picture-card"
    },
    showFile: {
      type: Boolean,
      default: true
    }
  },
  computed: {
    fileList() {
      let fileList = [];
      for (let i = 0; i < this.value.length; i++) {
        fileList.push({ url: this.value[i] });
      }

      return fileList;
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
      dialogImageUrl: '',
      dialogVisible: false,
      disabled: false
    };
  },
  methods: {
    emitInput(fileList) {
      let value = [];
      for (let i = 0; i < fileList.length; i++) {
        value.push(fileList[i].url);
      }
      this.$emit("input", value);
    },
    onPreview(file){
      this.dialogVisible = true;
      this.dialogImageUrl = file.url;
    },
    onRemove(file, fileList) {
      this.emitInput(fileList);
    },
    httpRequest(data) {
      let { file } = data;

      console.log("file", data);

      // 请求服务器，获取minio的请求的地址
      let uploadName = getUUID() + file.name;

      this.$http({
              url: this.$http.adornUrl(`/third-service/minio/getPresignedObjectUrl?name=${uploadName}`),
              method: "get"
            }).then(({ data }) => {
                // 获取到minio返回的上传地址
                let uploadUrl = data.url;

                // 解析出上传成功后文件的地址
                let url = uploadUrl.substr(0, uploadUrl.indexOf("?"));

                this.fileList.push({
                  name: uploadName,
                  url: url
                })

                this.$http
                // ({
                //     url: uploadUrl,
                //     method: "put",
                //     data:{file}
                //   })
                  .put(uploadUrl, file, {headers:{"Content-Type":"multiple/form-data"}})
                  .then((data) => {
                  this.fileList.pop();
                  this.fileList.push({ name: uploadName, url: url });
                  //  this.$emit("input", url)
                  this.showFileList = true

                  this.emitInput(this.fileList);

                });
        });
    },
    onExceed() {
      this.$message({
        message: "最多只能上传" + this.maxCount + "张图片",
        type: "warning",
        duration: 1000
      });
    }
  },
};
</script>
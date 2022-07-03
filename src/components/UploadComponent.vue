<template>
  <a-layout style="marginbottom: 25px">
    <a-row>
      <a-col :span="12" :offset="6">
        <a-upload-dragger
          v-model:fileList="fileList"
          name="file"
          :multiple="false"
          :action="UPLOAD_URL"
          :before-upload="beforeUpload"
          @change="handleChange"
          @drop="handleDrop"
        >
          <p class="ant-upload-drag-icon">
            <inbox-outlined></inbox-outlined>
          </p>
          <p class="ant-upload-text">
            Click or drag file to this area to upload
          </p>
          <p class="ant-u¡pload-hint">
            Support for a single or bulk upload. Strictly prohibit from
            uploading company data or other band files
          </p>
        </a-upload-dragger>
      </a-col>
    </a-row>
  </a-layout>
</template>
<script>
import { InboxOutlined } from "@ant-design/icons-vue";
import { defineComponent, ref } from "@vue/runtime-core";
import { message, Upload } from "ant-design-vue";
import { UPLOAD_URL } from "@/commons/api";

export default defineComponent({
  components: {
    InboxOutlined,
  },
  emits: ["upload-success"],
  setup(props, { emit }) {

    const handleChange = ({ file }) => {
      if (file.status === "done") {
        emit("upload-success");
      }
      if (file.status === "error") {
        message.error(`${file.name} uploads failed.`);
      }
    };

    const beforeUpload = (file) => {
      const isCSV = file.type === "text/csv";

      if (!isCSV) {
        message.error(`${file.name} is not a csv file`);
      }

      return isCSV || Upload.LIST_IGNORE;
    };

    const handleDrop = (event) => {
      console.log(event);
    };

    return {
      handleDrop,
      fileList: ref([]),
      handleChange,
      beforeUpload,
      UPLOAD_URL,
    };
  },
});
</script>

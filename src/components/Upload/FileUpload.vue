<template>

  <div class="upload-container">
    <a-modal :visible="previewVisible" :footer="null" @cancel="handleCancel">
      <img alt="example" style="width: 100%" :src="preview" />
    </a-modal>

    <a-upload
      name="file"
      accept="*"
      class="file-uploader"
      :action="action"
      :headers="headers"
      :multiple="true"
      :showUploadList="false"
      :remove="handleRemove"
      @before-upload="beforeUpload"
      @preview="handlePreview"
      @change="handleChange"
    >

      <a-button v-if="this.limit === 0 || inputValue.length < this.limit">
        <a-icon :type="loading ? 'loading' : 'upload'" />
      </a-button>

    </a-upload>
    <draggable v-model="inputValue" group="people" @start="drag=true" @end="drag=false" @change="draggableChange">
      <transition-group>
        <div :key="item + index" v-for="(item, index) in inputValue" class="ant-upload-list ant-upload-list-text">
          <div class="ant-upload-list-item ant-upload-list-item-done">
            <div class="ant-upload-list-item-info">
              <span>
                <i class="anticon anticon-paper-clip" style="top:2px"><a-icon type="paper-clip" /></i>
                <a target="_blank" rel="noopener noreferrer" class="ant-upload-list-item-name">
                  {{ item }}
                </a>
              </span>
            </div>
            <a-popconfirm title="确定要删除?" @confirm="handleRemove(item)" okText="是" cancelText="否">
              <i class="anticon anticon-close" style="top:2px">
                <a-icon type="close"/>
              </i>
            </a-popconfirm>
          </div>
        </div>
      </transition-group>
    </draggable>
  </div>
</template>

<script>
import Vue from 'vue'
import { UPLOAD_FILE_URL } from '@/modules/storage/api'
import { ACCESS_TOKEN } from '@/store/mutation-types'
import draggable from 'vuedraggable'
import _ from 'lodash'

export default {
  name: 'FileUpload',
  components: {
    draggable
  },
  props: {
    value: {
      type: Array,
      default: function () {
        return []
      }
    },
    fileSize: {
      type: Number,
      default: 1024 * 1024 * 3
    },
    limit: {
      type: Number,
      default: 0
    }
  },
  data () {
    const vm = this

    if (_.isString(vm.value) || _.isNumber(vm.value)) {
      vm.value = [vm.value + '']
    }

    return {
      loading: false,
      action: UPLOAD_FILE_URL,
      headers: {
        Authorization: 'Bearer ' + Vue.ls.get(ACCESS_TOKEN)
      },
      previewVisible: false,
      preview: '',
      inputValue: vm.value || []
    }
  },
  created: function () {
  },
  computed: {
  },
  watch: {
    'value': function (val) {
      this.inputValue = val
    }
  },
  methods: {
    emitInput (val) {
      this.$emit('input', val)
      this.$emit('change', val)
    },

    beforeUpload (file, fileList) {
      if (file.size > this.fileSize) {
        this.$message.warning(`请选择小于${this.limit / 1024 / 1024}M的图片`)

        return false
      }

      if (this.inputValue.length > this.limit) {
        return false
      }

      return true
    },

    handleCancel () {
      this.previewVisible = false
    },

    handlePreview (url) {
      this.preview = url
      this.previewVisible = true
    },

    handleChange ({ fileList, file, event }) {
      if (file.size > this.fileSize) {
        return
      }

      if (file.status === 'uploading') {
        this.loading = true
        return
      }

      if (file.status === 'done') {
        if (
          file.response !== undefined &&
          file.response.data !== undefined &&
          file.response.data.path !== undefined
        ) {
          this.inputValue.push(file.response.data.path)
          this.loading = false
          this.emitInput(this.inputValue)
        }
      }
    },

    handleRemove (url) {
      const index = this.inputValue.indexOf(url)
      const newInputValue = this.inputValue.slice()
      newInputValue.splice(index, 1)
      this.inputValue = newInputValue
      this.emitInput(this.inputValue)
    },

    draggableChange () {
      this.emitInput(this.inputValue)
    }
  }
}
</script>

<style lang="less" scoped>

</style>

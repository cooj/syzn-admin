<template>
  <a-drawer
    :title="title"
    :closable="false"
    :maskClosable="false"
    width="60%"
    :visible="visible"
    :body-style="{ paddingBottom: '80px' }"
    @close="onClose"
  >
    <a-form-model
      ref="ruleForm"
      :model="form"
      :label-col="labelCol"
      :wrapper-col="wrapperCol"
      :key="visible"
    >
      <a-form-model-item label="视频所属">
        <a-select
          allowClear
          v-model="form.productId"
          show-search
          placeholder="视频所属"
          option-filter-prop="children"
          style="width: 50%"
          :filter-option="(input,option) => option.componentOptions.children[0].text.toLowerCase().indexOf(input.toLowerCase()) >= 0"
          @change="handleVideoChange"
        >
          <a-select-option :value="item.id" v-for="item of parent.productData" :key="item.id">{{ item.title }}</a-select-option>
        </a-select>
      </a-form-model-item>
      <a-form-model-item label="视频链接">
        <a-input v-model="form.videoUrl" placeholder="视频链接"></a-input>
        <!-- <a-button type="primary" @click="handleAddClick">添加视频链接</a-button>
        <a-row :gutter="[20,20]" style="width: 60%;" v-for="(item,index) of videoUrlList" :key="index">
          <a-col :span="18">
            <a-input v-model="videoUrlList[index]"></a-input>
          </a-col>
          <a-col :span="4">
            <a-button type="danger" icon="minus" @click="handleDeleteClick(index)"></a-button>
          </a-col>
        </a-row> -->
      </a-form-model-item>
      <!-- <a-form-model-item label="视频封面图片">
        <a-upload
          list-type="picture-card"
          :file-list="fileList"
          :remove="hanldeImgRemove"
          @preview="handlePreview"
          :customRequest="handleUploadImgClick"
          class="upload-half"
          accept="image/*"
        >
          <div class="custom-btn" v-if="fileList.length<1">
            <a-icon type="ant-cloud" style="font-size: 26px" />
            <div class="ant-upload-text">添加视频封面(比例16:9)</div>
          </div>
        </a-upload>
      </a-form-model-item> -->
      <a-form-model-item label="是否推荐到首页">
        <a-select v-model="form.homeRecommend" placeholder="是否推荐到首页" style="width:50%;">
          <a-select-option :value="1">是</a-select-option>
          <a-select-option :value="0">否</a-select-option>
        </a-select>
      </a-form-model-item>
      <a-form-model-item label="排序">
        <a-input-number v-model="form.orderNum" placeholder="排序"></a-input-number>
      </a-form-model-item>
    </a-form-model>
    <div
      :style="{
        position: 'absolute',
        right: 0,
        bottom: 0,
        width: '100%',
        borderTop: '1px solid #e9e9e9',
        padding: '10px 16px',
        background: '#fff',
        textAlign: 'right',
        zIndex: 99999,
      }"
    >
      <a-button :style="{ marginRight: '8px' }" @click="onClose"> 关闭 </a-button>
      <a-button type="primary" @click="onSubmit"> 提交 </a-button>
    </div>
    <a-modal :visible="previewVisible" :footer="null" @cancel="previewVisible = false" width="60%">
      <img alt="example" style="width: 100%" :src="previewImage | filterUrl" v-if="judgeSourceType(previewImage)" />
      <video :src="previewImage | filterUrl" style="width: 100%" controls v-else></video>
    </a-modal>
  </a-drawer>
</template>

<script>
import { uploadFile,deleteFile } from '@/api/common'
import { getFileName } from '@/utils/util'
import { showMessage } from '@/utils/mixins'
import { addVideoItem,updateVideoItem } from '@/api/manage'

export default {
  mixins: [showMessage],
  props: {
    parent: {
      type: Object,
      default: () => ({})
    }
  },
  data() {
    return {
      tabkey: '1',
      title: '',
      labelCol: { span: 3 },
      wrapperCol: { span: 21 },
      form: {
        productId: undefined,
        videoUrl: undefined,
        // videoImageUrl: undefined,
        orderNum: 0,
        homeRecommend: undefined
      },
      videoUrlList: [],
      visible: false,
      status: undefined,
      fileList: [],
      previewVisible: false,
      previewImage: '',
      id: undefined
    }
  },
  methods: {
    handleVideoChange(val) {
      if(!val) {
        this.form.productId = 0
      }else {
        this.form.productId = val
      }
    },
    //删除
    handleDeleteClick(index) {
      this.videoUrlList.splice(index,1)
    },
    // 添加视频链接
    handleAddClick() {
      this.videoUrlList.push('')
    },
    //判断资源类型
    judgeSourceType(url) {
      if(/\.(png|jpg|gif|jpeg|webp)$/.test(url)) {
        return true
      }else {
        return false
      }
    },
    // 删除视频封面
    hanldeImgRemove(file) {
      deleteFile({filePath: file.url}).then(res => {
        if(res.code == '0') {
          // const _index = this.fileList.findIndex(item => item == file.url)
          this.showMessage(res,() => {
            this.fileList = []
            this.form.videoImageUrl = ''
            // this.fileList.splice(_index,1)
            // const _temp = []
            // this.fileList.forEach(item => _temp.push(item.url))
            // this.form.videoImageUrl = _temp.join(',')
          })
        }
      })
    },
   
    // 预览图片
    handlePreview(file) {
      this.previewVisible = true
      this.previewImage = file.url
    },
    //自定义上传图片
    handleUploadImgClick(file) {
      const _this = this
      const formdata = new FormData()
      formdata.append('file', file.file)
      uploadFile(formdata).then(res => {
        if(res.code == '0') {
          file.url = process.env.VUE_APP_API_ORIGIN+res.data
          file.uid = Math.random()
          file.fileame = getFileName( res.data )
          _this.fileList.push(file)
          // _this.form.images.push(res.data)
          _this.form.videoImageUrl = res.data
        }else {
          _this.$message.error(res.message)
        }
      })
    },
    // 关闭
    onClose() {
      this.visible = false
      this.fileList = []
      this.videoUrlList = []
      this.status = undefined
      this.id = undefined
      for(const key in this.form) {
        if(key == 'orderNum') {
          this.form[key] = 0
        }else {
          this.form[key] = undefined
        }
      }
    },
    // 提交
    onSubmit() {
      this.$refs.ruleForm.validate((vaild) => {
        if (vaild) {
          // this.form.videoUrl = this.videoUrlList.join(',')
          // const _arrs = []
          // this.fileList.forEach(item =>  _arrs.push(item.url))
          // this.form.videoImageUrl =_arrs.join(',')
          this.form.homeRecommend = this.form.homeRecommend == 1 ? true : 0
          if(this.status == 1) {
            addVideoItem(this.form).then(res => {
              this.showMessage(res,() => {
                this.onClose()
                this.parent.getList()
              })
            })
          }else {
            updateVideoItem({
              id: this.id,
              ...this.form
            }).then(res => {
              this.showMessage(res,() => {
                this.onClose()
                this.parent.getList()
              })
            })
          }
        } else {
          return false
        }
      })
    },
  },
}
</script>

<style lang="less">
.ant-upload.ant-upload-disabled {
  cursor: pointer !important;
}
.ant-upload {
  position: relative;
  width: 100px !important;
  height: 100px !important;
}
.custom-btn {
  width: 100px;
  height: 100px;
  box-sizing: border-box;
  display: flex;
  align-items: center;
  flex-direction: column;
  justify-content: center;
  position: absolute;
  top: 0;
  left: 0;
  z-index: 99;
}
</style>
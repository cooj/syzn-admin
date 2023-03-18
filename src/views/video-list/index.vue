<template>
  <page-header-wrapper :title="false">
    <a-card class="content-card">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="48">
            <a-col :md="6" :sm="24">
              <a-form-item label="视频所属">
                <a-select
                  allowClear
                  v-model="queryParam.productId"
                  show-search
                  placeholder="视频所属"
                  option-filter-prop="children"
                  style="width: 100%"
                  :filter-option="(input,option) => option.componentOptions.children[0].text.toLowerCase().indexOf(input.toLowerCase()) >= 0"
                >
                  <a-select-option :value="item.id" v-for="item of productData" :key="item.id">{{ item.title }}</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :md="!advanced && 6 || 24" :sm="24">
              <span class="table-page-search-submitButtons" :style="advanced && { float: 'right', overflow: 'hidden' } || {} ">
                <a-button type="primary" @click="getList">查询</a-button>
                <a-button style="margin-left: 8px" @click="() => this.queryParam = {}">重置</a-button>
              </span>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <a-button type="primary" icon="plus" @click="handleAddClick">添加视频</a-button>
      <a-table 
        style="margin-top: 20px;"
        size="middle"
        :columns="columns" 
        :data-source="data" 
        :rowKey="r=>r.id"
        :loading="loading"
        :pagination="pagination"
      >
        <template slot="homeRecommend" slot-scope="t,r">
          <a-tag color="green" v-if="r.homeRecommend">是</a-tag>
          <a-tag color="red" v-else>否</a-tag>
        </template>
        <template slot="productName" slot-scope="t,r">
          <a-tag color="green" v-if="r.productName">{{ r.productName }}</a-tag>
          <a-tag v-else>无</a-tag>
        </template>
        <template slot="action" slot-scope="r">
          <a @click="handleEditorClick(r)">修改</a>
          <a-divider type="vertical"></a-divider>
          <a-popconfirm
            title="您确定要删除吗?"
            ok-text="确定"
            cancel-text="取消"
            @confirm="handleDeleteClick(r)"
          >
            <a>删除</a>
          </a-popconfirm>
        </template>
      </a-table>
    </a-card>
    <!-- 新增产品 -->
    <CreateModal ref="createModal" :parent="this"></CreateModal>
  </page-header-wrapper>
</template>

<script>
import { getVideoList,deleteVideoItem,getProductList } from '@/api/manage'
import { showMessage,toggleQuery } from '@/utils/mixins'
import { getFileName } from '@/utils/util'
import CreateModal from './modules/CreateModal.vue'

export default {
  name: 'VideoList',
  mixins: [showMessage,toggleQuery],
  components: {
    CreateModal
  },
  data() {
    return {
      queryParam: {},
      loading: false,
      columns: [
        { title: '序号', customRender: (t,r,i) => `${i+1}`, width: 70, align: 'center' },
        { title: '视频封面图片', dataIndex: 'videoUrl',key: 'videoUrl', width: 450 },
        { title: '是否推荐', scopedSlots: {customRender: 'homeRecommend'}},
        { title: '所属产品',scopedSlots: {customRender: 'productName'}},
        { title: '操作', scopedSlots: { customRender: 'action' }, width: 120, fixed: 'right', align: 'center' }
      ],
      data: [],
      pagination: {
        total: 0,
        pageSize: 10,
        showTotal: total => `共 ${total} 条`
        // showSizeChanger: true,
        // pageSizeOptions: ['10', '20', '30', '40']
      },
      productData: [],
    }
  },
  methods: {
    // 增加
    handleAddClick() {
      this.$refs.createModal.status = 1
      this.$refs.createModal.title = '新增视频'
      this.$refs.createModal.visible = true
    },
    // 修改
    handleEditorClick(r) {
      const _form = this.$refs.createModal.form
      for(const key in _form) {
        if(key == 'productId') {
          _form[key] = Number(r[key])
        }else if(key == 'homeRecommend') {
          _form[key] = r[key] ? 1 : 0
        }else {
          _form[key] = r[key]
        }
      }
      // if(r.videoImageUrl) {
      //   this.$refs.createModal.fileList.push({
      //     uid: Math.random(),
      //     url: r.videoImageUrl.indexOf('http')!=-1 || r.videoImageUrl.indexOf('https')!=-1 ? r.videoImageUrl:process.env.VUE_APP_API_ORIGIN+r.videoImageUrl,
      //     name: getFileName(r.videoImageUrl)
      //   })
      // }
      this.$refs.createModal.status = 2
      this.$refs.createModal.id = r.id
      this.$refs.createModal.title = '修改视频'
      this.$refs.createModal.visible = true
    },
    // 删除
    handleDeleteClick(r) {
      deleteVideoItem({id: r.id}).then(res => {
        this.showMessage(res,this.getList)
      })
    },
    // 列表
    getList() {
      this.loading = true
      getVideoList(this.queryParam).then(res => {
        this.loading = false
        if(res.code == '0') {
          this.data = res.data
        }
      })
    },
     //获取系列树结构
    getProductList() {
      getProductList().then(res => {
        if(res.code == '0') {
          this.productData = res.data
        }
      })
    },
  },
  mounted() {
    this.getList()
    this.getProductList()
  }
}
</script>

<style lang="less">
.table-img {
  width: 160px;
  height: 90px;
}
</style>


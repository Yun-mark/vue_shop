<template>
    <div>
        <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>商品分类</el-breadcrumb-item>
        </el-breadcrumb>
        <!--卡片-->
        <el-card>
            <el-row>
                <el-col>
                    <el-button type="primary" @click="showAddCateDialog"
                    width="50%" @close="addCateDialogClosed">添加分类</el-button>
                </el-col>
            </el-row>
            <!--表格-->
             <tree-table class="treeTable" :data="catelist" :columns="columns"
              :selection-type="false" :expand-type="false"
              show-index index-text="#" border :show-row-hover="false">
              <template slot="isok" slot-scope="scope">
                  <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color : lightgreen"></i>
                  <i class="el-icon-error" v-else style="color : red"></i>
                  </template>
              <template slot="order" slot-scope="scope">
                <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
                <el-tag type="success" size="mini" v-else-if="scope.row.cat_level === 1">二级</el-tag>
                <el-tag type="warning" size="mini" v-else>三级</el-tag>
              </template>
              <template slot="opt" slot-scoped>
                  <el-button type="primary" size="mini" icon="el-icon-edit">编辑</el-button>
                  <el-button type="danger" size="mini" icon="el-icon-delete">删除</el-button>
              </template>
              </tree-table>
            <!--分页-->
            <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
            :current-page="querInfo.pagenum" :page-sizes="[3, 5, 10, 15]" :page-size="querInfo.pagesize"
            layout="total, sizes, prev, pager, next, jumper" :total="total"></el-pagination>
        </el-card>

        <!--添加分类的对话框-->
        <el-dialog title="添加分类" :visible.sync="addCateDialogVisible" width="50%">
          <!--添加分类的表单-->
          <el-form :model="addCateForm" :rules="addCateFormRules"
          ref="addCateFormRef" label-width="100px">
             <el-form-item label="分类名称：" prop="cate_name">
                 <el-input v-model="addCateForm.cat_name"></el-input>
             </el-form-item>
             <el-form-item label="父级分类：">
                 <el-cascader expand-trigger="hover" :options="parentCateList"
                 :props="cascaderProps" v-model="selectKeys" @change="parentCateChanged"
                 clearable change-on-select></el-cascader>
             </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click=" addCateDialogVisible = false">取消</el-button>
            <el-button type="primary" @click=" addCate">确定</el-button>
        </span>
        </el-dialog>
    </div>
</template>
<script>
export default {
  data() {
    return {
      querInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      catelist: [],
      total: 0,
      columns: [{
        label: '分类名称',
        prop: 'cat_name'
      },
      {
        label: '是否有效',
        type: 'template',
        template: 'isok'
      },
      {
        label: '排序',
        type: 'template',
        template: 'order'
      },
      {
        label: '操作',
        type: 'template',
        template: 'opt'
      }],
      // 控制添加分类对话框的显示与隐藏
      addCateDialogVisible: false,
      addCateForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      addCateFormRules: {
        cat_name: [
          {
            required: true, message: '请输入分类名称', trigger: 'blur'
          }
        ]
      },
      parentCateList: [],
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      selectedKeys: []
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.querInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品列表失败！')
      }
      console.log(res.data)
      this.catelist = res.data.result
      this.total = res.data.total
    },
    handleSizeChange(newSize) {
      this.querInfo.pagesize = newSize
      this.getCateList()
    },
    handleCurrentChange(newPage) {
      this.querInfo.pagenum = newPage
      this.getCateList()
    },
    // 点击按钮，展示添加分类的对话框
    showAddCateDialog() {
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级数据类型失败!')
      }
      console.log(res.data)
      this.parentCateList = res.data
    },
    parentCateChanged() {
      console.log(this.selectedKeys)

      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectedKeys[
          this.selectedKeys.length - 1
        ]
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addCateForm)

        if (res.meta.status !== 200) {
          return this.$message.error('添加分类失败！')
        }
        this.$message.success('添加分类成功!')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    }
  }
}
</script>
<style scoped>
.treeTable{
    margin-top: 15px;
}
.el-cascader{
    width: 100%;
}
</style>

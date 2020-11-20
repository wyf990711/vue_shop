<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card class="box-card">
      <el-row :gutter="20">
        <el-col :span="7"
          ><div class="grid-content bg-purple">
            <el-input
              placeholder="请输入内容"
              clearable
              v-model="queryInfo.query"
              @clear="getGoodsList"
            >
              <el-button
                slot="append"
                icon="el-icon-search"
                @click="getGoodsList"
              ></el-button>
            </el-input></div
        ></el-col>
        <el-col :span="4"
          ><div class="grid-content bg-purple">
            <el-button type="primary" @click="GoAddGoodsPage"
              >添加商品</el-button
            >
          </div></el-col
        >
      </el-row>
      <el-table :data="GoodsList" border stripe class="el-table" height="450">
        <el-table-column type="index" label="#"> </el-table-column>
        <el-table-column prop="goods_name" label="商品名称"> </el-table-column>
        <el-table-column prop="goods_price" label="商品价格" width="95px">
        </el-table-column>
        <el-table-column prop="goods_weight" label="商品重量" width="90px">
        </el-table-column>
        <el-table-column prop="add_time" label="创建时间" width="160px">
          <template slot-scope="scope">
            {{ scope.row.add_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="130px">
          <template slot-scope="scope">
            <el-button
              type="primary"
              size="mini"
              class="el-icon-edit"
              @click="ShowEditDialog(scope.row)"
            ></el-button>
            <el-button
              type="danger"
              size="mini"
              class="el-icon-delete"
              @click="DeleteGoods(scope.row)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 6, 9, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        background
      >
      </el-pagination>
    </el-card>
    <el-dialog
      title="编辑商品"
      :visible.sync="EditDialogVisible"
      width="50%"
      @close="CloseEditDialog"
    >
      <el-form
        :model="EditForm"
        :rules="EditFormRules"
        ref="EditFormRef"
        label-width="100px"
      >
        <el-form-item label="商品名称" prop="goods_name">
          <el-input v-model="EditForm.goods_name"></el-input>
        </el-form-item>
        <el-form-item label="商品价格" prop="goods_price">
          <el-input v-model="EditForm.goods_price"></el-input>
        </el-form-item>
        <el-form-item label="商品重量" prop="goods_weight">
          <el-input v-model="EditForm.goods_weight"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="EditDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="EditSubGoods">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      GoodsList: [],
      total: 0,
      queryInfo: {
        query: "",
        pagenum: 1,
        pagesize: 10,
      },
      EditForm: {
        goods_name: "",
        goods_price: 0,
        goods_weight: 0,
        goods_id: 0,
      },
      EditFormRules: {
        goods_name: [
          { required: true, message: "商品名称不能为空", trigger: "blur" },
        ],
        goods_price: [
          { required: true, message: "商品价格不能为空", trigger: "blur" },
        ],
        goods_weight: [
          { required: true, message: "商品重量不能为空", trigger: "blur" },
        ],
      },
      EditDialogVisible: false,
    };
  },
  methods: {
    // 获取商品列表
    async getGoodsList() {
      const res = await this.$http.get("goods", { params: this.queryInfo });
      if (res.data.meta.status !== 200) {
        return this.$message.error("获取商品列表失败！");
      }
      this.total = res.data.data.total;
      this.GoodsList = res.data.data.goods;
    },
    // 改变 每页显示多少条
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getGoodsList();
    },
    // 切换到当前页码
    handleCurrentChange(newNum) {
      this.queryInfo.pagenum = newNum;
      this.getGoodsList();
    },
    // 根据id 删除商品
    async DeleteGoods(row) {
      const res = await this.$confirm(
        "此操作将永久删除该商品, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((err) => err);
      if (res !== "confirm") {
        return this.$message.info("取消了删除商品操作！");
      }
      const result = await this.$http.delete(`goods/${row.goods_id}`);
      if (result.data.meta.status !== 200) {
        return this.$message.error("删除商品失败！");
      }
      this.$message.success("成功该删除商品！");
      this.getGoodsList();
    },
    // 跳转到 添加商品页面
    GoAddGoodsPage() {
      this.$router.push("/goods/add");
    },
    // 展示编辑商品的对话框
    ShowEditDialog(row) {
      this.QueryGoods(row);
      this.EditDialogVisible = true;
    },
    // 根据 ID 查询商品
    async QueryGoods(row) {
      const res = await this.$http.get(`goods/${row.goods_id}`);
      if (res.data.meta.status !== 200) {
        this.$message.error("查询商品失败！");
      }
      this.EditForm = res.data.data;
    },
    // 编辑提交商品
    EditSubGoods() {
      this.$refs.EditFormRef.validate(async (valid) => {
        if (!valid) {
          this.$message.error("请添加商品参数！");
        } else {
          const res = await this.$http.put(
            `goods/${this.EditForm.goods_id}`,
            this.EditForm
          );

          if (res.data.meta.status !== 200) {
            this.$message.error("修改商品失败！");
          }
          this.getGoodsList();
          this.$message.success("成功修改商品！");
          this.EditDialogVisible = false;
        }
      });
    },
    // 编辑商品对话框关闭时，重置表单校验
    CloseEditDialog() {
      this.$refs.EditFormRef.resetFields();
    },
  },

  created() {
    this.getGoodsList();
  },
};
</script>

<style scoped>
.el-table {
  margin-top: 15px;
}
</style>

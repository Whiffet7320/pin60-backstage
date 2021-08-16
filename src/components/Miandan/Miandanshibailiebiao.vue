<template>
  <div class="index">
    <div class="nav1">
      <div class="tit1">免单失败列表</div>
    </div>
    <div class="myForm">
      <el-form
        @submit.native.prevent
        :inline="true"
        :model="formInline"
        class="demo-form-inline"
      >
        <el-form-item label="关键词">
          <el-input
            v-model="formInline.search"
            placeholder="请输入关键词"
            @keyup.enter.native="onSearch"
          ></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="danger" @click="onSearch">查询</el-button>
        </el-form-item>
        <el-form-item>
          <el-button @click="onReset">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
    <div class="myTable">
      <el-table :data="tableData" style="width: 100%">
        <el-table-column
          :show-overflow-tooltip="true"
          prop="realname"
          label="客户名称"
          width="120"
        >
        </el-table-column>
        <el-table-column
          :show-overflow-tooltip="true"
          prop="mobile"
          label="联系方式"
        >
        </el-table-column>
        <el-table-column
          :show-overflow-tooltip="true"
          prop="order_no"
          label="订单编号"
        >
        </el-table-column>
        <el-table-column
          :show-overflow-tooltip="true"
          prop="goods_title"
          label="商品名称"
        >
        </el-table-column>
        <el-table-column
          :show-overflow-tooltip="true"
          prop="goods_pic"
          label="商品图"
        >
          <template slot-scope="scope">
            <img
              style="width: 40px; height: 40px; cursor: pointer"
              :src="scope.row.goods_pic"
            />
          </template>
        </el-table-column>
        <el-table-column
          :show-overflow-tooltip="true"
          prop="total_money"
          label="支付金额"
        >
        </el-table-column>
        <el-table-column
          :show-overflow-tooltip="true"
          prop="refund_money"
          label="免单金额"
        >
        </el-table-column>
        <el-table-column
          :show-overflow-tooltip="true"
          prop="update_time"
          label="退款处理时间"
        >
        </el-table-column>
        <el-table-column fixed="right" label="操作" width="100">
          <template slot-scope="scope">
            <el-button @click="tuikuan(scope.row)" type="text" size="small"
              >退款</el-button
            >
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        class="fenye"
        @size-change="this.handleSizeChange"
        @current-change="this.handleCurrentChange"
        :current-page="this.miandanshibaiPage"
        :page-sizes="[10, 15, 20, 50]"
        :page-size="10"
        layout="total, sizes, prev, pager, next, jumper"
        :total="this.total"
      >
      </el-pagination>
    </div>
  </div>
</template>

<script>
import { mapState } from "vuex";
export default {
  computed: {
    ...mapState(["miandanshibaiPage", "miandanshibaiPageSize"]),
  },
  data() {
    return {
      formInline: {
        search: "",
      },
      tableData: [],
      total: null,
    };
  },
  created() {
    this.getData();
  },
  watch: {
    page: function (page) {
      this.$store.commit("miandanshibaiPage", page);
      this.getData();
    },
    pageSize: function (pageSize) {
      this.$store.commit("miandanshibaiPageSize", pageSize);
      this.getData();
    },
  },
  methods: {
    async getData() {
      const res = await this.$api.freeorder_refundlist(
        sessionStorage.getItem("user_name"),
        sessionStorage.getItem("user_pass"),
        this.miandanshibaiPage,
        this.miandanshibaiPageSize
      );
      console.log(res);
      this.tableData = res.list;
      this.total = res.total;
      console.log(this.tableData);
    },
    // 退款
    async tuikuan(row) {
      const res = await this.$api.freeorder_refundsubmit(
        sessionStorage.getItem("user_name"),
        sessionStorage.getItem("user_pass"),
        row.id
      );
      console.log(res)
      if (res.result == 1) {
        this.$message({
          message: res.msg,
          type: "success",
        })
        this.getData()
      }else{
        this.$message.error(res.msg);
      }
    },
    onSearch() {
      this.getData();
    },
    // 重置
    onReset() {
      this.$store.commit("miandanshibaiPage", 1);
      this.formInline.search = "";
      this.getData();
    },
    // 分页
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`);
      this.$store.commit("miandanshibaiPageSize", val);
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
      this.$store.commit("miandanshibaiPage", val);
    },
  },
};
</script>

<style lang="scss" scoped>
.index {
}
.nav1 {
  width: 100%;
  height: 80px;
  opacity: 1;
  background: #ffffff;
  .tit1 {
    opacity: 1;
    font-size: 30px;
    font-family: YouSheBiaoTiHei, YouSheBiaoTiHei-Regular;
    font-weight: 400;
    text-align: left;
    line-height: 80px;
    margin-left: 40px;
    color: #ebbfcc;
  }
}
.myForm {
  margin-top: 30px;
  padding-left: 30px;
}
.myTable {
  margin-right: 30px;
}
</style>
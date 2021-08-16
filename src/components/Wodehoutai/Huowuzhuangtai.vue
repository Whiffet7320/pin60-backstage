<template>
  <div class="index">
    <div class="nav1">
      <!-- <div class="tit1">货物状态</div> -->
      <el-menu
        :default-active="activeIndex"
        class="el-menu-demo"
        mode="horizontal"
        @select="handleSelect"
        text-color="#6F7070"
        active-text-color="#ebbfcc"
      >
        <el-menu-item index="1">近3个月订单</el-menu-item>
        <el-menu-item index="2">3个月前订单</el-menu-item>
      </el-menu>
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
        <el-form-item label="下单时间">
          <el-date-picker
            value-format="yyyy-MM-dd"
            v-model="formInline.time"
            type="daterange"
            range-separator="至"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
          >
          </el-date-picker>
        </el-form-item>
        <el-form-item label="只看">
          <el-checkbox-group v-model="formInline.type" @change="onCheck">
            <el-checkbox label="待付款"></el-checkbox>
            <el-checkbox label="待发货"></el-checkbox>
            <el-checkbox label="待收货"></el-checkbox>
          </el-checkbox-group>
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
      <el-button
        type="info"
        style="margin-bottom: 10px"
        @click="exportSelectEvent"
        >导出</el-button
      >
      <vxe-table
        resizable
        highlight-current-row
        highlight-current-column
        border="inner"
        ref="xTree"
        :tree-config="{ children: 'goods' }"
        :data="tableData"
      >
        <vxe-table-column
          width="100"
          tree-node
          field="realname"
          title="客户名称"
        ></vxe-table-column>
        <vxe-table-column
          width="120"
          field="mobile"
          show-header-overflow
          show-overflow="title"
          show-footer-overflow
          title="手机号"
        ></vxe-table-column>
        <vxe-table-column
          field="orders_status"
          title="货物状态"
          width="120"
        ></vxe-table-column>
        <vxe-table-column field="paymoney" title="实付金额"></vxe-table-column>
        <vxe-table-column
          show-header-overflow
          show-overflow="title"
          show-footer-overflow
          field="goods_title"
          title="商品名称"
          width="140"
        ></vxe-table-column>
        <vxe-table-column
          field="goods_classify"
          title="类型"
        ></vxe-table-column>
        <vxe-table-column field="goods_pic" title="商品主图">
          <template slot-scope="scope">
            <img
              v-if="scope.row.goods_pic"
              style="width: 50px; height: 50px; cursor: pointer"
              :src="scope.row.goods_pic"
            />
          </template>
        </vxe-table-column>
        <vxe-table-column field="goods_num" title="数量"></vxe-table-column>
        <vxe-table-column field="goods_price" title="价格"></vxe-table-column>
        <vxe-table-column field="goods_postage" title="邮费"></vxe-table-column>
        <vxe-table-column title="操作" width="120">
          <template slot-scope="scope">
            <el-button @click="see(scope.row)" type="text" size="small"
              >查看</el-button
            >
            <el-button
              v-if="scope.row.orders_status != '待付款'"
              @click="cz(scope.row)"
              type="text"
              size="small"
              >操作</el-button
            >
          </template>
        </vxe-table-column>
      </vxe-table>
      <el-pagination
        class="fenye"
        @size-change="this.handleSizeChange"
        @current-change="this.handleCurrentChange"
        :current-page="this.huowuzhuangtaiPage"
        :page-sizes="[10, 15, 20, 50]"
        :page-size="10"
        layout="total, sizes, prev, pager, next, jumper"
        :total="this.total"
      >
      </el-pagination>
    </div>
    <el-dialog
      title="发货/完成"
      :visible.sync="dialogVisible"
      width="30%"
      :before-close="handleClose"
    >
      <el-form ref="form" :model="form" label-width="80px">
        <el-form-item label="设置状态">
          <el-select v-model="form.region">
            <el-option label="发货" value="2"></el-option>
            <el-option label="完成" value="3"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="onSubmit">立即创建</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>

<script>
import { mapState } from "vuex";
export default {
  computed: {
    ...mapState(["huowuzhuangtaiPage", "huowuzhuangtaiPageSize"]),
  },
  data() {
    return {
      activeIndex: "1",
      formInline: {
        search: "",
        type: [],
        time: [],
      },
      form: {
        region: "",
      },
      id: "",
      tableData: [],
      total: null,
      statusArr: [],
      status: "",
      dialogVisible: false,
    };
  },
  watch: {
    page: function (page) {
      this.$store.commit("huowuzhuangtaiPage", page);
      this.getData();
    },
    pageSize: function (pageSize) {
      this.$store.commit("huowuzhuangtaiPageSize", pageSize);
      this.getData();
    },
  },
  created() {
    const time = this.getLast3Month(Date.now()); //{now:"2019-07-04",last:"2019-04-04"}
    this.$set(this.formInline.time, 1, time.now);
    this.$set(this.formInline.time, 0, time.last);
    this.getData();
  },
  methods: {
    async getData() {
      const res = await this.$api.user_order(
        this.huowuzhuangtaiPage,
        this.huowuzhuangtaiPageSize,
        this.formInline.search,
        this.status,
        this.formInline.time[0],
        this.formInline.time[1]
      );
      console.log(res);
      this.tableData = res.list;
      this.total = res.total;
      console.log(this.tableData);
    },
    async exportSelectEvent() {
      const res = await this.$api.order_export(
        sessionStorage.getItem("user_name"),
        sessionStorage.getItem("user_pass"),
        this.formInline.time[0],
        this.formInline.time[1],
        this.status
      );
      console.log(res);
      if (res.result == 1) {
        this.$message({
          message: res.msg,
          type: "success",
        });
        window.open(res.down_url);
      } else {
        this.$message.error(res.msg);
      }
    },
    async onSubmit() {
      const res = await this.$api.order_setorderstatus(
        sessionStorage.getItem("user_name"),
        sessionStorage.getItem("user_pass"),
        this.id,
        this.form.region
      );
      console.log(res);
      if (res.result == 1) {
        this.$message({
          message: res.msg,
          type: "success",
        });
        this.dialogVisible = false;
        this.getData();
      }
    },
    handleSelect(val) {
      console.log(val);
      if (val == 1) {
        const time = this.getLast3Month(Date.now()); //{now:"2019-07-04",last:"2019-04-04"}
        this.$set(this.formInline.time, 1, time.now);
        this.$set(this.formInline.time, 0, time.last);
        this.getData();
      } else if (val == 2) {
        const time = this.getLast3Month(Date.now()); //{now:"2019-07-04",last:"2019-04-04"}
        this.$set(this.formInline.time, 1, time.last);
        this.$set(this.formInline.time, 0, "1998-01-01");
        this.getData();
      }
      // getLast3Month(Date.now())  //{now:"2019-07-04",last:"2019-04-04"}
    },
    handleClose() {
      this.dialogVisible = false;
    },
    onSearch() {
      this.$store.commit("huowuzhuangtaiPage", 1);
      console.log(this.formInline);
      this.getData();
    },
    // 近三个月
    getLast3Month() {
      var now = new Date();
      var year = now.getFullYear();
      var month = now.getMonth() + 1; //0-11表示1-12月
      var day = now.getDate();
      var dateObj = {};
      if (parseInt(month) < 10) {
        month = "0" + month;
      }
      if (parseInt(day) < 10) {
        day = "0" + day;
      }

      dateObj.now = year + "-" + month + "-" + day;

      if (parseInt(month) == 1) {
        //如果是1月份，则取上一年的10月份
        dateObj.last = parseInt(year) - 1 + "-10-" + day;
        return dateObj;
      }
      if (parseInt(month) == 2) {
        //如果是2月份，则取上一年的11月份
        dateObj.last = parseInt(year) - 1 + "-11-" + day;
        return dateObj;
      }
      if (parseInt(month) == 3) {
        //如果是3月份，则取上一年的12月份
        dateObj.last = parseInt(year) - 1 + "-12-" + day;
        return dateObj;
      }

      var preSize = new Date(year, parseInt(month) - 3, 0).getDate(); //开始时间所在月的总天数
      if (preSize < parseInt(day)) {
        // 开始时间所在月的总天数<本月总天数，比如当前是5月30日，在2月中没有30，则取下个月的第一天(3月1日)为开始时间
        let resultMonth =
          parseInt(month) - 2 < 10
            ? "0" + parseInt(month) - 2
            : parseInt(month) - 2;
        dateObj.last = year + "-" + resultMonth + "-01";
        return dateObj;
      }

      if (parseInt(month) <= 10) {
        dateObj.last = year + "-0" + (parseInt(month) - 3) + "-" + day;
        return dateObj;
      } else {
        dateObj.last = year + "-" + (parseInt(month) - 3) + "-" + day;
        return dateObj;
      }
    },
    // 查看详情
    see(row) {
      console.log(row);
      this.$router.push({ name: "Huowuxiangqing", params: { id: row.id } });
    },
    // 操作
    cz(row) {
      console.log(row);
      this.id = row.id;
      this.dialogVisible = true;
    },
    // 分页
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`);
      this.$store.commit("huowuzhuangtaiPageSize", val);
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
      this.$store.commit("huowuzhuangtaiPage", val);
    },
    // 重置
    onReset() {
      this.$store.commit("huowuzhuangtaiPage", 1);
      this.formInline.search = "";
      this.formInline.type = [];
      this.formInline.time = [];
      this.status = "";
      this.getData();
    },
    onCheck(val) {
      this.statusArr = [];
      val.forEach((ele) => {
        if (val.indexOf(ele) != -1) {
          if (ele == "待付款") {
            this.statusArr.push(0);
          }
          if (ele == "待发货") {
            this.statusArr.push(1);
          }
          if (ele == "待收货") {
            this.statusArr.push(2);
          }
        }
      });
      this.status = this.statusArr.toString();
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
  .el-menu-demo {
    margin-left: 20px;
    height: 100%;
    .el-menu-item {
      height: 100%;
      margin-right: 10px;
      line-height: 80px;
    }
  }
}
.myForm {
  margin-top: 30px;
  padding-left: 30px;
}
.myTable {
  margin-right: 30px;
  margin-bottom: 30px;
}
/deep/ .el-form-item {
  margin-right: 20px;
}
/deep/ .el-input__inner.el-range-editor {
  width: 300px;
}
/deep/ .el-checkbox {
  margin-right: 16px;
}
</style>
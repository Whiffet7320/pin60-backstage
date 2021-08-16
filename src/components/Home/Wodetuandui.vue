<template>
  <div class="index">
    <div class="nav1">
      <div class="tit1">我的团队</div>
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
            @keyup.enter.native="onSubmit"
          ></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="danger" @click="onSubmit">查询</el-button>
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
          prop="pic"
          label="用户头像"
        >
          <template slot-scope="scope">
            <!-- <div class="splbImg" v-for="(item, i) in scope.row.pic" :key="i"> -->
            <img
              style="width: 40px; height: 40px; cursor: pointer"
              :src="scope.row.facepic"
            />
            <!-- </div> -->
          </template>
        </el-table-column>
        <el-table-column
          :show-overflow-tooltip="true"
          prop="nickname"
          label="用户昵称"
        >
        </el-table-column>
        <el-table-column fixed="right" label="操作" width="100">
          <template slot-scope="scope">
            <el-button @click="see(scope.row)" type="text" size="small"
              >查看</el-button
            >
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        class="fenye"
        @size-change="this.handleSizeChange"
        @current-change="this.handleCurrentChange"
        :current-page="this.wodetuanduiPage"
        :page-sizes="[10, 15, 20, 50]"
        :page-size="10"
        layout="total, sizes, prev, pager, next, jumper"
        :total="this.total"
      >
      </el-pagination>
    </div>
    <!-- 下级列表 -->
    <el-dialog
      title="下级列表"
      :visible.sync="xiajiDialogVisible"
      width="70%"
      :before-close="xiajiHandleClose"
    >
      <div class="myTable1">
        <el-table :data="xiajiTable" style="width: 100%">
          <el-table-column
            :show-overflow-tooltip="true"
            prop="pic"
            label="用户头像"
          >
            <template slot-scope="scope">
              <img
                style="width: 40px; height: 40px; cursor: pointer"
                :src="scope.row.facepic"
              />
            </template>
          </el-table-column>
          <el-table-column
            :show-overflow-tooltip="true"
            prop="nickname"
            label="用户昵称"
          >
          </el-table-column>
          <el-table-column
            :show-overflow-tooltip="true"
            prop="addtime"
            label="注册时间"
          >
          </el-table-column>
          <el-table-column
            :show-overflow-tooltip="true"
            prop="invitation_time"
            label="邀请时间"
          >
          </el-table-column>
        </el-table>
        <el-pagination
          class="fenye"
          @size-change="this.xiajihandleSizeChange"
          @current-change="this.xiajihandleCurrentChange"
          :current-page="this.xiajiPage"
          :page-sizes="[10, 15, 20, 50]"
          :page-size="10"
          layout="total, sizes, prev, pager, next, jumper"
          :total="this.xiajiTotal"
        >
        </el-pagination>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { mapState } from "vuex";
export default {
  computed: {
    ...mapState([
      "wodetuanduiPage",
      "wodetuanduiPageSize",
      "xiajiPage",
      "xiajiPageSize",
    ]),
  },
  watch: {
    wodetuanduiPage: function (wodetuanduiPage) {
      this.$store.commit("wodetuanduiPage", wodetuanduiPage);
      this.getData();
    },
    wodetuanduiPageSize: function (wodetuanduiPageSize) {
      this.$store.commit("wodetuanduiPageSize", wodetuanduiPageSize);
      this.getData();
    },
    xiajiPage: function (xiajiPage) {
      this.$store.commit("xiajiPage", xiajiPage);
      this.getDataXiaji();
    },
    xiajiPageSize: function (xiajiPageSize) {
      this.$store.commit("xiajiPageSize", xiajiPageSize);
      this.getDataXiaji();
    },
  },
  data() {
    return {
      total: null,
      tableData: [],
      xiajiTable: [],
      xiajiTotal: null,
      formInline: {
        search: "",
      },
      xiajiDialogVisible: false,
      touserid: null,
    };
  },
  created() {
    this.getData();
  },
  methods: {
    async getData() {
      const res = await this.$api.references_list({
        username: sessionStorage.getItem("user_name"),
        userpass: sessionStorage.getItem("user_pass"),
        page: this.wodetuanduiPage,
        pagesize: this.wodetuanduiPageSize,
        keyword: this.formInline.search,
      });
      console.log(res);
      this.total = res.total;
      this.tableData = res.list;
    },
    async getDataXiaji() {
      const res = await this.$api.referencespush_list({
        username: sessionStorage.getItem("user_name"),
        userpass: sessionStorage.getItem("user_pass"),
        page: this.xiajiPage,
        pagesize: this.xiajiPageSize,
        touserid: this.touserid,
      });
      console.log(res);
      this.xiajiTotal = res.total;
      this.xiajiTable = res.list;
    },
    // 分页
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`);
      this.$store.commit("wodetuanduiPageSize", val);
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
      this.$store.commit("wodetuanduiPage", val);
    },
    xiajihandleSizeChange(val) {
      console.log(`每页 ${val} 条`);
      this.$store.commit("xiajiPageSize", val);
    },
    xiajihandleCurrentChange(val) {
      console.log(`当前页: ${val}`);
      this.$store.commit("xiajiPage", val);
    },
    xiajiHandleClose() {
      this.xiajiDialogVisible = false;
    },
    see(row) {
      this.$store.commit("xiajiPage", 1);
      this.xiajiDialogVisible = true;
      console.log(row);
      this.touserid = row.touserid;
      this.getDataXiaji()
    },
    // 重置
    onReset() {
      this.$store.commit("wodetuanduiPage", 1);
      this.formInline.search = "";
      this.getData();
    },
    // 搜索
    onSubmit() {
        this.getData()
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
.myTable {
  margin-right: 30px;
}
.myForm {
  margin-top: 30px;
}
.fenye {
  margin: 20px 0;
}
</style>
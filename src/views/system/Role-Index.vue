<template>
  <el-row>
    <el-row>
      <el-col :span="20">
        <!--    <el-row style="margin-bottom:6px; padding:0 100px;">
        --  <el-input v-model="queryData.routePath" placeholder="路由" :maxlength="20" size="mini" class="inputwinth150">
               <template slot="prepend">路由</template>
             </el-input>
             <el-input v-model="queryData.code" placeholder="编码" :maxlength="20" size="mini" class="inputwinth150">
               <template slot="prepend">编码</template>
             </el-input>
             <el-input v-model="queryData.name" placeholder="模糊名称" :maxlength="20" size="mini" class="inputwinth150">
               <template slot="prepend">名称</template>
             </el-input>
             <el-radio v-model="queryData.is_Enable" :label="1">启用</el-radio>
             <el-radio v-model="queryData.is_Enable" :label="0">弃用</el-radio>
           </el-row>-->
        <el-row>
          <el-button size="mini" icon="el-icon-plus" @click="openDialog('新增',0)">新增</el-button>
          <el-button size="mini" icon="fa fa-sign-in"> 导入</el-button>
          <el-button size="mini" icon="fa fa-sign-out" @click="exportExcel"> 导出</el-button>
        </el-row>
      </el-col>
      <el-col :span="4">
        <el-button size="mini" @click="cleanData" icon="fa fa-eraser"> 清 空</el-button>
        <el-button size="mini" @click="getPageData" icon="el-icon-search">查 询</el-button>
      </el-col>
    </el-row>
    <func-dialog :dialogTitle="dialogTitle" :dialogShow.sync="dialogShow" :dialogData="dialogData" @RefreshData="getPageData"></func-dialog>
    <el-table :data="tableData" :row-class-name="tableRowClassName" size="mini" v-loading="loadingdata" element-loading-text="正在加载中..." border>
      <!-- <el-table-column type="expand">
           <template slot-scope="props">
               <el-form label-position="left" inline class="demo-table-expand">
                   <el-form-item label="路由">
                       <span>{{ props.row.routePath }}</span>
                   </el-form-item>
                   <el-form-item label="类名">
                       <span>{{ props.row.styleName }}</span>
                   </el-form-item>
                   <el-form-item label="类型">
                       <span>{{ props.row.levelVal }}</span>
                   </el-form-item>
                   <el-form-item label="编码">
                       <span>{{ props.row.code }}</span>
                   </el-form-item>
                   <el-form-item label="名称">
                       <span>{{ props.row.name }}</span>
                   </el-form-item>
                   <el-form-item label="描述">
                       <span>{{ props.row.remarks }}</span>
                   </el-form-item>
                   <el-form-item label="父级">
                       <span>{{ props.row.parentId }}</span>
                   </el-form-item>
                   <el-form-item label="排序">
                       <span>{{ props.row.sortNum }}</span>
                   </el-form-item>
                   <el-form-item label="状态">
                       <span v-html="props.row.is_Enable==true?'启用':'弃用'"></span>
                   </el-form-item>
               </el-form>
           </template>
      </el-table-column>-->
      <el-table-column type="index" label="序号" width="80">
      </el-table-column>
      <el-table-column prop="code" label="编码" width="80">
      </el-table-column>
      <el-table-column prop="ownerId" label="拥有人ID" width="180">
      </el-table-column>
      <el-table-column prop="version" label="版本" width="170">
      </el-table-column>
      <el-table-column prop="code" label="编码" width="150">
      </el-table-column>
      <el-table-column prop="name" label="名称" width="200">
      </el-table-column>
      <el-table-column prop="remarks" label="描述" width="170">
      </el-table-column>
      <el-table-column prop="parentId" label="父级" sortable width="170">
      </el-table-column>
      <el-table-column prop="sortNum" label="排序" sortable width="170">
      </el-table-column>
      <el-table-column label="状态" width="170">
        <template slot-scope="prop">
          <span v-html="prop.row.is_Enable==1?'启用':'弃用'"></span>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="300">
        <template slot-scope="prop">
          <el-button @click="openDialog('编辑',prop.row.id)" type="text">编辑</el-button>
          <el-button @click="delId(prop.row.id)" type="text">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="pIndex" :page-sizes="[2,10, 20, 50, 100]" :page-size="pSize" layout="total, sizes, prev, pager, next, jumper" :total="totalNum">
    </el-pagination>
  </el-row>
</template>
<script>
  import funcdialog from "@/views/system/dialog/Func-Dialog";
  export default {
    components: { "func-dialog": funcdialog },
    data() {
      return {
        dialogData: { id: 0, is_Enable: 1, is_Delete: 0 },
        queryData: {
          routePath: "",
          code: "",
          name: "",
          is_Enable: ""
        },
        moduleName: "roleInfo",
        pIndex: 1,
        pSize: 10,
        totalNum: 0,
        loadingdata: false,
        tableData: [],
        dialogShow: false,
        dialogTitle: "",
        delVisible: false
      };
    },
    created() {
      this.getPageData();
    },
    methods: {
      cleanData() {
        this.queryData = {
          routePath: "",
          code: "",
          name: "",
          is_Enable: ""
        };
      },
      handleSizeChange(val) {
        console.log(`每页 ${val} 条`);
        this.pSize = val;
        this.getPageData();
      },
      handleCurrentChange(val) {
        console.log(`当前页: ${val}`);
        this.pIndex = val;
        this.getPageData();
      },
      tableRowClassName({ row, rowIndex }) {
        if (row.is_Enable == 0) {
          return "warning-row";
        }
        return "";
      },
      getcondition() {
        var obj = this.queryData;
        var condition = "";
        for (var k in obj) {
          if (obj[k] === null || obj[k] === "") continue;
          if (k == "code") condition += "&EQ_" + k + "=" + obj[k];
          else if (k == "name") condition += "&LIKE_" + k + "=" + obj[k];
          else condition += "&EQ_" + k + "=" + obj[k];
        }
        return condition;
      },
      getPageData() {
        var condition = this.getcondition();
        var getpath =
          this.moduleName +
          "/findAll?pIndex=" +
          this.pIndex +
          "&pSize=" +
          this.pSize +
          condition;
        this.loadingdata = true;
        this.$ajax.get(getpath).then(
          res => {
            this.loadingdata = false;
            this.tableData = res.data;
            this.totalNum = res.totalNum;
          },
          error => {
            this.loadingdata = false;
          }
        );
      },
      openDialog(title, id) {
        this.dialogTitle = title;
        this.dialogShow = true;
        if (id > 0) {
          this.$ajax
            .get(this.moduleName + "/findOne?id=" + id)
            .then(res => {
              this.dialogData = res.data;
            });
        } else {
          this.dialogData = { id: 0, is_Enable: 1, is_Delete: 0 };
        }
      },
      delId(id) {
        this.$ajax
          .post(this.moduleName + "/deleteOne?id=" + id)
          .then(res => {
            this.getPageData();
          });
      },
      exportExcel() {}
    }
  };
</script>

<template> 
  <div class="app-container">
    <!--    <el-button size="small" @click="back()">返回</el-button>-->
    <el-card class="filter-container" shadow="never">
      <i class="el-icon-tickets"></i>
      <span>退货申请</span>
    </el-card>
    <div class="table-container">
      <el-form :inline="false" :model="refundAdd" size="medium " label-width="100px">
        <el-form-item label="申请人：">
          <el-col :span="4">
            <el-input :disabled="true" v-model="listQuery.username" class="input-width"></el-input>
          </el-col>
        </el-form-item>
        <el-form-item label="备注：">
          <el-col :span="24">
            <el-input type="textarea"
                      :rows="2"
                      v-model="listQuery.remark"
                      class="input-width">
            </el-input>
          </el-col>
        </el-form-item>
      </el-form>
    </div>

    <el-card class="operate-container" shadow="never">
      <el-button icon="el-icon-circle-plus"
                 type="primary"
                 size="mini"
                 class="btn-add"
                 @click="handleAdd()"
                 style="float: left;
                 margin-bottom:15px">选择试剂
      </el-button>
    </el-card>

    <div class="table-container">
      <el-table ref="refundTable"
                :data="refundAdd"
                :row-class-name="rowClassName"
                style="width: 100%;"
                v-loading="listLoading"
                border>
        <el-table-column type="selection" width="60" align="center"></el-table-column>

        <el-table-column prop="newId" align="center" label="序号" width="60"></el-table-column>
        <el-table-column label="试剂名称" align="center">
          <template slot-scope="scope">{{ scope.row.reagentName }}</template>
        </el-table-column>
        <el-table-column label="供货商" align="center">
          <template slot-scope="scope">{{ scope.row.supplierShortName }}</template>
        </el-table-column>
        <el-table-column label="单位" align="center">
          <template slot-scope="scope">{{ scope.row.unit }}</template>
        </el-table-column>
        <el-table-column label="退货数量" align="center" width="150">
          <template slot-scope="scope">
            <el-input-number
              size="small"
              v-model="scope.row.quantity"
              :min="1" :max="scope.row.reagentNumber">
            </el-input-number>
          </template>
        </el-table-column>

        <el-table-column label="操作" width="220" align="center">
          <template slot-scope="scope">
            <el-button size="mini"
                       icon="el-icon-delete"
                       type="danger"
                       @click="handleDelete(scope.$index)">删除
            </el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-card shadow="never">
        <el-button
          type="primary"
          size="mini"
          @click="submitRefund"
          style="float: left;
                 margin-bottom:15px">提交
        </el-button>
      </el-card>


      <el-dialog title="试剂库存信息" :visible.sync="dialogAddTableVisible" width="84%">
        <el-card shadow="hover" :body-style="{ padding: '10px'}">
          <el-button icon="el-icon-circle-plus"
                     type="primary"
                     size="mini"
                     class="btn-add"
                     @click="toggleSelection()"
                     style="float: left; margin-bottom:11px">添加
          </el-button>
        </el-card>
        <div class="table-container">
          <el-table ref="refundAddTable"
                    :data="list"
                    :row-class-name="rowClassName"
                    height="500"
                    style="width: 100%;"
                    @selection-change="handleSelectionChange"
                    v-loading="listLoading" border>
            <el-table-column type="selection" width="50" fixed align="center"></el-table-column>
            <el-table-column prop="newId" align="center" label="序号" width="60"></el-table-column>

            <el-table-column label="试剂编号" width="110" align="center">
              <template slot-scope="scope">{{ scope.row.reagentId }}</template>
            </el-table-column>
            <el-table-column label="试剂名称" align="center">
              <template slot-scope="scope">{{ scope.row.reagentName }}</template>
            </el-table-column>
            <el-table-column label="单位" align="center">
              <template slot-scope="scope">{{ scope.row.reagentUnit }}</template>
            </el-table-column>
            <el-table-column label="规格型号" align="center">
              <template slot-scope="scope">{{ scope.row.reagentType }}</template>
            </el-table-column>
            <el-table-column label="生产厂家" align="center">
              <template slot-scope="scope">{{ scope.row.factory }}</template>
            </el-table-column>
            <el-table-column label="供货商" align="center">
              <template slot-scope="scope">{{ scope.row.supplierName }}</template>
            </el-table-column>
            <el-table-column label="储存温度" align="center">
              <template slot-scope="scope">{{ scope.row.reagentTemp }}</template>
            </el-table-column>
            <el-table-column label="库存数量" align="center">
              <template slot-scope="scope">{{ scope.row.quantity }}</template>
            </el-table-column>
            <el-table-column label="单价" align="center">
              <template slot-scope="scope">¥ {{ scope.row.reagentPrice }}</template>
            </el-table-column>
          </el-table>
        </div>
        <div class="pagination-container">
          <el-pagination
            background
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            layout="total, sizes,prev, pager, next,jumper"
            :current-page.sync="listQuery.pageNum"
            :page-size="listQuery.pageSize"
            :page-sizes="[50,100,200]"
            :total="total">
          </el-pagination>
        </div>
      </el-dialog>

    </div>
    <el-button style="margin-top: 20px" size="small" @click="back()">返回</el-button>
  </div>
</template>
<script>
import {searchList} from '@/api/stock';
import {formatDate} from '@/utils/date';
import {createRefundItem} from '@/api/refundAdd';
import {getCookie} from "@/utils/support";

const defaultListQuery = {
  pageNum: 1,
  pageSize: 50,
  remark: null,
  username: getCookie("username"),
  refundType: 2,
  stockType: 3,
  refundStatus: 1,
  billType: 3,
  inType: null,
};
export default {
  name: "refundAddList",
  data() {
    return {
      listQuery: Object.assign({}, defaultListQuery),
      loading: false,
      listLoading: false,
      list: null,
      total: null,
      refundAdd: [],
      multipleSelection: [],
      quantity: 1,
      dialogAddTableVisible: false
    }
  },
  created() {
    this.listLoading = false;
  },
  filters: {
    formatDateTime(time) {
      if (time == null || time === '') {
        return 'N/A';
      }
      let date = new Date(time);
      return formatDate(date, 'yyyy-MM-dd')
    }
  },
  methods: {
    rowClassName({row, rowIndex}) {
      //把每一行的索引放进row.id
      row.newId = (this.listQuery.pageSize * (this.listQuery.pageNum - 1) ) + rowIndex + 1;
    },
    back() {
      if (window.history.length <= 1) {
        this.$router.push({path: '/'})
        return false
      } else {
        this.$router.go(-1)
      }
    },
    handleSizeChange(val) {
      this.listQuery.pageNum = 1;
      this.listQuery.pageSize = val;
      this.getList();
    },
    handleCurrentChange(val) {
      this.listQuery.pageNum = val;
      this.getList();
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    handleAdd() {
      this.dialogAddTableVisible = true;
      this.getList();
    },
    submitRefund() {
      const arr = [];
      this.refundAdd.forEach(item => {
        const obj = {
          id: item.id,
          stockNo: item.stockNo,
          reagentId: item.reagentId,
          supplierShortName: item.supplierShortName,
          quantity: item.quantity,
          reagentPrice: item.reagentPrice,
        }
        arr.push(obj);
      })
      let sendData = {
        refundType: this.listQuery.refundType,
        stockType: this.listQuery.stockType,
        refundStatus: this.listQuery.refundStatus,
        billType: this.listQuery.billType,
        inType: this.listQuery.inType,
        refundMessList: arr,
        remark: this.listQuery.remark,
        createBy: this.listQuery.username
      }
      createRefundItem(sendData);
      this.$message({
        message: '提交成功',
        type: 'success',
        duration: 1000
      });
      this.$router.push("/rms/refundTC");
    },

    toggleSelection() {
      if (this.multipleSelection == null || this.multipleSelection.length < 1) {
        this.$message({
          message: '请选择要添加的试剂',
          type: 'warning',
          duration: 1000
        });
      } else {
        let _this = this;
        this.listLoading = true;
        for (let i = 0; i < this.multipleSelection.length; i++) {
          _this.refundAdd.push(this.covertRefund(this.multipleSelection[i]));
        }
        this.dialogAddTableVisible = false;
        this.listLoading = false;
        return _this.refundAdd;
      }
    },

    covertRefund(refund) {
      return {
        id: refund.id,
        stockNo: refund.stockNo,
        reagentId: refund.reagentId,
        reagentName: refund.reagentName,
        unit: refund.reagentUnit,
        supplierShortName: refund.supplierName,
        reagentPrice: refund.reagentPrice,
        reagentNumber: refund.quantity,
        quantity: this.quantity,
        createBy: this.listQuery.username,
        remark: this.listQuery.remark,
      };
    },
    handleDelete(index) {
      this.refundAdd.splice(index, 1);
    },
    getList() {
      this.listLoading = true;
      searchList(this.listQuery).then(response => {
        this.listLoading = false;
        this.list = response.data.list;
        this.total = response.data.total;
      });
    },
  }
}
</script>



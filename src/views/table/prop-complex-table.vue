<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input v-model="listQuery.prop_code" placeholder="道具编码" style="width: 200px;" class="filter-item" @keyup.enter.native="handleFilter" />
      <el-input v-model="listQuery.prop_name" placeholder="道具名称" style="width: 200px;" class="filter-item" @keyup.enter.native="handleFilter" />
      <el-select v-model="listQuery.sort" style="width: 140px" class="filter-item" @change="handleFilter">
        <el-option v-for="item in sortOptions" :key="item.key" :label="item.label" :value="item.key" />
      </el-select>
      <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" @click="handleFilter">
        搜索
      </el-button>
      <el-button class="filter-item" style="margin-left: 10px;" type="primary" icon="el-icon-edit" @click="handleCreate">
        新增
      </el-button>
      <el-button v-waves :loading="downloadLoading" class="filter-item" type="primary" icon="el-icon-download" @click="handleDownload">
        导出
      </el-button>
      <el-checkbox v-model="showStockDetail" class="filter-item" style="margin-left:15px;" @change="tableKey=tableKey+1">
        库存详情
      </el-checkbox>
    </div>

    <el-table
      :key="tableKey"
      v-loading="listLoading"
      :data="list"
      stripe
      fit
      highlight-current-row
      style="width: 100%;"
      @sort-change="sortChange"
    >
      <el-table-column type="expand">
        <template slot-scope="{row}">
          <el-form label-position="left" inline class="demo-table-expand">
            <el-form-item label="图片">
              <el-avatar shape="square" size="large" :src="row.img" />
            </el-form-item>
            <el-form-item label="道具ID">
              <span>{{ row.propId }}</span>
            </el-form-item>
            <el-form-item label="道具编码">
              <span>{{ row.propCode }}</span>
            </el-form-item>
            <el-form-item label="道具名称">
              <span>{{ row.propName }}</span>
            </el-form-item>
            <el-form-item label="单价(元)">
              <span>{{ row.price }}</span>
            </el-form-item>
            <el-form-item label="总库存">
              <span>{{ row.totalStock }}</span>
            </el-form-item>
            <el-form-item label="库存详情">
              <div v-for="(depot, index) in row.depotProp" :key="index">
                <span>{{ depot.depotName }}：{{ depot.stock }} 个(件)</span>
              </div>
            </el-form-item>
            <el-form-item label="备注">
              <span>{{ row.remark }}</span>
            </el-form-item>
            <el-form-item label="添加时间">
              <span>{{ row.createTime }}</span>
            </el-form-item>
            <el-form-item label="更新时间">
              <span>{{ row.updateTime }}</span>
            </el-form-item>
          </el-form>
        </template>
      </el-table-column>
      <el-table-column label="图片" width="150px" align="center">
        <template slot-scope="{row}">
          <el-avatar shape="square" size="medium" :src="row.img" />
        </template>
      </el-table-column>
      <el-table-column label="道具ID" width="170px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.propId }}</span>
        </template>
      </el-table-column>
      <el-table-column label="道具编码" width="150px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.propCode }}</span>
        </template>
      </el-table-column>
      <el-table-column label="道具名称" width="150px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.propName }}</span>
        </template>
      </el-table-column>
      <el-table-column label="单价(元)" width="150px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.price }}</span>
        </template>
      </el-table-column>
      <el-table-column label="总库存" width="150px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.totalStock }}</span>
        </template>
      </el-table-column>
      <el-table-column v-if="showStockDetail" label="库存详情" width="200px" align="center">
        <template slot-scope="{row}">
          <div v-for="(depot, index) in row.depotProp" :key="index">
            <span>{{ depot.depotName }}: {{ depot.stock }} 个(件)</span>
          </div>
        </template>
      </el-table-column>
      <el-table-column label="备注" width="150px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.remark }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" width="230" class-name="small-padding fixed-width">
        <template slot-scope="{row}">
          <el-button type="primary" size="mini" @click="handleUpdate(row)">
            编辑
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination v-show="total>0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit" @pagination="getList" />

    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
      <el-form ref="dataForm" :rules="rules" :model="temp" label-position="left" label-width="100px" style="width: 400px; margin-left:50px;">
        <el-form-item label="图片" prop="img">
          <el-avatar shape="square" size="large" :src="temp.img" />
        </el-form-item>
        <el-form-item label="道具ID" prop="propId">
          <el-input v-model="temp.propId" :disabled="true" />
        </el-form-item>
        <el-form-item label="道具编码" prop="propCode">
          <el-input v-model="temp.propCode" />
        </el-form-item>
        <el-form-item label="道具名称" prop="propName">
          <el-input v-model="temp.propName" />
        </el-form-item>
        <el-form-item label="单价(元)" prop="price">
          <el-input-number v-model="temp.price" :precision="2" size="mini" controls-position="right" :min="0" :max="9999" label="单价(元)" />
        </el-form-item>
        <el-form-item label="库存详情" prop="timestamp">
          <div v-for="(depot, index) in temp.depotProp" :key="index">
            <span>{{ depot.depotName }}：</span>
            <el-input-number v-model="depot.stock" size="mini" controls-position="right" :min="0" :max="9999" label="库存" @change="handleChange" />
          </div>
        </el-form-item>
        <el-form-item label="总库存" prop="totalStock">
          <span>{{ temp.totalStock }}个(件)</span>
        </el-form-item>
        <el-form-item label="备注" prop="remark">
          <el-input v-model="temp.remark" type="textarea" :rows="3" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">
          取消
        </el-button>
        <el-button type="primary" @click="dialogStatus==='create'?createData():updateData()">
          确认
        </el-button>
      </div>
    </el-dialog>

    <el-dialog :visible.sync="dialogPvVisible" title="Reading statistics">
      <el-table :data="pvData" border fit highlight-current-row style="width: 100%">
        <el-table-column prop="key" label="Channel" />
        <el-table-column prop="pv" label="Pv" />
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogPvVisible = false">Confirm</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<style>
  .demo-table-expand {
    font-size: 0;
  }
  .demo-table-expand label {
    width: 90px;
    color: #99a9bf;
  }
  .demo-table-expand .el-form-item {
    margin-right: 0;
    margin-bottom: 0;
    width: 50%;
  }
</style>

<script>
import { fetchList, fetchPv, createProp, updateProp } from '@/api/prop'
import waves from '@/directive/waves' // waves directive
import { parseTime } from '@/utils'
import Pagination from '@/components/Pagination' // secondary package based on el-pagination

const calendarTypeOptions = [
  { key: 'CN', display_name: 'China' },
  { key: 'US', display_name: 'USA' },
  { key: 'JP', display_name: 'Japan' },
  { key: 'EU', display_name: 'Eurozone' }
]

// arr to obj, such as { CN : "China", US : "USA" }
const calendarTypeKeyValue = calendarTypeOptions.reduce((acc, cur) => {
  acc[cur.key] = cur.display_name
  return acc
}, {})

export default {
  name: 'PropComplexTable',
  components: { Pagination },
  directives: { waves },
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'info',
        deleted: 'danger'
      }
      return statusMap[status]
    },
    typeFilter(type) {
      return calendarTypeKeyValue[type]
    }
  },
  data() {
    return {
      tableKey: 0,
      list: null,
      total: 0,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 20,
        prop_code: undefined,
        prop_name: undefined,
        sort: 'prop_id:asc'
      },
      importanceOptions: [1, 2, 3],
      calendarTypeOptions,
      sortOptions: [{ label: '按ID升序', key: 'prop_id:asc' }, { label: '按ID降序', key: 'prop_id:dec' }],
      statusOptions: ['published', 'draft', 'deleted'],
      showStockDetail: false,
      temp: {
        id: undefined,
        importance: 1,
        remark: '',
        timestamp: new Date(),
        title: '',
        type: '',
        status: 'published'
      },
      dialogFormVisible: false,
      dialogStatus: '',
      textMap: {
        update: '编辑',
        create: '新增'
      },
      dialogPvVisible: false,
      pvData: [],
      rules: {
        type: [{ required: true, message: 'type is required', trigger: 'change' }],
        timestamp: [{ type: 'date', required: true, message: 'timestamp is required', trigger: 'change' }],
        title: [{ required: true, message: 'title is required', trigger: 'blur' }]
      },
      downloadLoading: false
    }
  },
  created() {
    this.getList()
  },
  methods: {
    getList() {
      this.listLoading = true
      fetchList(this.listQuery).then(response => {
        this.list = response.data.items
        this.computeTotalStock()
        this.total = response.data.total
        this.listLoading = false
      })
    },
    handleFilter() {
      this.listQuery.page = 1
      this.getList()
    },
    handleModifyStatus(row, status) {
      this.$message({
        message: '操作Success',
        type: 'success'
      })
      row.status = status
    },
    sortChange(data) {
      const { prop, order } = data
      if (prop === 'id') {
        this.sortByID(order)
      }
    },
    sortByID(order) {
      if (order === 'ascending') {
        this.listQuery.sort = '+id'
      } else {
        this.listQuery.sort = '-id'
      }
      this.handleFilter()
    },
    resetTemp() {
      this.temp = {
        id: undefined,
        importance: 1,
        remark: '',
        timestamp: new Date(),
        title: '',
        status: 'published',
        type: ''
      }
    },
    handleCreate() {
      this.resetTemp()
      this.dialogStatus = 'create'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    createData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.temp.id = parseInt(Math.random() * 100) + 1024 // mock a id
          this.temp.author = 'vue-element-admin'
          createProp(this.temp).then(() => {
            this.list.unshift(this.temp)
            this.dialogFormVisible = false
            this.$notify({
              title: 'Success',
              message: 'Created Successfully',
              type: 'success',
              duration: 2000
            })
          })
        }
      })
    },
    handleUpdate(row) {
      this.temp = Object.assign({}, row) // copy obj
      this.temp.timestamp = new Date(this.temp.timestamp)
      this.dialogStatus = 'update'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    computeTotalStock() {
      for (let i = 0; i < this.list.length; i++) {
        this.list[i].totalStock = 0
        for (let j = 0; j < this.list[i].depotProp.length; j++) {
          this.list[i].totalStock += this.list[i].depotProp[j].stock
        }
      }
    },
    updateData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          const tempData = Object.assign({}, this.temp)
          updateProp(tempData).then(() => {
            const index = this.list.findIndex(v => v.id === this.temp.id)
            this.list.splice(index, 1, this.temp)
            this.dialogFormVisible = false
            this.$notify({
              title: 'Success',
              message: 'Update Successfully',
              type: 'success',
              duration: 2000
            })
          })
        }
      })
    },
    handleDelete(row, index) {
      this.$notify({
        title: 'Success',
        message: 'Delete Successfully',
        type: 'success',
        duration: 2000
      })
      this.list.splice(index, 1)
    },
    handleFetchPv(pv) {
      fetchPv(pv).then(response => {
        this.pvData = response.data.pvData
        this.dialogPvVisible = true
      })
    },
    handleDownload() {
      this.downloadLoading = true
      import('@/vendor/Export2Excel').then(excel => {
        const tHeader = ['道具ID', '道具编码', '道具名称', '单价(元)', '总库存', '库存详情', '备注', '添加时间', '更新时间']
        const filterVal = ['propId', 'propCode', 'propName', 'price', 'totalStock', 'stockDetailStr', 'remark', 'createTime', 'updateTime']
        const data = this.formatJson(filterVal)
        excel.export_json_to_excel({
          header: tHeader,
          data,
          filename: '道具信息'
        })
        this.downloadLoading = false
      })
    },
    formatJson(filterVal) {
      this.list = this.list.map(v => {
        const data = v.depotProp.map(w => {
          return w.depotName + ': ' + w.stock
        })
        return Object.assign({}, v, { 'stockDetailStr': data.toString() })
      })
      return this.list.map(v => filterVal.map(j => {
        if (j === 'createTime' || j === 'updateTime') {
          return parseTime(v[j])
        } else {
          return v[j]
        }
      }))
    },
    getSortClass: function(key) {
      const sort = this.listQuery.sort
      return sort === `+${key}` ? 'ascending' : 'descending'
    },
    handleChange(value) {
      this.temp.totalStock = 0
      for (let j = 0; j < this.temp.depotProp.length; j++) {
        this.temp.totalStock += this.temp.depotProp[j].stock
      }
    }
  }
}
</script>

<template>
  <div>
    <br>
    <br>
    <div>
      <el-form ref="form" :model="form" label-width="100px">
        <div style="display: flex; flex-wrap: wrap; gap: 20px">
          <el-form-item label="管理员UID">
            <el-input v-model="form.adminUid" placeholder="请输入管理员UID" style="width: 200px"/>
          </el-form-item>

          <el-form-item label="我的组ID">
            <el-input v-model="form.myGid" placeholder="请输入我的组ID" style="width: 200px"/>
          </el-form-item>

          <el-form-item label="转发组ID">
            <el-input v-model="form.forwardGid" placeholder="请输入转发组ID" style="width: 200px"/>
          </el-form-item>

          <el-form-item>
            <el-button @click="queryData">查询</el-button>
            <el-button type="primary" @click="addRow">新增</el-button>
          </el-form-item>
        </div>
      </el-form>
    </div>
    <br><br>

    <el-table v-if="showTable" :data="tableData" border style="width: 100%">
      <el-table-column prop="id" label="ID" width="80" />
      <el-table-column prop="AdminUid" label="管理员UID" width="150">
        <template slot-scope="scope">
          <el-input v-model="scope.row.adminUid" />
        </template>
      </el-table-column>
      <el-table-column prop="MyGid" label="我的组ID" width="250">
        <template slot-scope="scope">
          <el-input v-model="scope.row.myGid" />
        </template>
      </el-table-column>
      <el-table-column prop="forwardGid" label="转发组ID" width="250">
        <template slot-scope="scope">
          <el-input v-model="scope.row.forwardGid" />
        </template>
      </el-table-column>
      <el-table-column prop="quote" label="报价" width="100">
        <template slot-scope="scope">
          <el-select v-model="scope.row.quote">
            <el-option v-for="option in statusOptions" :key="option.value" :label="option.label" :value="option.value" />
          </el-select>
        </template>
      </el-table-column>
      <el-table-column prop="forward" label="转发" width="100">
        <template slot-scope="scope">
          <el-select v-model="scope.row.forward">
            <el-option v-for="option in statusOptions" :key="option.value" :label="option.label" :value="option.value" />
          </el-select>
        </template>
      </el-table-column>
      <el-table-column prop="buy" label="购买" width="100">
        <template slot-scope="scope">
          <el-select v-model="scope.row.buy">
            <el-option v-for="option in statusOptions" :key="option.value" :label="option.label" :value="option.value" />
          </el-select>
        </template>
      </el-table-column>
      <el-table-column prop="idistinct" label="去重" width="100">
        <template slot-scope="scope">
          <el-select v-model="scope.row.idistinct">
            <el-option v-for="option in statusOptions" :key="option.value" :label="option.label" :value="option.value" />
          </el-select>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="200">
        <template slot-scope="scope">
          <el-button size="mini" type="primary" @click="saveRow(scope.row)">保存</el-button>
          <el-button size="mini" type="danger" @click="deleteRow(scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      form: {
        adminUid: '',
        myGid: '',
        forwardGid: ''
      },
      tableData: [],
      showTable: true,
      statusOptions: [
        { label: '开启', value: 1 },
        { label: '关闭', value: 0 }
      ]
    }
  },
  methods: {
    async queryData() {
      try {
        const response = await axios.get('/money/getConfig');
        const allData = response.data.data;

        if (!Array.isArray(allData)) {
          throw new Error('返回的数据不是数组');
        }

        this.tableData = allData.filter(item => {
          const adminUidMatch = !this.form.adminUid ||
            (item.adminUid && item.adminUid.toLowerCase().includes(this.form.adminUid.toLowerCase()));
          const myGidMatch = !this.form.myGid ||
            (item.myGid && item.myGid.toLowerCase().includes(this.form.myGid.toLowerCase()));
          const forwardGidMatch = !this.form.forwardGid ||
            (item.forwardGid && item.forwardGid.toLowerCase().includes(this.form.forwardGid.toLowerCase()));
          return adminUidMatch && myGidMatch && forwardGidMatch;
        });

        this.showTable = this.tableData.length > 0;
        this.$message.success('查询成功');
      } catch (error) {
        console.error('查询失败:', error);
        this.$message.error('查询失败，请稍后重试');
        this.tableData = [];
        this.showTable = false;
      }
    },
    async saveRow(row) {
      try {
        const response = await axios.post('/money/saveConfig', row);
        if (response.data.code === '200') {
          this.$message.success('保存成功');
          this.queryData(); // 刷新表格数据
        }
      } catch (error) {
        console.error('保存失败:', error);
        this.$message.error('保存失败，请稍后重试');
      }
    },
    async deleteRow(row) {
      this.$confirm('确定要删除该记录吗?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async() => {
        try {
          const response = await axios.delete(`/money/deleteConfig/${row.id}`);
          if (response.data.code === '200') {
            this.$message.success('删除成功');
            this.queryData(); // 刷新表格数据
          }
        } catch (error) {
          console.error('删除失败:', error);
          this.$message.error('删除失败，请稍后重试');
        }
      }).catch(() => {
        this.$message.info('已取消删除');
      });
    },
    addRow() {
      this.tableData.unshift({
        adminUid: '',
        myGid: '',
        forwardGid: '',
        quote: 0,
        forward: 0,
        buy: 0,
        idistinct: 0
      });
      this.showTable = true;
    }
  }
}
</script>

<style scoped>
.el-table {
  margin-top: 20px;
}

.el-select {
  width: 100%;
}
</style>

<template>
  <div>
    <br>
    <br>
    <div>
      <el-form ref="form" :model="form" label-width="80px" prop="role">
        <div style="display: flex">
          <el-form-item label="微信UID">
            <el-input id="field1" v-model="form.weChatUid" placeholder="请输入微信UID" type="text" style="width: 200px"/>
          </el-form-item>

          <el-form-item label="微信昵称">
            <el-input id="field2" v-model="form.weChatNickname" type="text" placeholder="请输入微信昵称" style="width: 200px"/>
          </el-form-item>

          <el-form-item label="状态">
            <el-select id="field3" v-model="form.status" filterable placeholder="请选择状态" @change="valSelectChange">
              <el-option v-for="option in options" :key="option.value" :label="option.label" :value="option.value">
                {{ option.label }}
              </el-option>
            </el-select>
          </el-form-item>

          <el-form-item>
            <el-button @click="queryData">查询</el-button>
            <el-button type="primary" @click="addRow">新增</el-button>
          </el-form-item>
        </div>
      </el-form>
    </div>
    <br><br>

    <el-form>
      <el-table v-if="showTable1" id="t1" :data="tableData1" class="table2">
        <el-table-column prop="weChatUid" label="微信UID">
          <template slot-scope="scope">
            <el-input v-model="scope.row.weChatUid"></el-input>
          </template>
        </el-table-column>
        <el-table-column prop="weChatNickname" label="微信昵称">
          <template slot-scope="scope">
            <el-input v-model="scope.row.weChatNickname"></el-input>
          </template>
        </el-table-column>
        <el-table-column prop="status" label="状态">
          <template slot-scope="scope">
            <el-select v-model="scope.row.status">
              <el-option v-for="option in options" :key="option.value" :label="option.label" :value="option.value">
                {{ option.label }}
              </el-option>
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
      <br><br>
    </el-form>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      form: {
        weChatUid: '',
        weChatNickname: '',
        status: null
      },
      tableData1: [],
      showTable1: true,
      options: [

        { label: '全部', value: null },
        { label: '开启', value: 1 },
        { label: '关闭', value: 0 }
      ]
    }
  },
  methods: {
    async queryData() {
      try {
        const response = await axios.get('/wx/getGroup');
        const allData = response.data.data;

        console.log('提取的数组数据:', allData);

        if (!Array.isArray(allData)) {
          throw new Error('返回的 data 字段不是数组');
        }

        this.tableData1 = allData.filter(item => {
          const uidMatch = !this.form.weChatUid ||
            (item.weChatUid && item.weChatUid.toLowerCase().includes(this.form.weChatUid.toLowerCase()));
          const nicknameMatch = !this.form.weChatNickname ||
            (item.weChatNickname && item.weChatNickname.toLowerCase().includes(this.form.weChatNickname.toLowerCase()));
          // const statusMatch = !this.form.status ||
          //   (item.status && item.status === this.form.status);
          const statusMatch = this.form.status === null ||
            this.form.status === undefined ||
            item.status === this.form.status;
          return uidMatch && nicknameMatch && statusMatch;
        });

        this.showTable1 = this.tableData1.length > 0;
        this.$message({
          message: '查询成功',
          type: 'success'
        });
      } catch (error) {
        console.error('查询失败:', error);
        this.$message.error('查询失败，请稍后重试');
        this.tableData1 = [];
        this.showTable1 = false;
      }
    },
    async saveRow(row) {
      try {
        const response = await axios.post('/wx/saveGroup', row);
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
      }).then(async () => {
        try {
          const response = await axios.delete(`/wx/deleteGroup/${row.id}`);
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
      this.tableData1.unshift({
        weChatUid: '',
        weChatNickname: '',
        status: 1
      });
      this.showTable1 = true;
    },
    valSelectChange(val) {
      this.form.status = val;
    }
  }
}
</script>

<style scoped>
table {
  width: 100%;
}
th, td {
  padding: 10px;
  border: 1px solid #ccc;
}
.table2 {
  position: absolute;
  top: 90px;
}
</style>

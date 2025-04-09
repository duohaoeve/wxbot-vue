<template>
  <div>
    <br>
    <br>
    <div>
      <el-form ref="form" :model="form" label-width="100px">
        <div style="display: flex; flex-wrap: wrap; gap: 20px">
          <el-form-item label="CA字段">
            <el-input v-model="form.ca" placeholder="请输入CA字段" style="width: 200px"/>
          </el-form-item>

          <el-form-item label="符号">
            <el-input v-model="form.symbol" placeholder="请输入符号" style="width: 200px"/>
          </el-form-item>

          <el-form-item label="创建时间">
            <el-input v-model="form.createdAt" placeholder="请输入创建时间" style="width: 200px"/>
          </el-form-item>

          <el-form-item label="组名称">
            <el-input v-model="form.groupName" placeholder="请输入组名称" style="width: 200px"/>
          </el-form-item>

          <el-form-item label="发送者">
            <el-input v-model="form.sender" placeholder="请输入发送者" style="width: 200px"/>
          </el-form-item>

          <el-form-item>
            <el-button @click="queryData">查询</el-button>
          </el-form-item>
        </div>
      </el-form>
    </div>
    <br>
    <div>总记录数: {{ totalCount }}</div>
    <br>

    <el-table v-if="showTable" :data="tableData" border style="width: 100%">
      <el-table-column prop="ca" label="CA字段" width="200"/>
      <el-table-column prop="symbol" label="符号" width="120" />
      <el-table-column prop="chain" label="链" width="80"/>
      <el-table-column prop="marketCap" label="市值" width="80" />
      <el-table-column prop="createdAt" label="创建时间" />
      <el-table-column prop="roomId" label="群组ID" />
      <el-table-column prop="groupName" label="组名称" />
      <el-table-column prop="sender" label="发送者" />
      <el-table-column prop="senderName" label="发送者名称" />
      <el-table-column prop="highMc" label="最高市值" />
      <el-table-column prop="multiple" label="倍数" width="80"/>
    </el-table>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      form: {
        ca: '',
        symbol: '',
        createdAt: '',
        groupName: '',
        sender: ''
      },
      tableData: [],
      showTable: true,
      totalCount: 0
    }
  },
  methods: {
    async queryData() {
      try {
        // 获取列表数据
        const listResponse = await axios.get('/money/getAllCa');
        const allData = listResponse.data.data;

        if (!Array.isArray(allData)) {
          throw new Error('返回的数据不是数组');
        }

        // 过滤数据
        this.tableData = allData.filter(item => {
          const caMatch = !this.form.ca ||
            (item.ca && item.ca.toLowerCase().includes(this.form.ca.toLowerCase()));
          const symbolMatch = !this.form.symbol ||
            (item.symbol && item.symbol.toLowerCase().includes(this.form.symbol.toLowerCase()));
          const createdAtMatch = !this.form.createdAt ||
            (item.createdAt && item.createdAt.toLowerCase().includes(this.form.createdAt.toLowerCase()));
          const groupNameMatch = !this.form.groupName ||
            (item.groupName && item.groupName.toLowerCase().includes(this.form.groupName.toLowerCase()));
          const senderMatch = !this.form.sender ||
            (item.sender && item.sender.toLowerCase().includes(this.form.sender.toLowerCase()));
          return caMatch && symbolMatch && createdAtMatch && groupNameMatch && senderMatch;
        });

        this.showTable = this.tableData.length > 0;

        // 获取总数
        const countResponse = await axios.get('/money/getAllCaCmt');
        this.totalCount = countResponse.data.data;

        this.$message.success('查询成功');
      } catch (error) {
        console.error('查询失败:', error);
        this.$message.error('查询失败，请稍后重试');
        this.tableData = [];
        this.showTable = false;
        this.totalCount = 0;
      }
    }
  }
}
</script>

<style scoped>
.el-table {
  margin-top: 20px;
}
</style>


<template>
  <div>

    <br>
    <br>
    <div>

      <!-- 多个查询字段 -->
      <el-form ref="form" :model="form" label-width="80px" prop="role">
        <div style="display: flex">
          <el-form-item label="类型">
            <!-- 下拉框 -->
            <el-select v-model="selected" filterable placeholder="请选择表格" @change="valSelectChange">
              <el-option v-for="option in options" :key="option.value" :label="option.label" :value="option.value">{{
                option.label }}
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="姓名">
            <el-input id="field1" v-model="name" label="姓名" placeholder="请输入姓名" type="text" style="width: 200px"/>

          </el-form-item>

          <el-form-item label="性别">
            <el-input id="field2" v-model="age" label="性别" type="text" placeholder="请输入性别" style="width: 200px"/>
          </el-form-item>

          <el-form-item>
            <el-button @click="queryData">查询</el-button>
          </el-form-item>
        </div>
        <!--      <label for="field1">姓名：</label>-->
        <!--      <el-input id="field1" v-model="name" label="字段1" placeholder="请输入姓名" type="text" style="width: 200px"/>-->

      </el-form>

      <!-- 查询按钮 -->
<!--      <button @click="queryData"></button>-->
    </div>
    <br><br>

    <el-form >
    <el-table v-if="showTable1" id="t1" v-model="tableData1" :data="tableData1" class="table2" >
      <el-table-column prop="name" label="姓名"></el-table-column>
      <el-table-column prop="age" label="年龄"></el-table-column>
      <el-table-column prop="gender" label="性别"></el-table-column>
      <el-table-column prop="start_time" label="时间戳" :formatter="formatDate"></el-table-column>
    </el-table>
    <br><br>
    <el-table v-if="showTable2" id="t2" v-model="tableData2" class="table2">
      <el-table-column label="列21"></el-table-column>
      <el-table-column label="列22"></el-table-column>
    </el-table>
    <br><br>
    <el-table v-if="showTable3" id="t3" v-model="tableData3" class="table2">
      <el-table-column label="列321"></el-table-column>
      <el-table-column label="列322"></el-table-column>
    </el-table>
    </el-form>
  </div>
</template>

<script>
export default {
  data() {
    return {
      name: '',
      age: '',
      tableData1: [], // 表格数据
      tableData2: [],
      tableData3: [],
      showTable1: true,
      showTable2: false,
      showTable3: false,
      selected: '',
      options: [
        { label: '表格1', value: 'a' },
        { label: '表格2', value: 'b' },
        { label: '表格3', value: 'c' }
      ]
    }
  },
  methods: {


    formatDate(row) {
      const timestamp = Number(row.start_time);
      if (typeof timestamp === 'number' && !isNaN(timestamp)) {
        return new Date(timestamp*1000).toLocaleString();
      } else {
        console.log(row);
        return '';


      }
    },


    queryData() {
      // 模拟更新后端
      // axios.put('/api/update/' + row.id, row).then(response => {
      //   this.$message({
      //     message: '数据更新成功',
      //     type: 'success'
      //   })
      // }).catch(error => {
      //   console.log(error)
      //   this.$message.error('数据更新失败')
      // })
      // 模拟更新后端数据
      // 更新表格数据
      this.tableData1 = [
        { name: '张三', age: 21, gender: '男' ,start_time:'1694240712'},
        { name: '李四', age: 26, gender: '女' ,start_time:'1694240712'},
        { name: '王五', age: 23, gender: '男' ,start_time:'1694240712'}
      ];
    },
    valSelectChange(val) {
      if (val === 'a') {
        this.showTable1 = true
        this.showTable2 = false
        this.showTable3 = false
      }
      if (val === 'b') {
        this.showTable1 = false
        this.showTable2 = true
        this.showTable3 = false
      }
      if (val === 'c') {
        this.showTable1 = false
        this.showTable2 = false
        this.showTable3 = true
      }
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


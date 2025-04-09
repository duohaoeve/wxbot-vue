<template>
  <div style="padding: 20px;">
    <!-- 顶部查询区域 -->
    <div style="display: flex; justify-content: space-between;">
      <!-- 左侧群组查询 -->
      <div style="width: 45%;">
        <el-form :model="groupForm" label-width="80px">
          <el-form-item label="群组名称">
            <el-input v-model="groupForm.weChatNickname" placeholder="请输入群组名称" style="width: 200px"/>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="queryGroups">查询群组</el-button>
          </el-form-item>
        </el-form>
        <!-- 群组列表 -->
        <el-table
          :data="filteredGroupData"
          style="width: 100%;"
          @row-click="handleGroupSelect"
          highlight-current-row
        >
          <el-table-column prop="weChatUid" label="群组ID"></el-table-column>
          <el-table-column prop="weChatNickname" label="群组名称"></el-table-column>
        </el-table>
      </div>

      <!-- 右侧成员查询 -->
      <div style="width: 45%;">
        <el-form :model="memberForm" label-width="80px">
          <div style="display: flex; flex-wrap: wrap;">
            <el-form-item label="微信UID">
              <el-input v-model="memberForm.weChatUid" placeholder="请输入微信UID" style="width: 200px"/>
            </el-form-item>
            <el-form-item label="微信昵称">
              <el-input v-model="memberForm.groupNickName" placeholder="请输入微信昵称" style="width: 200px"/>
            </el-form-item>
          </div>
          <div style="display: flex; flex-wrap: wrap; margin-top: 10px;">
            <el-form-item label="状态">
              <el-input v-model="memberForm.state" placeholder="请输入状态" style="width: 200px"/>
            </el-form-item>
            <el-form-item>
              <el-button type="primary" @click="queryMembers">查询成员</el-button>
            </el-form-item>
          </div>
        </el-form>
        <!-- 成员列表 -->
        <el-table
          :data="filteredMemberData"
          style="width: 100%;"
          @row-click="handleMemberSelect"
          highlight-current-row
        >
          <el-table-column prop="weChatUid" label="微信UID"></el-table-column>
          <el-table-column prop="groupNickName" label="微信昵称"></el-table-column>
          <el-table-column prop="state" label="状态"></el-table-column>
        </el-table>
      </div>
    </div>

    <!-- 下方选中数据表格 -->
    <div style="margin-top: 20px;">
      <el-table :data="selectedData" style="width: 100%;">
        <el-table-column prop="weChatUid" label="群组ID"></el-table-column>
        <el-table-column prop="weChatNickname" label="群组名称"></el-table-column>
        <el-table-column prop="memberWeChatUid" label="微信UID"></el-table-column>
        <el-table-column prop="memberGroupNickName" label="微信昵称"></el-table-column>
        <el-table-column prop="state" label="状态">
          <template slot-scope="scope">
            <el-select v-model="scope.row.state">
              <el-option v-for="option in stateOptions" :key="option.value" :label="option.label" :value="option.value">
                {{ option.label }}
              </el-option>
            </el-select>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="200">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" @click="saveRow(scope.row)">保存</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      // 群组查询表单
      groupForm: {
        weChatNickname: '',
      },
      // 成员查询表单
      memberForm: {
        weChatUid: '',
        groupNickName: '',
        state: null,
      },
      // 群组列表数据（原始数据）
      groupData: [],
      // 过滤后的群组数据
      filteredGroupData: [],
      // 成员列表数据（原始数据）
      memberData: [],
      // 过滤后的成员数据
      filteredMemberData: [],
      // 选中的群组和成员数据
      selectedData: [],
      // 状态选项
      stateOptions: [
        { label: '全部', value: null },
        { label: '开启', value: 1 },
        { label: '关闭', value: 0 },
      ],
      // 当前选中的群组
      selectedGroup: null,
    };
  },
  methods: {
    // 查询群组
    async queryGroups() {
      try {
        const response = await axios.get('/money/getWxGroup');
        if (response.data.code === '200') {
          this.groupData = response.data.data || [];
          this.filterGroups();
          this.$message.success('查询群组成功');
        } else {
          throw new Error(response.data.msg || '查询失败');
        }
      } catch (error) {
        console.error('查询群组失败:', error);
        this.$message.error('查询群组失败，请稍后重试');
        this.groupData = [];
        this.filteredGroupData = [];
      }
    },
    // 过滤群组（模糊查询）
    filterGroups() {
      const keyword = this.groupForm.weChatNickname.trim().toLowerCase();
      if (!keyword) {
        this.filteredGroupData = [...this.groupData];
      } else {
        this.filteredGroupData = this.groupData.filter(item =>
          item.weChatNickname.toLowerCase().includes(keyword)
        );
      }
    },
    // 查询成员
    async queryMembers() {
      if (!this.selectedGroup) {
        this.$message.warning('请先选择一个群组');
        return;
      }
      try {
        const response = await axios.post('/wechat/cgi/wcf/groupMember/list', {
          groupNo: this.selectedGroup.weChatUid,
        });
        if (response.data.code === '200') {
          this.memberData = response.data.data || [];
          this.filterMembers();
          this.$message.success('查询成员成功');
        } else {
          throw new Error(response.data.msg || '查询失败');
        }
      } catch (error) {
        console.error('查询成员失败:', error);
        this.$message.error('查询成员失败，请稍后重试');
        this.memberData = [];
        this.filteredMemberData = [];
      }
    },
    // 过滤成员（模糊查询）
    filterMembers() {
      let filtered = [...this.memberData];
      const weChatUid = this.memberForm.weChatUid.trim().toLowerCase();
      const groupNickName = this.memberForm.groupNickName.trim().toLowerCase();
      const state = this.memberForm.state;

      if (weChatUid) {
        filtered = filtered.filter(item =>
          item.weChatUid.toLowerCase().includes(weChatUid)
        );
      }
      if (groupNickName) {
        filtered = filtered.filter(item =>
          item.groupNickName.toLowerCase().includes(groupNickName)
        );
      }
      if (state !== null) {
        filtered = filtered.filter(item => item.state === state);
      }
      this.filteredMemberData = filtered;
    },
    // 选中群组
    handleGroupSelect(row) {
      this.selectedGroup = row;
      this.memberData = [];
      this.filteredMemberData = [];
      this.selectedData = [];
      this.$message.info(`已选择群组：${row.weChatNickname}`);
    },
    // 选中成员
    handleMemberSelect(row) {
      const selectedItem = {
        weChatUid: this.selectedGroup.weChatUid, // 群组ID
        weChatNickname: this.selectedGroup.weChatNickname, // 群组名称
        memberWeChatUid: row.weChatUid, // 成员微信UID
        memberGroupNickName: row.groupNickName, // 成员群昵称
        state: 1, // 状态，默认为 1
      };
      if (!this.selectedData.some(item => item.memberWeChatUid === row.weChatUid)) {
        this.selectedData.push(selectedItem);
      }
      this.$message.info(`已选择成员：${row.groupNickName}`);
    },
    // 保存行数据
    async saveRow(row) {
      try {
        const response = await axios.post('/wx/addRoomAndMem', {
          id: null, // id 留空
          weChatUid: row.memberWeChatUid, // 成员微信UID
          weChatNickname: row.memberGroupNickName, // 成员群昵称
          groupUid: row.weChatUid, // 群组ID
          groupName: row.weChatNickname, // 群组名称
          status: 1, // 状态
        });
        if (response.data.code === '200') {
          this.$message.success('保存成功');
        } else {
          throw new Error(response.data.msg || '保存失败');
        }
      } catch (error) {
        console.error('保存失败:', error);
        this.$message.error('保存失败，请稍后重试');
      }
    },
  },
  watch: {
    // 监听群组名称输入框变化，实时过滤
    'groupForm.weChatNickname'() {
      this.filterGroups();
    },
    // 监听成员查询条件变化，实时过滤
    'memberForm.weChatUid'() {
      this.filterMembers();
    },
    'memberForm.groupNickName'() {
      this.filterMembers();
    },
    'memberForm.state'() {
      this.filterMembers();
    },
  },
};
</script>

<style scoped>
.el-table {
  margin-top: 10px;
}
</style>

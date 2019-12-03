<template>
  <div class="view">
    <Panel :title="$t('m.Group_Group') ">
      <div slot="header">
        <el-row :gutter="20">
          <el-col :span="8">
            <el-button v-show="selectedGroups.length"
                       type="warning" icon="el-icon-fa-trash"
                       @click="deleteGroups(selectedGroupIDs)">Delete
            </el-button>
          </el-col>
          <el-col :span="selectedGroups.length ? 16: 24">
            <el-input v-model="keyword" prefix-icon="el-icon-search" placeholder="Keywords"></el-input>
          </el-col>
        </el-row>
      </div>
      <el-table
        v-loading="loadingTable"
        element-loading-text="loading"
        @selection-change="handleSelectionChange"
        ref="table"
        :data="groupList"
        style="width: 100%">
        <el-table-column type="selection" width="55"></el-table-column>

        <el-table-column prop="id" label="ID"></el-table-column>

        <el-table-column prop="groupname" label="Group Name"></el-table-column>

        <el-table-column fixed="right" label="Option" width="200">
          <template slot-scope="{row}">
            <icon-btn name="Edit" icon="edit" @click.native="openGroupDialog(row.id)"></icon-btn>
            <icon-btn name="User" icon="list-ol" @click.native="goGroupUserList(row.id)"></icon-btn>
            <icon-btn name="Delete" icon="trash" @click.native="deleteGroups([row.id])"></icon-btn>
          </template>
        </el-table-column>
      </el-table>
      <div class="panel-options">
        <el-pagination
          class="page"
          layout="prev, pager, next"
          @current-change="currentChange"
          :page-size="pageSize"
          :total="total">
        </el-pagination>
      </div>
    </Panel>


    <Panel :title="$t('m.Create_Group')">
      <el-form :model="formCreateGroup" ref="createGroupName">
        <el-row type="flex" justify="space-between">
          <el-col :span="8">
            <el-form-item label="Group Name" prop="group name">
              <el-input v-model="formCreateGroup.groupname" placeholder="Group Name"></el-input>
            </el-form-item>
          </el-col>
        </el-row>

        <el-form-item>
          <el-button type="primary" @click="createGroup" icon="el-icon-fa-users" :loading="loadingGenerate">Generate
          </el-button>
        </el-form-item>
      </el-form>
    </Panel>
    <!--对话框-->
    <el-dialog :title="$t('m.Group_Info')" :visible.sync="showGroupDialog" :close-on-click-modal="false">
      <el-form :model="group" label-width="120px" label-position="left">
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item :label="$t('m.Group_Groupname')" required>
              <el-input v-model="group.groupname"></el-input>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <cancel @click.native="showGroupDialog = false">Cancel</cancel>
        <save @click.native="saveGroup()"></save>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  import papa from 'papaparse'
  import api from '../../api.js'
  import utils from '@/utils/utils'

  export default {
    name: 'Group',
    data () {
      return {
        // 一页显示的用户数
        pageSize: 10,
        // 用户总数
        total: 0,
        // 用户列表
        // 搜索关键字
        keyword: '',
        groupList: [],
        // 是否显示组对话框
        showGroupDialog: false,
        // 当前用户model
        group: {},
        loadingTable: false,
        loadingGenerate: false,
        // 当前页码
        currentPage: 0,
        selectedGroups: [],
        formCreateGroup: {
          groupname: ''
        }
      }
    },
    mounted () {
      this.getGroupList(1)
    },
    methods: {
      // 切换页码回调
      currentChange (page) {
        this.currentPage = page
        this.getGroupList(page)
      },
      // 提交修改用户的信息
      saveGroup () {
        console.log(this.group)
        api.editGroup(this.group).then(res => {
          // 更新列表
          this.getGroupList(this.currentPage)
        }).then(() => {
          this.showGroupDialog = false
        }).catch(() => {
        })
      },
      // 打开用户对话框
      openGroupDialog (id) {
        this.showGroupDialog = true
        // todo #################################
        api.getGroup(id).then(res => {
          this.group = res.data.data
        })
      },
      // 获取用户列表
      getGroupList (page) {
        this.loadingTable = true
        // todo #############################################################
        api.getGroupList((page - 1) * this.pageSize, this.pageSize, this.keyword).then(res => {
          this.loadingTable = false
          this.total = res.data.data.total
          // console.log(res.data.data.results)
          this.groupList = res.data.data.results
          // this.groupList = [{id: 1, username: 'liwan'}, {id: 2, username: 'hah'}]
        }, res => {
          this.loadingTable = false
        })
      },
      // 跳转到组列表
      goGroupUserList (groupId) {
        this.$router.push({name: 'group-user-list', params: {groupId}})
      },
      deleteGroups (ids) {
        this.$confirm('Sure to delete the group? The associated resources created by this group will be deleted as well.', 'confirm', {
          type: 'warning'
        }).then(() => {
          api.deleteGroups(ids.join(',')).then(res => {
            this.getGroupList(this.currentPage)
          }).catch(() => {
            this.getGroupList(this.currentPage)
          })
        }, () => {
        })
      },
      handleSelectionChange (val) {
        this.selectedGroups = val
      },
      createGroup () {
        this.loadingGenerate = true
        let data = Object.assign({}, this.formCreateGroup)
        api.createGroup(data).then(res => {
          this.loadingGenerate = false
          this.getGroupList(1)
        }).catch(() => {
          this.loadingGenerate = false
        })
      }
    },
    computed: {
      selectedGroupIDs () {
        let ids = []
        for (let group of this.selectedGroups) {
          ids.push(group.id)
        }
        return ids
      }
    },
    watch: {
      'keyword' () {
        this.currentChange(1)
      }
    }
  }
</script>

<style scoped lang="less">
  .import-user-icon {
    color: #555555;
    margin-left: 4px;
  }

  .userPreview {
    padding-left: 10px;
  }

  .notification {
    p {
      margin: 0;
      text-align: left;
    }
  }
</style>

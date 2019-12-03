<template>
  <div class="view">
    <Panel :title="$t('m.Group_User') ">
      <div slot="header">
        <el-row :gutter="20">
          <el-col :span="8">
            <el-button v-show="selectedDeleteUsers.length"
                       type="warning" icon="el-icon-fa-trash"
                       @click="deleteGroupUsers(selectedDeleteUserIDs)">Delete
            </el-button>
          </el-col>
          <el-col :span="selectedDeleteUsers.length ? 16: 24">
            <el-input v-model="keyword" prefix-icon="el-icon-search" placeholder="Keywords"></el-input>
          </el-col>
        </el-row>
      </div>
      <el-table
        v-loading="loadingGroupUserTable"
        element-loading-text="loading"
        @selection-change="handleSelectionChange"
        ref="table"
        :data="groupUserList"
        style="width: 100%">
        <el-table-column type="selection" width="55"></el-table-column>

        <el-table-column prop="user_id" label="ID"></el-table-column>

        <el-table-column prop="username" label="Username"></el-table-column>

        <el-table-column
          prop="user_type"
          label="Admin">
          <template slot-scope="scope">
            <el-switch v-model="scope.row.user_type"
                       active-text=""
                       inactive-text=""
                       @change="updateGroupUser(scope.row.user_id, scope.row.user_type)">
            </el-switch>
          </template>
        </el-table-column>

        <el-table-column fixed="right" label="Option" width="100">
          <template slot-scope="scope">
            <icon-btn name="Delete" icon="trash" @click.native="deleteGroupUsers([scope.row.user_id])"></icon-btn>
          </template>
        </el-table-column>
      </el-table>
      <div class="panel-options">
        <el-button v-if="groupId" type="primary"
                   size="big" icon="el-icon-plus"
                   @click="addUserDialogVisible = true">Add User
        </el-button>
        <el-pagination
          class="page"
          layout="prev, pager, next"
          @current-change="currentChange"
          :page-size="pageSize"
          :total="total">
        </el-pagination>
      </div>
    </Panel>
    <el-dialog title="Add Group User"
               v-if="groupId"
               width="60%"
               :visible.sync="addUserDialogVisible"
               @close-on-click-modal="false">
      <add-user-to-group-component :groupID="groupId" @on-change="getGroupUserList"></add-user-to-group-component>
    </el-dialog>
  </div>
</template>

<script>
  import papa from 'papaparse'
  import api from '../../api.js'
  import utils from '@/utils/utils'
  import AddUserToGroupComponent from './AddUserToGroup.vue'

  export default {
    name: 'GroupUserList',
    components: {
      AddUserToGroupComponent
    },
    data () {
      return {
        groupId: '',
        // 一页显示的用户数
        pageSize: 10,
        // 用户总数
        total: 0,
        // 用户列表
        groupUserList: [],
        userList: [],
        // 搜索关键字
        keyword: '',
        // 当前用户model
        user: {},
        loadingGroupUserTable: false,
        addUserDialogVisible: false,
        // 当前页码
        currentPage: 0,
        selectedDeleteUsers: []
      }
    },
    mounted () {
      this.routeName = this.$route.name
      this.groupId = this.$route.params.groupId
      this.getGroupUserList(1)
    },
    methods: {
      // 切换页码回调
      currentChange (page) {
        this.currentPage = page
        this.getGroupUserList(page)
      },
      // 获取用户列表
      // getUserList (page) {
      //   this.loadingUserTable = true
      //   api.getUserList((page - 1) * this.pageSize, this.pageSize, this.keyword).then(res => {
      //     this.loadingUserTable = false
      //     this.total = res.data.data.total
      //     this.userList = res.data.data.results
      //   }, res => {
      //     this.loadingUserTable = false
      //   })
      // },
      getGroupUserList (page) {
        this.loadingGroupUserTable = true
        api.getGroupUserList(this.groupId, (page - 1) * this.pageSize, this.pageSize, this.keyword).then(res => {
          this.loadingGroupUserTable = false
          this.total = res.data.data.total
          this.groupUserList = res.data.data.results
        }, res => {
          this.loadingGroupUserTable = false
        })
      },
      updateGroupUser (userId, userType) {
        let data = {group_id: this.groupId, user_id: userId, user_type: userType}
        api.updateGroupUser(data)
      },
      deleteGroupUsers (ids) {
        this.$confirm('Sure to delete the user from this group? ', 'confirm', {
          type: 'warning'
        }).then(() => {
          api.deleteGroupUsers(this.groupId, ids).then(res => {
            this.getGroupUserList(this.currentPage)
          }).catch(() => {
            this.getGroupUserList(this.currentPage)
          })
        }, () => {
        })
      },
      handleSelectionChange (val) {
        this.selectedDeleteUsers = val
      }
    },
    computed: {
      selectedDeleteUserIDs () {
        let ids = []
        for (let user of this.selectedDeleteUsers) {
          ids.push(user.user_id)
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
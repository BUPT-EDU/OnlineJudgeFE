<template>
  <div>
    <div slot="header">
        <el-row :gutter="20">
          <el-col :span="8">
            <el-button v-show="selectedUsers.length"
                       type="warning" icon="plus"
                       @click="addUsers(selectedUserIDs)">Add
            </el-button>
          </el-col>
          <el-col :span="selectedUsers.length ? 16: 24">
            <el-input v-model="keyword" prefix-icon="el-icon-search" placeholder="Keywords"></el-input>
          </el-col>
        </el-row>
      </div>
    <el-table
    v-loading="loadingTable"
    element-loading-text="loading"
    @selection-change="handleSelectionChange"
    ref="table"
    :data="userList"
    style="width: 100%">
    <el-table-column type="selection" width="55"></el-table-column>

    <el-table-column prop="id" label="ID"></el-table-column>

    <el-table-column prop="username" label="Username"></el-table-column>

    <!-- <el-table-column fixed="right" label="Option" width="200">
        <template slot-scope="{row}">
        <icon-btn name="Edit" icon="edit" @click.native="openUserDialog(row.id)"></icon-btn>
        <icon-btn name="Delete" icon="trash" @click.native="deleteUsers([row.id])"></icon-btn>
        </template>
    </el-table-column> -->
    <el-table-column
        label="option"
        align="center"
        width="100"
        fixed="right">
        <template slot-scope="{row}">
          <icon-btn icon="plus" name="Add the user"
                    @click.native="addUsers([row.id])"></icon-btn>
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
  </div>
</template>

<script>
  import api from '@admin/api'
  
  export default {
    name: 'add-group-user',
    props: ['groupID'],
    data () {
      return {
        // 一页显示的用户数
        pageSize: 10,
        // 用户总数
        total: 0,
        // 用户列表
        userList: [],
        // 搜索关键字
        keyword: '',
        loadingTable: false,
        // 当前页码
        currentPage: 0,
        selectedUsers: []
      }
    },
    mounted () {
      this.getUserList(1)
    },
    methods: {
      // 切换页码回调
      currentChange (page) {
        this.currentPage = page
        this.getUserList(page)
      },
      // 获取用户列表
      getUserList (page) {
        this.loadingTable = true
        api.getUserList((page - 1) * this.pageSize, this.pageSize, this.keyword).then(res => {
          this.loadingTable = false
          this.total = res.data.data.total
          this.userList = res.data.data.results
        }, res => {
          this.loadingTable = false
        })
      },
      addUsers (ids) {
        // todo ##################################################
        // console.log(ids)
        // console.log(this.groupID)
        api.addGroupUsers(this.groupID, ids).then(res => {
          this.$emit('on-change')
        }, res => {
        })
      },
      handleSelectionChange (val) {
        this.selectedUsers = val
      }
    },
    computed: {
      selectedUserIDs () {
        let ids = []
        for (let user of this.selectedUsers) {
          ids.push(user.id)
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

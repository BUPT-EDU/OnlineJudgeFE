<template>
  <div>
    <div slot="header">
        <el-row :gutter="20">
          <el-col :span="8">
            <el-button v-show="selectedGroups.length"
                       type="warning" icon="plus"
                       @click="addGroups(selectedGroupIDs)">Add
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

    <el-table-column prop="groupname" label="Groupname"></el-table-column>

    <el-table-column
        label="option"
        align="center"
        width="100"
        fixed="right">
        <template slot-scope="{row}">
          <icon-btn icon="plus" name="Add the group"
                    @click.native="addGroups([row.id])"></icon-btn>
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
    name: 'add-contest-group',
    props: ['contestID'],
    data () {
      return {
        // 一页显示的用户数
        pageSize: 10,
        // 用户总数
        total: 0,
        // 用户列表
        groupList: [],
        // 搜索关键字
        keyword: '',
        loadingTable: false,
        // 当前页码
        currentPage: 0,
        selectedGroups: []
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
      // 获取用户列表
      getGroupList (page) {
        this.loadingTable = true
        api.getGroupList((page - 1) * this.pageSize, this.pageSize, this.keyword).then(res => {
          this.loadingTable = false
          this.total = res.data.data.total
          this.groupList = res.data.data.results
        }, res => {
          this.loadingTable = false
        })
      },
      addGroups (ids) {
        // todo ##################################################
        // console.log(ids)
        // console.log(this.groupID)
        api.addContestGroups(this.contestID, ids).then(res => {
          this.$emit('on-change')
        }, res => {
        })
      },
      handleSelectionChange (val) {
        this.selectedGroups = val
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

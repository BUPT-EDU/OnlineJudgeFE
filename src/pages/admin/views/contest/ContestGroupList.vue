<template>
  <div class="view">
    <Panel :title="$t('m.Contest_Group') ">
      <div slot="header">
        <el-row :gutter="20">
          <el-col :span="8">
            <el-button v-show="selectedDeleteGroups.length"
                       type="warning" icon="el-icon-fa-trash"
                       @click="deleteContestGroups(selectedDeleteGroupIDs)">Delete
            </el-button>
          </el-col>
          <el-col :span="selectedDeleteGroups.length ? 16: 24">
            <el-input v-model="keyword" prefix-icon="el-icon-search" placeholder="Keywords"></el-input>
          </el-col>
        </el-row>
      </div>
      <el-table
        v-loading="loadingTable"
        element-loading-text="loading"
        @selection-change="handleSelectionChange"
        ref="table"
        :data="contestGroupList"
        style="width: 100%">
        <el-table-column type="selection" width="55"></el-table-column>

        <el-table-column prop="id" label="ID"></el-table-column>

        <el-table-column prop="groupname" label="Group Name"></el-table-column>

        <el-table-column fixed="right" label="Option" width="100">
          <template slot-scope="scope">
            <icon-btn name="Delete" icon="trash" @click.native="deleteContestGroups([scope.row.id])"></icon-btn>
          </template>
        </el-table-column>

      </el-table>
      <div class="panel-options">
        <el-button v-if="contestId" type="primary"
                   size="big" icon="el-icon-plus"
                   @click="addGroupDialogVisible = true">Add Group
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
    <el-dialog title="Add Contest Group"
      width="60%"
      :visible.sync="addGroupDialogVisible"
      @close-on-click-modal="false">
      <add-group-to-contest-component :contestID="contestId" @on-change="getContestGroupList"></add-group-to-contest-component>
    </el-dialog>
  </div>
</template>

<script>
  import papa from 'papaparse'
  import api from '../../api.js'
  import utils from '@/utils/utils'
  import AddGroupToContestComponent from './AddGroupToContest.vue'
  
  export default {
    name: 'add-contest-group',
    components: {
      AddGroupToContestComponent
    },
    data () {
      return {
        contestId: '',
        // 一页显示的用户数
        pageSize: 10,
        // 用户总数
        total: 0,
        // 用户列表
        contestGroupList: [],
        // 搜索关键字
        keyword: '',
        loadingTable: false,
        // 当前页码
        currentPage: 0,
        selectedDeleteGroups: [],
        addGroupDialogVisible: false
      }
    },
    mounted () {
      this.routeName = this.$route.name
      this.contestId = this.$route.params.contestId
      this.getContestGroupList(1)
    },
    methods: {
      // 切换页码回调
      currentChange (page) {
        this.currentPage = page
        this.getContestGroupList(page)
      },
      // 获取用户列表
      getContestGroupList (page) {
        this.loadingTable = true
        api.getContestGroupList(this.contestId, (page - 1) * this.pageSize, this.pageSize, this.keyword).then(res => {
          this.loadingTable = false
          this.total = res.data.data.total
          this.contestGroupList = res.data.data.results
        }, res => {
          this.loadingTable = false
        })
      },
      deleteContestGroups (ids) {
        this.$confirm('Sure to delete the group from this contest? ', 'confirm', {
          type: 'warning'
        }).then(() => {
          api.deleteContestGroups(this.contestId, ids).then(res => {
            this.getContestGroupList(this.currentPage)
          }).catch(() => {
            this.getContestGroupList(this.currentPage)
          })
        }, () => {
        })
      },
      handleSelectionChange (val) {
        this.selectedDeleteGroups = val
      }
    },
    computed: {
      selectedDeleteGroupIDs () {
        let ids = []
        for (let group of this.selectedDeleteGroups) {
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

<template>
  <div class="strategy-group">
    <div class="strategy-group-search b-wrapper">
     <el-form :inline="true" size="medium">
       <el-form-item label="应用组">
         <el-select v-model="applyGroup" placeholder="应用组">
           <p v-for="(item, index) in applyGroupData" :key="index">
             <el-option :label="item" :value="item"></el-option>
           </p>
         </el-select>
       </el-form-item>
       <el-form-item label="应用名称">
         <el-select v-model="applyName" placeholder="应用名称">
           <p v-for="(item, index) in anyApplyData" :key="index">
             <el-option :label="item" :value="item"></el-option>
           </p>
         </el-select>
       </el-form-item>
       <el-form-item label="降级预案">
         <el-input v-model="sdsScheme" placeholder="降级预案" @keyup.enter.native="doSearch"></el-input>
       </el-form-item>
     </el-form>
      <div class="condition-btn-wrapper">
        <el-button type="primary" @click="doSearch" icon="el-icon-search">查 找</el-button>
        <el-button type="primary" @click="addSdsScheme" icon="el-icon-plus">新增降级预案</el-button>
      </div>
    </div>
    <div class="strategy-group-table b-wrapper">
      <el-table
        :data="tableData.data"
        style="width: 100%">
        <el-table-column
          prop="appGroupName"
          label="应用组"
          width="180"
          :show-overflow-tooltip="true">
        </el-table-column>
        <el-table-column
          prop="appName"
          label="应用名称"
          width="300"
          :show-overflow-tooltip="true">
        </el-table-column>
        <el-table-column
          prop="sdsSchemeName"
          label="降级预案"
          width="180"
          :show-overflow-tooltip="true">
        </el-table-column>
        <el-table-column
          prop="operatorName"
          label="更新人"
          >
          <template slot-scope="scope">
            <el-popover trigger="hover" placement="top">
              <p>更新时间: {{ scope.row.modifiedTime }}</p>
              <div slot="reference" class="name-wrapper">
                <el-tag type="warning" color="#fff">{{scope.row.operatorName || '测试者'}}</el-tag>
              </div>
            </el-popover>
          </template>
        </el-table-column>
        <el-table-column
          prop="creatorName"
          label="创建人"
          >
          <template slot-scope="scope">
            <el-popover trigger="hover" placement="top">
              <p>创建时间: {{ scope.row.createTime }}</p>
              <div slot="reference" class="name-wrapper">
                <el-tag color="#fff">{{scope.row.creatorName || '创建者'}}</el-tag>
              </div>
            </el-popover>
          </template>
        </el-table-column>
        <el-table-column
          prop="modifiedTime"
          label="更新时间"
          width="180"
          :show-overflow-tooltip="true">
        </el-table-column>
        <el-table-column
                prop="createTime"
                label="创建时间"
                width="180"
                :show-overflow-tooltip="true">
        </el-table-column>
        <el-table-column label="操作" width="220">
          <template slot-scope="scope">
            <el-button
              size="mini"
              @click="handleEdit(scope.row)">编辑</el-button>
            <el-button
              size="mini"
              @click="handleClone(scope.row)">克隆</el-button>
            <el-button
              size="mini"
              type="danger"
              @click="handleDelete(scope.row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <div class="application-group-pagination">
        <el-pagination
          background
          small
          layout="prev, next"
          @current-change="currentChangeData"
          :total="tableData.total"
          :page-size="tableData.ps"
          >
        </el-pagination>
      </div>
    </div>
   
    <el-dialog title="新增降级预案" :visible.sync="dialogSdsScheme">
      <el-form :inline="true" class="add-strategy-group" label-position="right" label-width="120px">
        <p class="of-application-group">
          <el-form-item label="应用组名称">
            <el-select v-model="addApplyGroupName" placeholder="">
              <p v-for="(item, index) in applyGroupData" :key="index">
                <el-option :label="item" :value="item"></el-option>
              </p>
            </el-select>
          </el-form-item>
        </p>
        <p class="of-strategy-group">
          <el-form-item label="应 用 名 称">
            <el-select v-model="addApplyName" placeholder="">
              <p v-for="(item, index) in someApplyData" :key="index">
                <el-option :label="item" :value="item"></el-option>
              </p>
            </el-select>
          </el-form-item>
        </p>
        <p class="add-strategy-group-name">
          <el-form-item label="降级预案名称">
            <el-input v-model="form.sdsSchemeName"></el-input>
          </el-form-item>
          <el-form-item>
            <el-tooltip class="item-warning" effect="dark" content="注意：只能使用汉字、字母、数字、-、_" placement="bottom">
              <i class="el-icon-question"></i>
            </el-tooltip>
          </el-form-item>
        </p>
        <!-- <p class="of-application-group">
          <el-form-item label="所用降级预案">
            <el-select v-model="form.sdsScheme" placeholder="">
              <el-option label="降级预案1" value="1"></el-option>
              <el-option label="降级预案2" value="2"></el-option>
            </el-select>
          </el-form-item>
        </p> -->
        <p class="dialog-footer">
          <el-form-item>
            <el-button type="primary" @click="closeDialogAddSdsSchemey">取消</el-button>
            <el-button type="primary" @click="doSubmitAddSdsScheme">确定</el-button>
          </el-form-item>
        </p>
      </el-form>
    </el-dialog>
    <el-dialog title="修改降级预案" :visible.sync="dialogEditSdsScheme">
      <el-form :inline="true" class="edit-apply" label-position="right" label-width="120px">
        <p class="of-application-group">
          <el-form-item label="原降级预案名称">
            <el-input v-model="editRowData.sdsSchemeName" disabled></el-input>
          </el-form-item>
        </p>
        <p class="edit-apply-name">
          <el-form-item label="新降级预案名称">
            <el-input v-model="editSdsSchemeName"></el-input>
          </el-form-item>
          <el-form-item>
            <el-tooltip class="item-warning" effect="dark" content="注意：只能使用汉字、字母、数字、-、_" placement="bottom">
              <i class="el-icon-question"></i>
            </el-tooltip>
          </el-form-item>
        </p>
        <p class="dialog-footer">
          <el-form-item>
            <el-button type="primary" @click="closeDialogEditSdsScheme">取消</el-button>
            <el-button type="primary" @click="doSubmitEditSdsScheme">确定</el-button>
          </el-form-item>
        </p>
      </el-form>
    </el-dialog>
    <el-dialog title="克隆降级预案" :visible.sync="dialogCloneSdsScheme">
      <el-form :inline="true" class="edit-apply" label-position="right" label-width="120px">
        <p class="of-application-group">
          <el-form-item label="原应用组名称">
            <el-select v-model="cloneRowData.appGroupName" placeholder="" disabled>
              <p v-for="(item, index) in applyGroupData" :key="index">
                <el-option :label="item" :value="item"></el-option>
              </p>
            </el-select>
          </el-form-item>
        </p>
        <p class="of-strategy-group">
          <el-form-item label="应 用 名 称">
            <el-select v-model="cloneRowData.appName" placeholder="" disabled>
              <p v-for="(item, index) in someApplyData" :key="index">
                <el-option :label="item" :value="item"></el-option>
              </p>
            </el-select>
          </el-form-item>
        </p>
        <p class="of-application-group">
          <el-form-item label="原降级预案名称">
            <el-select v-model="cloneRowData.sdsSchemeName" placeholder="" disabled>
              <p v-for="(item, index) in allSdsSchemeData" :key="index">
                <el-option :label="item" :value="item"></el-option>
              </p>
            </el-select>
          </el-form-item>
        </p>
        <p class="edit-apply-name">
          <el-form-item label="新降级预案名称">
            <el-input v-model="newSdsSchemeName"></el-input>
          </el-form-item>
          <el-form-item>
            <el-tooltip class="item-warning" effect="dark" content="注意：只能使用汉字、字母、数字、-、_" placement="bottom">
              <i class="el-icon-question"></i>
            </el-tooltip>
          </el-form-item>
        </p>
        <p class="dialog-footer">
          <el-form-item>
            <el-button type="primary" @click="closeDialogCloneSdsScheme">取消</el-button>
            <el-button type="primary" @click="doSubmitCloneSdsScheme">确定</el-button>
          </el-form-item>
        </p>
      </el-form>
    </el-dialog>
  </div>
</template>

<style lang="scss">
  .strategy-group {
    .condition-btn-wrapper {
      margin-top: 20px;
    }
    .top-line {
      height: 1px;
      background: #f6f6f6;
      margin: 0 -20px;
    }
    .strategy-group-table {
      margin-top: 20px;
    }
  }
  
  .application-group-pagination {
    text-align: center;
    margin-top: 20px;
  }
  .el-pagination.is-background .el-pager li:not(.disabled).active {
    border: none;
  }
  .el-table--enable-row-transition .el-table__body td {
    text-align: center;
  }
  .el-table tr th {
    text-align: center;
  }
  .strategy-group-search .el-button {
    padding: 10px 14px;
  }
  .strategy-group-search .top-btn-right {
    float: right;
  }
  .strategy-group-search .el-form-item {
    margin-bottom: 0;
  }
  .el-dialog__header {
    padding: 20px;
    border-bottom: 1px solid #dcdfe6;
  }
  .dialog-footer {
    padding-left: 80px;
  }
</style>

<script>
    /* eslint-disable */
    import Api from '../api/api'
    import TimeFormat from '../utils/timeFormat'

    let mockTableData = [
    {
      applicationGroupName: '青桔',
      applyName: 'apply1',
      usingPolicyGroup: 'group1',
      operatorTime: '2016-05-02',
      operatorName: '小明',
      createTime: '2016-04-02'
    },
    {
      applicationGroupName: '王者荣耀',
      applyName: 'apply2',
      usingPolicyGroup: 'group2',
      operatorTime: '2016-06-02',
      operatorName: '小彬',
      createTime: '2016-04-02'
    },
    {
      applicationGroupName: '今日头条',
      applyName: 'apply3',
      usingPolicyGroup: 'group3',
      operatorTime: '2016-07-02',
      operatorName: '小杨',
      createTime: '2016-04-02'
    },
    {
      applicationGroupName: '美团外卖',
      applyName: 'apply4',
      usingPolicyGroup: 'group4',
      operatorTime: '2016-08-02',
      operatorName: '小小',
      createTime: '2016-06-02'
    },
    {
      applicationGroupName: '青桔',
      applyName: 'apply5',
      usingPolicyGroup: 'group5',
      operatorTime: '2016-05-02',
      operatorName: '小明',
      createTime: '2016-04-02'
    },
    {
      applicationGroupName: '王者荣耀',
      applyName: 'apply6',
      usingPolicyGroup: 'group6',
      operatorTime: '2016-06-02',
      operatorName: '小彬',
      createTime: '2016-04-02'
    },
    {
      applicationGroupName: '今日头条',
      applyName: 'apply7',
      usingPolicyGroup: 'group7',
      operatorTime: '2016-07-02',
      operatorName: '小杨',
      createTime: '2016-04-02'
    },
    {
      applicationGroupName: '美团外卖',
      applyName: 'apply8',
      usingPolicyGroup: 'group8',
      operatorTime: '2016-08-02',
      operatorName: '小小',
      createTime: '2016-06-02'
    }
  ]
  let mockSdsScheme = ['strategy1', 'strategy2', 'strategy3', 'strategy4', 'strategy5', 'strategy6']
  export default {
    data () {
      return {
        cloneRowData: {},
        newSdsSchemeName: '',
        allSdsSchemeData: [],
        primarySdsSchemeName: '',
        dialogCloneSdsScheme: false,
        editSdsSchemeName: '',
        anyApplyData: [],
        applyGroup: '',
        applyName: '',
        sdsScheme: '',
        someApplyData: [],
        addApplyName: '',
        addApplyGroupName: '',
        sdsSchemeData: mockSdsScheme,
        editSdsScheme: '',
        editApplyName: '',
        editRowData: {},
        dialogEditApply: false,
        dialogSdsScheme: false,
        dialogEditSdsScheme: false,
        applicationGroup: "",
        tableData: {
          data: [],
          ps: 8,
          pn: 1,
          total: 1
        },
        applyGroupData: [],
        form: {
          applyName: '',
          applyGroup: '',
          sdsSchemeName: ''
        }
      }
    },
    created () {
      this.appgroupListall()
      // this.sdsschemeListall()
    },
    computed: {
    },
    watch: {
      //监听应用组变化 获取该应用组下所有应用
      'addApplyGroupName': function(newVal, oldVal) {
        this.addApplyName = ''
        console.log('watch addApplyGroupName', newVal)
        this.getAppinfoListall(newVal).then((res) => {
          if(res.code === 200) {
            this.someApplyData = res.data
          } else {
            this.$message({
              message: res.msg,
              type: 'warning'
            })
          }
        }).catch((err) => {
          this.$message({
            message: "查询该应用组下的所有应用失败",
            type: 'warning'
          })
        })
      },
      'applyGroup': function(newVal, oldVal) {
        this.applyName = ''
        this.getAppinfoListall(newVal).then((res) => {
          if(res.code === 200) {
            this.anyApplyData = res.data
          } else {
            this.$message({
              message: res.msg,
              type: 'warning'
            })
          }
        }).catch((err) => {
          this.$message({
            message: "查询该应用组下的所有应用失败",
            type: 'warning'
          })
        })
      }
      // 'addApplyName': function(newVal, oldVal) {
      //   let params = {
      //     appGroupName: this.addApplyGroupName,
      //     appName: newVal
      //   }
      //   Api.sdsschemeListall(params).then((res) => {
      //     if (res.code === 200) {
      //       console.log('sdsschemeListall', res.data)
      //       this.allSdsSchemeData = res.data
      //     } else {
      //       this.$message({
      //         message: res.msg,
      //         type: 'warning'
      //       })
      //     }
      //   }).catch((err) => {
      //     this.$message({
      //       message: "查询该应用下面的所有降级预案失败",
      //       type: 'warning'
      //     })
      //   })
      // }
    },
    methods: {
      //克隆降级预案请求接口逻辑
      doSubmitCloneSdsScheme () {
        let params = {
          appGroupName: this.cloneRowData.appGroupName,
          appName: this.cloneRowData.appName,
          sdsSchemeName: this.cloneRowData.sdsSchemeName,
          newSdsSchemeName: this.newSdsSchemeName,
          operatorId: this.cloneRowData.operatorId
        }
        Api.sdsschemeClone(params).then((res) => {
          if (res.code === 200) {
            console.log('sdsschemeClone', res.data)
            this.$message({
              message: "克隆降级预案成功",
              type: 'success'
            })
          } else {
            this.$message({
              message: res.msg,
              type: 'warning'
            })
          }
        }).catch((err) => {
          this.$message({
            message: "克隆降级预案失败",
            type: 'warning'
          })
        })
        this.dialogCloneSdsScheme = false
      },
      //关闭克隆降级预案
      closeDialogCloneSdsScheme () {
        this.dialogCloneSdsScheme = false
      },
      //克隆降级预案逻辑
      handleClone (rowData) {
        this.dialogCloneSdsScheme = true;
        this.cloneRowData = rowData
      },
      //新增降级预案逻辑
      doSubmitAddSdsScheme () {
        let params = {
          appGroupName: this.addApplyGroupName,
          appName: this.addApplyName,
          sdsSchemeName: this.form.sdsSchemeName
        };
        Api.sdsschemeAdd(params).then((res) => {
          if(res.code === 200) {
            console.log(res.data)
            this.$message({
              message: '新增降级预案成功',
              type: 'success'
            })
          } else {
            this.$message({
              message: res.msg,
              type: 'warning'
            })
          }
        }).catch((err) => {
          this.$message({
            message: "新增降级预案失败",
            type: 'warning'
          })
        })
        this.dialogSdsScheme = false;
      },
      //查询某应用组下面的所有应用
      getAppinfoListall (appGroup) {
        let params = {
          appGroupName: appGroup
        }
        return Api.appinfoListall(params)
      },
      //下一页前一页查询
      currentChangeData(page) {
        this.sdsschemeListpage(page).then((res) => {
          if(res.code === 200) {
            res.data.forEach(function(item){
              item.createTime = TimeFormat.utcTimeFormat(item.createTime);
              item.modifiedTime = TimeFormat.utcTimeFormat(item.modifiedTime);
            });
            this.tableData.data = res.data;
            if (this.tableData.data.length < this.tableData.ps) {
              this.tableData.total = page * this.tableData.ps
            }
          } else {
            this.$message({
              message: res.msg,
              type: 'warning'
            })
          }
        }).catch((err) => {
          this.$message({
            message: "分页查询应用失败",
            type: 'warning'
          })
        })
      },
      // //查询所有降级预案
      // sdsschemeListall () { // todo
      //   let params = {
      //     appGroupName: 'apply',
      //     appName: 'yy1'
      //   }
      //   Api.sdsschemeListall(params).then((res) => {
      //     if (res.code === 200) {
      //       console.log('sdsschemeListall', res.data)
      //     } else {
      //       this.$message({
      //         message: res.msg,
      //         type: 'warning'
      //       })
      //     }
      //   }).catch((err) => {
      //     this.$message({
      //       message: "查询所有降级预案失败",
      //       type: 'warning'
      //     })
      //   })
      // },
      //修改降级预案
      doSubmitEditSdsScheme () {
        let params = {
          appGroupName: this.editRowData.appGroupName,
          appName: this.editRowData.appName,
          sdsSchemeName: this.editRowData.sdsSchemeName,
          newSdsSchemeName: this.editSdsSchemeName,
          operatorId: this.editRowData.operatorId
        }
        Api.sdsschemeEdit(params).then((res) => {
          if(res.code === 200) {
            this.$message({
              message: "修改降级预案成功",
              type: 'success'
            })
          } else {
            this.$message({
              message: res.msg,
              type: 'warning'
            })
          }
        }).catch((err) => {
          console.log('err', err)
          this.$message({
            message: "编辑应用失败",
            type: 'warning'
          })
        })
        this.dialogEditSdsScheme = false
      },
      closeDialogEditSdsScheme () {
        this.dialogEditSdsScheme = false
      },
      appgroupListall () {
        let params = {
        }
        Api.appgroupListall(params).then((res) => {
          if(res.code === 200) {
            // res.data.forEach(function(item){
            //   item.createTime = TimeFormat.timeFormat(item.createTime)
            //   item.modifiedTime = TimeFormat.timeFormat(item.modifiedTime)
            // })
            console.log('appgroupListall', res.data)
            this.applyGroupData = res.data
          } else {
            this.$message({
              message: res.msg,
              type: 'warning'
            })
          }
        }).catch((err) => {
          this.$message({
            message: "查询应用失败",
            type: 'warning'
          })
        })
      },
      addSdsScheme() {
        this.dialogSdsScheme = true
      },
      closeDialogAddSdsSchemey () {
        this.dialogSdsScheme = false
      },
      handleEdit(row) {
        this.dialogEditSdsScheme = true
        this.editRowData = row
      },
      //删除降级预案
      handleDelete(rowData) {
        let params = {
          appGroupName: rowData.appGroupName,
          appName: rowData.appName,
          sdsSchemeName: rowData.sdsSchemeName,
          operatorId: rowData.operatorId
        }
        this.$confirm('确定删除该降级预案吗?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          Api.deleteStrategygroup(params).then((res) => {
            if(res.code === 200) {
              this.$message({
                message: "删除降级预案成功",
                type: 'success'
              })
            } else {
              this.$message({
                message: res.msg,
                type: 'warning'
              })
            }
          }).catch((err => {
            console.log('err', err)
            this.$message({
              message: "删除降级预案失败",
              type: 'warning'
            })
          }))
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          });
        });
      },
      // 分页查询降级预案
      sdsschemeListpage(page, pageSize) {
        let params = {
          appGroupName: this.applyGroup,
          appName: this.applyName,
          sdsSchemeName: this.sdsScheme,
          page: page || 1,
          pageSize: pageSize || this.tableData.ps
        }
        return Api.sdsschemeListpage(params)
      },
      //查询应用
      doSearch() {
        console.log('doSearch', this.applyGroupData)
         this.sdsschemeListpage().then((res) => {
          if(res.code === 200) {
            res.data.forEach(function(item){
              item.createTime = TimeFormat.utcTimeFormat(item.createTime)
              item.modifiedTime = TimeFormat.utcTimeFormat(item.modifiedTime)
            })
            this.tableData.data = res.data
            if (this.tableData.data.length < this.tableData.ps) {
              this.tableData.total = this.tableData.data.length
            } else {
              this.tableData.total = 100
            }
          } else {
            this.$message({
              message: res.msg,
              type: 'warning'
            })
          }
        }).catch((err) => {
          this.$message({
            message: "查询降级预案失败",
            type: 'warning'
          })
        })
      }
    }
  }
</script>

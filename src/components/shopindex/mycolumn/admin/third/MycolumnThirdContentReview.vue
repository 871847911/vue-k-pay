<template>
  <div class="my-column-third-content-review">
    <div>
      <div class="serch-header">
          <div class="screening">
            <span :class="[currentStatus===20?'active':'']" @click="screening(20)">待审核</span>
            <span>.</span>
            <span :class="[currentStatus===50?'active':'']" @click="screening(50)">已审核</span>
          </div>
          <div class="search">
            <!-- <el-select v-model="searchData.onStatus" placeholder="全部状态" class="el-select-custom">
                <el-option
                    v-for="item in newOptions"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value">
                </el-option>
            </el-select> -->
            <el-input type="text" placeholder="标题名称" v-model="inputValue"></el-input>
            <button class="search-btn" @click="search">搜索</button>
          </div>
      </div>
      
        <div style="padding:0 20px;" v-show="currentStatus===20">
          <el-table
              :data="list"
              stripe
              style="width: 100%">
              <el-table-column
                  row-class-name
                  label="标题"
                  class-name="clomun-left">
                <template slot-scope="scope">
                  <p>{{scope.row.contentName}}</p>
                </template>
              </el-table-column>
              <el-table-column
                  align="center"
                  prop="columnName"
                  label="专栏"
                  width="250"
                  :show-overflow-tooltip="true">
              </el-table-column>
              <el-table-column
                  align="center"
                  prop="createUser"
                  label="作者"
                  width="250">
              </el-table-column>
              <!-- <el-table-column
                  align="center"
                  label="状态">
                  <template slot-scope="scope">
                    {{scope.row.auditStatus===10||20?"待审核":"已审核"}}
                  </template>
              </el-table-column> -->
              <el-table-column
                  align="center"
                  prop="modifyDate"
                  label="提交时间"
                  width="250">
              </el-table-column>
              <el-table-column
                  align="center" label="操作" width="150">
                <template slot-scope="scope">
                    <el-button
                    size="mini"
                    @click="handleLook(scope.$index, scope.row)">查看</el-button>
                </template>
              </el-table-column>
          </el-table>
        </div>

        <div style="padding:0 20px;" v-show="currentStatus===50">
          <el-table
              :data="list"
              stripe
              style="width: 100%">
              <el-table-column
                  row-class-name
                  label="标题"
                  class-name="clomun-left">
                <template slot-scope="scope">
                  <p>{{scope.row.contentName}}</p>
                </template>
              </el-table-column>
              <el-table-column
                  align="center"
                  prop="columnName"
                  label="专栏"
                  width="250"
                  :show-overflow-tooltip="true">
              </el-table-column>
              <el-table-column
                  align="center"
                  prop="createUser"
                  label="作者"
                  width="250">
              </el-table-column>
              <el-table-column
                  align="center"
                  label="状态">
                  <template slot-scope="scope">
                    <span v-if="scope.row.auditStatus === 30">
                      <i class="iconfont icon-zitikuicon-" style="color:#6CDDC7;"></i>
                      已通过
                    </span>
                    <span v-if="scope.row.auditStatus === 40">
                      <i class="iconfont icon-zitikuicon-" style="color:#F42C21;"></i>
                      未通过
                    </span>
                  </template>
              </el-table-column>
              <el-table-column
                  align="center"
                  prop="auditDate"
                  label="审核时间"
                  width="250">
              </el-table-column>
              <el-table-column
                  align="center" label="操作" width="150">
                <template slot-scope="scope">
                    <el-button
                    size="mini"
                    @click="handleLook(scope.$index, scope.row)">查看</el-button>
                </template>
              </el-table-column>
          </el-table>
        </div>
        <div class="pagination">
          <el-pagination
              background
              @size-change="handleSizeChange"
              @current-change="handleCurrentChange"
              :current-page.sync="currentPage"
              :page-size="pageSize"
              layout="prev, pager, next, sizes, jumper"
              :page-sizes="[5, 10, 15, 25]"
              :total="total">
          </el-pagination>
        </div>
    </div>
  </div>
</template>

<script>
import api from 'fetch/api'
import {mapGetters} from 'vuex'

const UNVERIFY = 10;
const VERIFYING = 20;
const VERIFYED = 50;

export default {
  name: 'MycolumnThirdContentReview',
  data () {
    return {
      list:[], // 列表数据
      newOptions: [{
        value: '0',
        label: '未上架'
      },{
        value:'1',
        label:'已上架'
      }],
      currentPage: 1, // 查询的是第几页
      pageSize: 10, // 每页条数
      total: 0, // 列表数据总条数
      activeIdx: 0,
      active:'',
      actionNames: ['专栏列表', '专栏审核', '内容审核'],
      currentStatus: 20,
      inputValue: ''
    }
  },

  computed: {
    ...mapGetters([
      'imageHead'
    ])
  },

  components: {
  },

  created () {
    this.init()
  },
  

  methods: {
    // 初始化数据
    init(){
      return api.findColumnContentAudit({
        auditStatus: this.currentStatus,
        contentName: this.inputValue,
        page: this.currentPage,
        pageSize: this.pageSize 
      }).then(res => {
          res.data.list.map(v => {
            v.picUrl = this.imageHead + v.contentUrl
            return v
          })

          this.total = res.data.total
          this.list = res.data.list
      })
    },
    search(){
      this.init()
    },

    // 筛选
    screening(status) { 
      this.currentStatus=status
      this.init()
    },

    handleDropDown(e,item){
      switch (e) {
        case "0":
          this.$router.push({
            name:'MycolumnListStoreView',
            query:{
              id:item.id
            }
          })
          break;
        case "2":
          this.shelves(item.id,item.onStatus)
          break; 
      }
    },

    // 查看按钮
    handleLook(i, row){
      this.$router.push({
        name: 'MycolumnThirdContentAuditDetail', 
        query: { 
          contentId: row.id
        }
      })
    },

     // 改变每页显示的条数重新渲染列表
    handleSizeChange(val) {
      this.pageSize = val
      this.init()
    },

    // 改变页码重新渲染列表
    handleCurrentChange(val) {
      this.currentPage = val
      this.init()
    },

    // 序号补0
    indexMethod(index){
      const cur = this.currentPage
      const size = this.pageSize
      
      index++
      if(cur > 1){
        index = (cur - 1) * size + index
      }

      return index;
    }
  }
}
</script>

<style lang="stylus" scoped>
@import "~assets/styles/mixins"
@import "~assets/styles/varibles"

.my-column-third-content-review
  padding-bottom .2rem
  .pagination
    display flex
    justify-content flex-end
    margin 0.2rem 0.2rem 0 0
.serch-header
  height 0.78rem
  flexBetween()
  .screening
    flex()
    height 0.4rem
    line-height 0.4rem
    padding-left 0.2rem
    span 
      cursor pointer
      min-width 50px
      height 14px
      text-align center
      font-size 14px
      font-weight 400
      line-height 14px
      &:nth-child(2)
        line-height 7px
    span.active
      color rgba(108,221,199,1)
  .search
    width 100%
    height .4rem
    position relative
    display flex
    flex-direction row
    justify-content flex-end
    padding-right 0.2rem
    align-items center
    .el-input
      width 3rem
      margin-right 5px
    .search-btn
      height .4rem
      fontCenter(.4rem)
      font-size .14rem
      color #fff
      border-radius .05rem
      background $color-theme
      width .7rem
      cursor pointer
>>>.clomun-center
  text-align center
>>>.clomun-left
  text-align left
  .cell
    display flex
    flex-flow row nowrap
    align-items center
    justify-content flex-start
    img 
      width .6rem
      height .6rem
      padding .1rem .1rem .1rem 0
</style>
<template>
  <div
    class="view"
    v-loading="loading"
    element-loading-text="加载中..."
    element-loading-spinner="el-icon-loading"
  >
    <el-form ref="ruleForm" :inline="true" style="height:50px;">
      <el-form-item>
        <!-- <region-select
          :size="'mini'"
          v-model="regionName"
          @getRegion="getSelectRegion"
        ></region-select> -->
      </el-form-item>
      <el-form-item>
        <el-select v-model="dimension" size="mini" style="width:80px;">
          <el-option value="2" label="日"></el-option>
          <el-option value="3" label="月"></el-option>
          <el-option value="4" label="年"></el-option>
        </el-select>
        <el-date-picker
          size="mini"
          v-model="time"
          v-if="dimension === '2'"
          value-format="yyyyMMdd"
          type="date"
          placeholder="选择日期"
        >
        </el-date-picker>
        <el-date-picker
          size="mini"
          v-else-if="dimension === '3'"
          v-model="time"
          type="month"
          value-format="yyyyMM"
          placeholder="选择月"
        >
        </el-date-picker>
        <el-date-picker
          size="mini"
          value-format="yyyy"
          v-else-if="dimension === '4'"
          v-model="time"
          type="year"
          placeholder="选择年"
        >
        </el-date-picker>
      </el-form-item>

      <el-form-item>
        <el-button
          icon="el-icon-search"
          size="mini"
          @click="handle"
          type="primary"
        >
          查询</el-button
        >
      </el-form-item>
    </el-form>
    <el-divider></el-divider>
    <div class="content">
      <RegionTree class="tree" @nodeClick="nodeClick"></RegionTree>
      <div class="contenttable">
        <CommonTable
          :options="options"
          :dataSource="tableData"
          :columns="columns"
          :treeProps="treeProps"
          :rowKey="'id'"
        ></CommonTable>
      </div>
    </div>
  </div>
</template>

<script>
// import RegionSelect from '@/views/modules/pob/region-select'
import RegionTree from '@/views/common/RegionTree'
import CommonTable from '@/views/common/Table'
import momentTime from '@/utils/time'
export default {
  components: {
    // RegionSelect,
    RegionTree,
    CommonTable
  },
  data() {
    return {
      loading: false,
      regionid: '',
      //   regionName: this.$cookie.get('regionName'),
      dimension: '3',
      time: this.$moment().format('YYYYMM'),
      options: {
        loading: false,
        index: true,
        indexfix: true
      },
      tableData: [],
      columns: [],
      treeProps: { children: 'children' }
    }
  },
  created() {
    this.$nextTick(() => {
      //   this.handle()
    })
  },
  watch: {
    dimension(newValue, oldValue) {
      switch (newValue) {
        case '2':
          this.time = this.$moment().format('YYYYMMDD')
          break
        case '3':
          this.time = this.$moment().format('YYYYMM')

          break
        case '4':
          this.time = this.$moment().format('YYYY')

          break
      }
    },
    time(newValue, oldValue) {
      this.handle()
    }
  },
  methods: {
    //格式化表格列
    formatColumns() {
      let temp = [
        {
          label: '分项名称',
          prop: 'name',
          align: 'left',
          fixed: true,
          width: '200'
        }
      ]
      switch (this.dimension) {
        case '2':
          this.columns = momentTime.hour.map((item, key) => {
            return { label: item, prop: 'a' + key }
          })
          this.columns = temp.concat(this.columns)
          break
        case '3':
          {
            let number = this.$moment(this.time, 'YYYYMM').daysInMonth()
            console.log(number)
            let arr = []
            for (let i = 1; i < number + 1; i++) {
              arr.push(i + '日')
            }
            console.log(arr)
            this.columns = arr.map((item, key) => {
              return { label: item, prop: 'a' + key }
            })
            this.columns = temp.concat(this.columns)
          }

          break
        case '4':
          this.columns = momentTime.month.map((item, key) => {
            return { label: item, prop: 'a' + key }
          })
          this.columns = temp.concat(this.columns)
          break
      }
    },
    getSelectRegion(data) {
      this.regionid = data.id
      this.regionName = data.label
      this.handle()
      //   this.getDataList()
    },
    handle() {
      this.formatColumns()
      this.loading = true
      this.options.loading = true
      let body = {
        deptId: this.regionid,
        time: this.time,
        dimension: this.dimension
      }
      console.log(body)
      this.$api.energyhome.itemstatics(body).then(res => {
        console.log(res)
        let arr = this.columns.map(item => {
          return item.prop
        })
        console.log(arr)
        if (res.data.length > 0) {
          let tableData = res.data.map((item, index) => {
            let obj = {}
            item.yArrays.forEach((item, key) => {
              obj[`a${key}`] = item
            })
            let aaa = Object.assign(
              { name: item.label, id: item.id, parentId: item.parentId },
              obj
            )
            return aaa
          })
          console.log(tableData)
          this.loop(tableData, [], '78')
        } else {
          this.tableData = []
        }
        this.options.loading = false
      })

      setTimeout(() => {
        this.loading = false
      }, 500)
    },
    loop(list, data, parentId) {
      list.forEach(item => {
        if (item.parentId === parentId) {
          let child = Object.assign(item, { children: [] })
          this.loop(list, child.children, item.id)
          data.push(child)
          this.tableData = data
        }
      })
    },
    nodeClick(data) {
      this.regionid = data.id
      this.handle()
    }
  }
}
</script>

<style lang="scss" scoped>
.view {
  height: 100%;
  display: flex;
  flex-direction: column;
  .el-divider--horizontal {
    margin: 0;
  }
  .content {
    margin-top: 10px;
    height: calc(100% - 64px);
    display: flex;
    .tree {
      width: 350px;
    }
    .contenttable {
      overflow: auto;
      flex: 1;
      margin-top: -20px;
    }
  }
}
</style>

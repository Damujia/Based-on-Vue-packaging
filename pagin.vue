<template>
  <div>
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange1"
      :current-page="currpage"
      :page-sizes="[30, 50, 100, 200]"
      :page-size="pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
    ></el-pagination>
  </div>
</template>

<script>
/*
* 分页组件-前端把所有数据进行分页
* <Pagin :getArr="getArr" ref="pagefun" :currpage.sync="currpage" :pageSize.sync="pageSize" :total="total" @excisform="pageform" />
*
*
*/
export default {
  name: 'pagin',
  props: {
    // 所有数据
    getArr: {
      type: Array,
      required: true
    },
    // 每页显示数量
    pageSize: {
      type: Number,
      required: true
    },
    // 页数
    currpage: {
      type: Number,
      required: true
    },
    // 总条数
    total: {
      type: Number,
      required: true
    }
  },
  data () {
    return {
      formworker: []
    }
  },
  methods: {
    handleSizeChange (val) {
      this.formworker = this.excision(this.getArr, val)[this.currpage - 1]
      this.$emit('excisform', this.formworker)
      this.$emit('update:pageSize', val)
    },
    handleCurrentChange1 (val) {
      this.formworker = this.excision(this.getArr, this.pageSize)[val - 1]
      this.$emit('excisform', this.formworker)
      this.$emit('update:currpage', val)
    },
    // 数组分割
    excision (arr, size) {
      let arrobj = []
      let index = 0
      let arrlen = arr.length / size
      for (let i = 0; i < arrlen; i++) {
        let arrteam = []
        for (let j = 0; j < size; j++) {
          arrteam[j] = arr[index++]
          if (index === arr.length) {
            break
          }
        }
        arrobj[i] = arrteam
      }
      return arrobj
    }
  }
}
</script>

<style>
</style>

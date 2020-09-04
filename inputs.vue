<template>
  <div class="getInput selectdemo">
    <el-input size="small" @click.native.stop="inputbtn" placeholder="请输入内容" v-model="fworkshows1" clearable>
      <template slot="prepend">组名</template>
    </el-input>
    <div v-if="selectbox" class="selectbuttom" @click.stop="selectbox = true">
      <div>
        <el-checkbox v-model="checkAll2" @change="classquan">全选</el-checkbox>
      </div>
      <div>
        <el-tree @check-change="checkChangesbtn" ref="tree" :data="fworkNums" :check-on-click-node="true" show-checkbox node-key="value" :props="defaultProps">
        </el-tree>
      </div>
    </div>
  </div>
</template>

<script>
/*
用例:
<Inputs :fworkshows.sync="fworkshows" localName="orderTaskshow" />
*/
export default {
  name: 'pagin',
  props: {
    // 输入框绑定的值
    fworkshows: {
      type: String,
      required: true,
      default: ''
    },
    // 本地存储的键
    localName: {
      type: String,
      required: true
    }
  },
  data () {
    return {
      fworkshows1: '',
      checkAll2: false,
      defaultProps: {
        label: 'label'
      },
      fworkNums: [],
      selectbox: false
    }
  },
  mounted () {
    let getfwork = localStorage.getItem(this.localName)
    if (getfwork) {
      this.fworkshows1 = getfwork
    }
    window.addEventListener('click', this.clickdian)
    this.getshow()
  },
  watch: {
    fworkshows1: function (val, oldVal) {
      this.$emit('update: fworkshows', val)
      localStorage.setItem(this.localName, val)
      if (val === '') {
        this.checkAll2 = false
      }
    }
  },
  methods: {
    async getshow () {
      let res1 = await this.$api.getMesVDepartment()
      this.fworkNums = res1.data.values.map(e => ({ value: e.fname, label: e.fname }))
    },
    clickdian (event) {
      this.selectbox = false
      event.stopPropagation()
    },
    inputbtn () {
      this.selectbox = !this.selectbox
      if (this.selectbox) {
        let setData = []
        if (this.fworkshows1 !== '') {
          setData = this.fworkshows1.split(',')
          let setArr = []
          setData.forEach(e => {
            setArr.push(
              {
                value: e,
                label: e
              }
            )
          })
          this.$nextTick(function () {
            this.$refs.tree.setCheckedNodes(setArr)
          })
        }
      }
    },
    // 组号筛选
    classquan (val) {
      if (val) {
        this.$refs.tree.setCheckedNodes(this.fworkNums)
      } else {
        this.$refs.tree.setCheckedNodes([])
      }
    },
    checkChangesbtn (data, checked, indeterminate) {
      let checkdata = this.$refs.tree.getCheckedNodes()
      if (checkdata.length === 0) {
        this.checkAll2 = false
      }
      let chenum = checkdata.length
      this.checkAll2 = chenum === this.fworkNums.length
      let dataArrs = []
      checkdata.forEach(e => {
        dataArrs.push(e.label)
      })
      if (dataArrs.length !== 0) {
        this.fworkshows1 = dataArrs.join(',')
      } else {
        this.fworkshows1 = ''
      }
    }
  }
}
</script>

<style lang="less">
.selectdemo {
  position: relative;
  .selectbuttom {
    position: absolute;
    top: 35px;
    right: 0;
    z-index: 999;
    min-width: 220px;
    border: 1px solid rgb(160, 160, 160);
    background-color: #fff;
    box-shadow: 2px 2px 5px #c7c7c7;
    border-radius: 3px;
    > div {
      &:nth-child(1) {
        width: 100%;
        padding-left: 3px;
        box-sizing: border-box;
        background-color: rgb(219, 219, 219);
      }
      &:nth-child(2) {
        width: 100%;
        max-height: 220px;
        overflow-y: auto;
      }
    }
  }
}
</style>

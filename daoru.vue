<template>
  <div>
    <div class="dao_dmeo">
      <div @click="daobtns" v-loading.fullscreen.lock="fullscreenLoading">
        <span class="el-icon-plus"></span>导入表格
      </div>
      <input id="file" type="file" accept=".xlsx, .xls" @change="onImportExcel" />
    </div>
    <!-- 导入模板 -->
    <vxe-modal v-model="kExteriorBox" show-footer resize title="导入模板" width="60%" :lock-view="false" :mask="false" remember>
      <div style="display:flex;">
        <el-table class="table" :data="addshebei" style="width: 100%;" border :height="500">
          <el-table-column v-for="(value, index) in daoNum" :key="index" :label="value.label">
            <template slot-scope="scope">
              <el-input placeholder="请输入内容" v-model="scope.row[value.name]" clearable></el-input>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <div slot="footer" class="dialog-footer footercenter">
        <el-button size="medium" round @click="kExteriorBox = false">取 消</el-button>
        <el-button size="medium" round type="primary" @click="addxleorder">确 定</el-button>
      </div>
    </vxe-modal>
  </div>
</template>

<script>
/*
导入Excel文件，并添加到程序中
<Daoru @daofu="importNum" @init="init" :addshebei="addshebei" apiurl="addPriceArgs" :daoNum="daoNum" />
 */

import XLSX from 'xlsx'
export default {
  name: 'daoru',
  props: {
    // 添加接口
    apiurl: {
      type: String,
      required: true
    },
    /*
    导入表头数据，添加对用的数据
    daoNum: [
        {
          label: '等级ID',
          name: 'classId'
        },
        {
          label: '订单等级',
          name: 'className'
        },
        {
          label: '颜色',
          name: 'color'
        }
      ],
    */
    daoNum: {
      type: Array,
      require: true
    },
    /* importNum (data) {
      this.addshebei = data.map((item, i) => {
        return {
          classId: item['等级ID'],
          className: item['订单等级'],
          color: item['颜色']
        }
      })
    }, */
    addshebei: {
      type: Array,
      require: true
    }
  },
  data () {
    return {
      fullscreenLoading: false,
      kExteriorBox: false
    }
  },
  methods: {
    daobtns () {
      this.fullscreenLoading = true
    },
    addxleorder () {
      const loading = this.$loading({
        lock: true,
        text: '正在导入中',
        spinner: 'el-icon-loading',
        background: 'rgba(0, 0, 0, 0.7)'
      })
      this.$api[this.apiurl](this.addshebei)
        .then(res => {
          if (res.data.result === true) {
            loading.close()
            this.$poin('success', `成功导入${res.data.success}条数据`)
          } else if (res.data.result === false) {
            this.$poin('success', '导入数据添加失败')
          }
          this.kExteriorBox = false
          this.$emit('init')
        })
        .catch(() => {
          this.$poin('error', '添加失败，网络或服务器异常')
        })
    },
    onImportExcel (file) {
      // 获取上传的文件对象
      const { files } = file.target
      // 通过FileReader对象读取文件
      const fileReader = new FileReader()
      fileReader.onload = event => {
        try {
          const { result } = event.target
          // 以二进制流方式读取得到整份excel表格对象
          const workbook = XLSX.read(result, { type: 'binary' })
          // 存储获取到的数据
          let data = []
          // 遍历每张工作表进行读取（这里默认只读取第一张表）
          for (const sheet in workbook.Sheets) {
            // esline-disable-next-line
            if (workbook.Sheets.hasOwnProperty(sheet)) {
              // 利用 sheet_to_json 方法将 excel 转成 json 数据
              data = data.concat(
                XLSX.utils.sheet_to_json(workbook.Sheets[sheet])
              )
              // break; // 如果只取第一张表，就取消注释这行
            }
          }
          // 最终获取到并且格式化后的 json 数据
          this.$emit('daofu', data)
          // this.addshebei = data.map((item, i) => {
          //   return this.importers
          // })
          this.fullscreenLoading = false
          this.$poin('success', '导入成功')

          this.kExteriorBox = true
          document.getElementById('file').value = ''
        } catch (e) {
          // 这里可以抛出文件类型错误不正确的相关提示
          this.$poin('error', '导入失败，网络或服务器错误')
        }
      }
      // 以二进制方式打开文件
      fileReader.readAsBinaryString(files[0])
    }
  }
}
</script>

<style>
.file {
  height: 40px;
}
</style>

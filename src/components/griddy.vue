<template>
  <div class="wrapper" :style="{width: width, height: height + 'px'}">
    <div class="grid-header">
      <span class="title">{{title}}</span>
    </div>
    <div class="grid-body" :style="{height: tableBodyHeight + 'px'}">
      <div class="table-header-wrapper">
        <table class="table-header">
          <tr>
            <th v-for="column in columns" :data-column-name= "column.name" :width="column.width ? column.width : '100'"> {{column.label}}</th>
          </tr>
        </table>
      </div>
      <div class="table-body-wrapper">
        <table class="table-body">
          <tr v-for="row in rows">
            <td v-for="column in columns" :width="column.width ? column.width : '100'"> {{val(row, column.field)}}</td>
          </tr>
        </table>
      </div>
    </div>
    <div class="grid-footer">
      <div v-if="!error">
        <span><a @click="handlePrevious">Previous</a></span>
        <span>{{page}}</span>
        <span><a @click="handleNext">Next</a></span>
      </div>
      <span v-else>{{error}}</span>
    </div>
  </div>
</template>

<script>
import _ from 'lodash'
export default {
  name: 'griddy',
  created () {
    this.resource = this.$resource(this.resourceURL)
    this.fetchData()
  },
  data () {
    return {
      grips: [],
      rows: [],
      error: null,
      page: 1,
      records: 10
    }
  },
  computed: {
    tableBodyHeight: function () {
      return this.height - 80
    }
  },
  props: {
    title: {type: String, default: 'Griddy'},
    columns: {type: Array, required: true},
    width: {type: String, default: '100%'},
    height: {type: String, default: '300'},
    resourceURL: {type: String, required: true}
  },
  methods: {
    val (object, attrib) {
      return _.get(object, attrib)
    },
    greaterThenMinWidth (colName, width) {
      const vm = this
      const column = vm.columns.find((c) => c.name === colName)
      const colWidth = parseInt(column.width) || 50 // Let us keep 50px as defult width

      if (!column) { return false }
      console.log(width, colWidth)

      if (width > colWidth) {
        return true
      }

      return false
    },
    onMouseDown (e) {
      const vm = this
      vm.thElm = e.target.parentNode
      vm.startOffset = vm.thElm.offsetWidth - e.pageX
    },
    onMouseMove (e) {
      const vm = this
      if (vm.thElm) {
        const colName = vm.thElm.getAttribute('data-column-name')
        const width = vm.startOffset + e.pageX

        if (vm.greaterThenMinWidth(colName, width)) {
          vm.thElm.width = width + 'px'
          vm.syncColumnWidths()
        }
      }
    },
    onMouseUp (e) {
      const vm = this
      vm.thElm = undefined
      vm.syncColumnWidths()
    },
    fetchData () {
      this.resource.get({results: this.records, page: this.page}).then((response) => {
        this.rows = response.body.results
      }).catch(() => {
        this.error = 'Error occured while trying to fetch data'
      })
    },
    setResizeGrips () {
      const vm = this
      const headerCols = Array.from(vm.header.getElementsByTagName('th'))
      headerCols.forEach((th) => {
        th.style.position = 'relative'

        const grip = document.createElement('div')
        grip.className = 'grip'
        grip.innerHTML = '&nbsp'
        grip.style.top = 0
        grip.style.right = 0
        grip.style.bottom = 0
        grip.style.width = '5px'
        grip.style.position = 'absolute'
        grip.style.cursor = 'col-resize'
        grip.addEventListener('mousedown', this.onMouseDown)
        th.appendChild(grip)
        vm.grips.push(grip)
      })

      document.addEventListener('mousemove', this.onMouseMove)
      document.addEventListener('mouseup', this.onMouseUp)
    },
    syncColumnWidths () {
      const vm = this
      const headerCols = Array.from(vm.header.getElementsByTagName('th'))
      const widths = headerCols.map((h) => h.width ? h.width : h.clientWidth)
      const bodyCols = Array.from(vm.body.querySelectorAll('tr:first-child>td'))
      bodyCols.forEach((c, i) => {
        c.width = widths[i] + 'px'
      })
    },
    handlePrevious () {
      if (this.page > 1) {
        this.page -= 1
        this.fetchData()
      }
    },
    handleNext () {
      this.page += 1
      this.fetchData()
    }
  },
  mounted () {
    const vm = this
    vm.header = vm.$el.getElementsByClassName('table-header')[0]
    vm.body = vm.$el.getElementsByClassName('table-body')[0]
    vm.setResizeGrips()
    vm.syncColumnWidths()
  },
  beforeDestroy () {
    const vm = this
    vm.grips.forEach((grip) => grip.removeEventListener('mousedown', vm.onMouseDown))
    document.removeEventListener('mousemove', vm.onMouseMove)
    document.removeEventListener('mouseup', vm.onMouseUp)
  }
}
</script>

<style scoped>

  .wrapper {
    overflow-y: scroll;
    background-color: #aaa;
  }

  .grid-header, .grid-footer {
    display: flex;
    flex-flow: row wrap;
    min-height: 40px;
    background-color: #efefef;
    line-height: 40px;
    padding: 0 5px 0 5px;
  }

  .grid-header > .title {
    font-weight: bold;
    text-transform: uppercase;
  }

  .grid-footer > div {
    display: flex;
    flex-flow: row wrap;
    align-items: center;
    width: 100%;
    justify-content: space-between;
    font-size: 0.8em;
    font-weight: normal;
    text-transform: uppercase;
  }

  .grid-body {
    min-height: 100px;
    border: #efefef 1px solid;
    overflow-x: scroll;
  }

  .table-body-wrapper {
    overflow-y: scroll;
  }

  table {
    border-collapse: collapse;
    border-sizing: border-box;
    background-color: #fff;
  }

  table, td, th {
    border: 1px solid #efefef;
    padding: 2px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
</style>

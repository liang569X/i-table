<template>
  <div class="ITable">
    <el-table
        ref="ITable"
        v-loading="loading"
        :data="data"
        v-bind="$attrs"
        v-on="$listeners"
    >
      <el-table-column
          v-for="(head, index) in header"
          :key="head.id || index"
          v-bind="$attrs"
          :prop="head.prop"
          :label="head.label"
          :type="head.type"
          :column-key="head.columnKey"
          :width="head.width"
          :min-width="head.minWidth"
          :fixed="head.fixed"
          :render-header="head.renderHeader"
          :sortable="head.sortable || false"
          :sort-method="head.sortMethod"
          :sort-by="head.sortBy"
          :sort-orders="head.sortOrders"
          :resizable="head.resizable || true"
          :formatter="head.formatter"
          :show-overflow-tooltip="head.showOverflowTooltip || true"
          :align="head.align || 'center'"
          :header-align="head.headerAlign || head.align || 'center'"
          :class-name="head.className || ''"
          :label-class-name="head.labelClassName || ''"
          :selectable="head.selectable || null"
          :reserve-selection="head.reserveSelection || false"
          :filters="head.filters || null"
          :filter-placement="head.filterPlacement"
          :filter-multiple="head.filterMultiple"
          :filter-method="head.filterMethod"
          :filtered-value="head.filteredValue"
          v-on="$listeners"
      >
        <template v-if="head.render" #default="{ row }">
          <slot
              v-if="Object.prototype.toString.call(head.render) === '[object String]'"
              :name="head.render"
              :row="row"
          />
          <Jsx
              v-else
              :row="row"
              :Jsx="head.render"
          />
        </template>
        <!--        <template-->
        <!--          v-else-if="head.type !== 'index' && head.type !== 'selection' && !head.formatter && head.prop.split('.').length === 1"-->
        <!--          #default="{ row }"-->
        <!--         >-->
        <!--          {{ handleDefaultString(row[head.prop]) }}-->
        <!--        </template>-->
      </el-table-column>
    </el-table>
    <div v-if="pagination" :style="[{ marginTop: '10px', textAlign: 'right' }]">
      <el-pagination
          :current-page.sync="query.pageNum"
          :page-sizes="[10, 20, 30]"
          :page-size.sync="query.pageSize"
          :layout="'total, sizes, prev, pager, next, jumper'"
          :total="total"
      />
    </div>
  </div>
</template>

<script>
/**
 * Author: LHY
 * Date: 2020-11-01
 * Tips-> 在Vue-> 2.6.10 elementUI-> 2.13.2 lodash-> 4.17.20 axios-> 0.18.1 test
 * 获取table的数据 函数名: get-table-data<Function> , 查询参数: query<any> (把需要查询的参数放进query对象里面)
 * Tips-> 只有查询table数据的函数才传过来， 其他的查询在父组件使用
 */
import { debounce } from 'lodash'
import Jsx from './Jsx'

export default {
  name: 'ITable',
  components: { Jsx },
  props: {
    /**
     * 是否显示pagination -> e.g. :pagination='Boolean'
     */
    pagination: {
      type: Boolean,
      default: true
    },
    /**
     * 分页的总数 -> e.g. :total='Number'
     */
    total: {
      required: true,
      type: Number,
      default: 0
    },
    /**
     *  控制table的loading -> e.g. :loading='Boolean'
     */
    loading: {
      type: Boolean,
      default: false
    },
    /**
     *  是否debounce 需要防抖的字段 e.g. :debounce='[...String]'
     */
    debounce: {
      type: Array,
      default: () => []
    },
    /**
     *  表头及label对应的prop e.g. :header='[...Object]'
     */
    header: {
      required: true,
      type: Array,
      default: () => []
    },
    /**
     *  table的数据 e.g. :data='[...Object]'
     */
    data: {
      required: true,
      type: Array,
      default: () => []
    },
    /**
     *  获取table的数据的函数 e.g. :get-table-data='Function'
     */
    getTableData: {
      type: Function,
      required: true,
      default: () => () => {}
    },
    /**
     *  查询参数对象 e.g. :query='{...any}'
     */
    query: {
      required: true,
      type: Object,
      default: () => {}
    }
  },
  /**
   *  computed 观察value和oldValue的异同
   */
  computed: {
    computedQuery() {
      return { ...this.query }
    }
  },
  created() {
    /**
     *  比对需要debounce的字段和不需要debounce的字段
     */
    if (this.pagination) {
      this.$watch('computedQuery', function(value, oldValue) {
        if (!this.debounce.length || !oldValue || !value) { this.getTableData() } else {
          const OPH = Object.prototype.hasOwnProperty
          for (const keys in value) {
            if (OPH.call(value, keys)) {
              for (const key in oldValue) {
                if (OPH.call(oldValue, key)) {
                  keys === key
                      ? oldValue[key] !== value[keys]
                      ? (typeof value[keys] === 'string' && typeof oldValue[key] === 'string')
                          ? oldValue[key].length > value[keys].length
                              ? this.getTableData()
                              : this.debounce.some(item => key === item)
                                  ? this.debounceGetTableData()
                                  : this.getTableData()
                          : this.getTableData()
                      : void 0
                      : void 0
                }
              }
            }
          }
        }
      }, { deep: true, immediate: true })
    } else {
      this.getTableData()
    }
  },
  methods: {
    /**
     * 全局设定 如果prop对应的值为空 自定义显示  e.g. -> '-'
     */
    // handleDefaultString(val) {
    //   if (val === 0 || val === false || val) {
    //     return val
    //   } else {
    //     return '-'
    //   }
    // },
    /**
     * debounce
     */
    debounceGetTableData: debounce(function() {
      this.getTableData()
    }, 1000)
  }
}
</script>

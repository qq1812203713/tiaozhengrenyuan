
<template>
  <div class="custom-tree-node">
    <a-input-search
      style="margin-bottom: 8px; width: 100%"
      allowClear
      placeholder="搜索机构/用户"
      @change="filterNode"
      :maxLength="100"
      v-model="searchValue"
    />
    <div class="custom-tree-box" :style="{ maxHeight: maxHeight }">
      <a-tree
        v-if="newDeptOptions.length > 0"
        ref="tree"
        :checkbox-prop="'checked'"
        checkable
        v-model="num"
        :defaultSelectedKeys="[openDeptId]"
        :treeData="newDeptOptions"
        :replaceFields="replaceFields"
        :show-icon="showIcon"
        :showLine="showLine"
        defaultExpandAll
        @expand="onExpand"
        @check="onCheck"
      >
        <img src="@/assets/images/zuzhiLogo.png" slot="nodeicon" />
        <img src="@/assets/images/userLogo.png" slot="user" />
        <template slot="custom" slot-scope="text">
          <span :title="text.label">{{ text.label }}</span>
          <slot name="custom" :text="text"></slot>
        </template>
      </a-tree>
    </div>
  </div>
</template>
<script>
import cloneDeep from 'lodash'

export default {
  name: 'DeptTree',
  props: {
    deptOptions: {
      type: Array,
      required: true
    },
    defaultExpandAll: {
      type: Boolean,
      default: false
    },
    replaceFields: {
      type: Object,
      default: () => {
        return { children: 'children', title: 'label', key: 'id', value: 'id' }
      }
    },
    maxHeight: {
      type: String,
      default: () => '880px'
    },
    showAll: {
      // 是否显示‘全部’节点
      type: Boolean,
      default: true
    },
    openDeptId: {
      type: String,
      default: ''
    },
    type: {
      type: String,
      default: ''
    },
    value: {
      type: String,
      default: ''
    },
    dataListAll: {
      type: Array,
      // eslint-disable-next-line vue/require-valid-default-prop
      default: []
    },
    numcheckBox: {
      type: Array,
      // eslint-disable-next-line vue/require-valid-default-prop
      default: []
    }
  },
  components: {},
  data () {
    return {
      showLine: false,
      showIcon: true,
      deptNodes: [],
      expandedKeys: [],
      autoExpandParent: true,
      newDeptOptions: [],
      id: '',
      searchValue: '',
      num: [],
      formerDeptNodes: [],
      dataListNum: [],
      NewcommonArr: []
    }
  },
  filters: {},
  computed: {},
  watch: {
    deptOptions: {
      immediate: true,
      handler (val) {
        const newVal = this.showAll
          ? this.formatAllData(cloneDeep(val))
          : this.formatAllData(cloneDeep(val[0].children))
        newVal
          ? this.newDeptOptions.splice(0, this.newDeptOptions.length, ...cloneDeep(newVal))
          : this.newDeptOptions.splice(0, this.newDeptOptions.length, ...[])
      },
      deep: true
    },
    value: {
      handler (val) {
        this.searchValue = val
      }
    }
  },
  created () {
    this.resetForm()
    if (this.numcheckBox.length) {
      this.num = this.numcheckBox
    }
  },
  methods: {
    onCheck (e) {
      // eslint-disable-next-line no-empty
      if (this.searchValue) {
        this.$emit('searchVal', this.formerDeptNodes, this.dataListNum, this.num, this.NewcommonArr)
      } else {
        this.$emit('emitVal', this.num)
      }
    },
    cls () {
      this.num = []
    },
    resetForm () {
      this.searchValue = ''
      this.$emit('input', '')
    },
    // 查找
    getParentKey (id, tree) {
      let parentKey
      for (let i = 0; i < tree.length; i++) {
        const node = tree[i]
        if (node.children) {
          if (node.children.some((item) => item.id === id)) {
            parentKey = node.id
          } else if (this.getParentKey(id, node.children)) {
            parentKey = this.getParentKey(id, node.children)
          }
        } else if (node.id && node.id === id) {
          parentKey = node.id
        }
      }
      return parentKey
    },
    // 获取全部节点
    getAllDeptNode (nodes) {
      if (!nodes || nodes.length === 0) {
        return []
      }
      nodes.forEach((node) => {
        this.deptNodes.push({
          id: node.id,
          label: node[this.replaceFields.title],
          children: node.children ? node.children : null,
          certIdNum: node.certIdNum ? node.certIdNum : null,
          slots: { icon: 'nodeicon' }
        })
        return this.getAllDeptNode(node.children)
      })
    },
    // 筛选节点
    filterNode (e) {
      this.deptNodes = []
      this.getAllDeptNode(this.deptOptions)
      const value = e.target.value
      if (!value) {
        Object.assign(this, {
          searchValue: value,
          autoExpandParent: true
        })
        this.expandedKeys.splice(0, this.expandedKeys.length)
        this.newDeptOptions.splice(0, this.newDeptOptions.length, ...cloneDeep(this.deptOptions))
        this.formerDeptNodes = []
        return
      }
      const gData = this.deptOptions
      const newDeptOptions = []
      const expandedKeys = this.deptNodes
        .map((item) => {
          if (item.label.indexOf(value) > -1) {
            if (item.children) {
              return this.getParentKey(item.id, gData)
            } else {
              if (item.certIdNum) {
                item.slots = { icon: 'user' }
                newDeptOptions.push(item)
              } else {
                item.slots = { icon: 'nodeicon' }
              }
            }
            return this.getParentKey(item.id, gData)
          }
          return null
        })
        .filter((item, i, self) => item && self.indexOf(item) === i)
      Object.assign(this, {
        expandedKeys: expandedKeys,
        searchValue: value,
        autoExpandParent: true,
        newDeptOptions
      })
      this.dataListNum = this.dataListAll
      this.formerDeptNodes = newDeptOptions
      let selectedR = []
      let selectedL = []
      this.dataListNum.forEach((item) => {
        selectedR.push(item.id)
      })
      this.formerDeptNodes.forEach((item) => {
        selectedL.push(item.id)
      })
      const commonArr = selectedR.filter((item) => selectedL.includes(item))
      this.NewcommonArr = commonArr
    },
    // 节点单击事件
    // handleNodeClick (keys, event) {
    //     this.selectedKeys = selectedKeys;
    //     this.$emit('select', keys[0], event.node._props.dataRef)
    // },
    // 展开子项
    onExpand (expandedKeys) {
      this.expandedKeys = expandedKeys
      this.autoExpandParent = false
    },
    // 给所有数据增加icon
    formatAllData (data) {
      data.forEach((item) => {
        if (item.children) {
          item.slots = { icon: 'nodeicon' }
        } else {
          if (item.certIdNum) {
            item.slots = { icon: 'user' }
          } else {
            item.slots = { icon: 'nodeicon' }
          }
        }
        if (item.children && item.children.length) {
          this.formatAllData(item.children)
        }
      })
      return data
    }
  }
}
</script>
<style scoped lang="less">
::v-deep {
  .ant-input {
    border-radius: 5px;
  }
}
.custom-tree-node {
  min-width: 200px;
  & ::v-deep .ant-tree li .ant-tree-node-content-wrapper {
    width: 83%;
    position: relative;
  }
  .custom-tree-box {
    overflow-y: scroll;
  }
}
::v-deep .operation {
  display: none;
}
::v-deep .ant-tree-node-content-wrapper:hover {
  .operation {
    display: block;
    position: absolute;
    right: 0px;
    top: 0px;
    width: 60px;
    .anticon {
      margin-left: 3px;
    }
  }
}
::v-deep .ant-tree-title span:first-child {
  width: 65%;
  display: inline-block;
}

::v-deep .ant-input-search {
  width: 95% !important;
}
::v-deep .el-tree-node__checkbox {
  position: absolute;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
}
::v-deep .ant-tree-iconEle {
  margin-right: 5px !important;
}
</style>

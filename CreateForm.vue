<template>
  <div>
    <a-modal
      :visible="visible"
      title="调整人员"
      width="55%"
      :confirm-loading="confirmLoading"
      @ok="handleOk"
      @cancel="handleCancel"
    >
      <div class="content-box">
        <div class="tree">
          <div class="tree_left">
            <wang-tree
              ref="treeData"
              searchPlaceholder="行政区划名称/号码"
              :deptOptions="treeData"
              @select="clickDeptNode"
              @emitVal="emitData"
              :dataListAll="dataList"
              @searchVal="searchData"
              :numcheckBox="numcheckBox"
            ></wang-tree>
          </div>
          <div class="shu"></div>
          <div class="tree_right">
            <div class="num">
              <p>已添加人员：{{ dataList?.length }} 人</p>
              <p @click="clsNum">全部清除</p>
            </div>
            <div class="listName">
              <div class="list_num" v-for="item in dataList" :key="item.id">
                <span>{{ item.label }}</span>
                <span @click="closeId(item.id)"><a-icon type="close" /></span>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div slot="footer">
        <a-button @click="handleCancel">取消</a-button>
        <a-button type="primary" @click="handleOk" v-if="!id">提交</a-button>
      </div>
    </a-modal>
  </div>
</template>

<script>
import WangTree from './treeForm.vue'
import { treeselect } from '@/api/system/dept'
import { treeselectuserAdd, vopackageAdduser, getQueryList } from '@/api/trainingManagement/curriculum'
export default {
  components: { WangTree },
  data () {
    return {
      treeData: [],
      selectedNodes: [],
      dataList: [],
      nums: [],
      numcheckBox: []
    }
  },
  props: {
    visible: {
      type: Boolean,
      default: false
    },
    id: {
      type: Number || String,
      default: ''
    },
    listUser: {
      type: Array,
      // eslint-disable-next-line vue/require-valid-default-prop
      default: []
    }
  },
  watch: {
    dataList: {
      handler (wld, old) {
        if (wld?.length > old?.length) {
          let intervalId = setInterval(() => {
            const dom = document.querySelectorAll('.list_num')
            if (wld.length === dom.length) {
              clearInterval(intervalId) // 停止检查
              // 在此处执行您要执行的操作
              this.bottomSr(dom[dom.length - 1])
            }
          }, 1000)
          // eslint-disable-next-line no-empty
        } else {
        }
      },
      immediate: true,
      deep: true
    }
  },
  created () {
    if (this.listUser?.length) {
      let datas = []
      this.listUser.forEach(item => {
        datas.push({ id: item.id, label: item.deptName, areaCode: item.areaCode, certIdNum: item.certidnum, packageName: item.packageName, deptId: item.parentId, strategyType: item.strategyType })
        this.numcheckBox.push(item.id)
      })
      this.dataList = datas
    }
    let UserID = this.$store.state.user.user.deptId
    console.log(UserID)
    treeselectuserAdd().then((res) => {
      this.treeData = res.data
    })
    // treeselectuserAdd().then((res) => {
    //   this.treeData = res.data
    //  function formatAllData (data) {
    //   data.forEach((item) => {
    //     if (item.id === UserID) {
    //       return
    //     }
    //     item.disabled = true
    //     if (item.children && item.children.length) {
    //       formatAllData(item.children)
    //     }
    //   })
    //   return data
    // }
    //   formatAllData(this.treeData)
    // })
    // function dig (path = '0', level = 3) {
    //   const list = []
    //   for (let i = 0; i < 5; i += 1) {
    //     const key = `${path}-${i}`
    //     const treeNode = {
    //       label: key,
    //       id: key
    //     }
    //     if (level > 0) {
    //       treeNode.children = dig(key, level - 1)
    //     }
    //     list.push(treeNode)
    //   }
    //   return list
    // }
    // this.treeData = dig()
    // this.getTreeselect()
  },
  methods: {
    closeId (id) {
      this.nums = []
      this.dataList.forEach((item, index) => {
        if (item.id === id) {
          this.dataList.splice(index, 1)
        }
      })
      this.dataList.forEach((item) => {
        this.nums.push(item.id)
      })
      this.$refs.treeData.num = this.nums
    },
    clsNum () {
      this.dataList = []
      this.nums = []
      this.$refs.treeData.cls()
    },
    emitData (num) {
      this.dataList = []
      function findNodeById (nodes, id) {
        for (let node of nodes) {
          if (node?.id === id) {
            return node
          } else if (node.children) {
            let found = findNodeById(node.children, id)
            if (found) {
              return found
            }
          }
        }
        return null
      }
      num.forEach((item) => {
        let treeUser = findNodeById(this.treeData, item)
        if (treeUser.certIdNum) {
          this.dataList.push(treeUser)
        } else {
        }
      })
      this.dataList.forEach((item, index) => {
        if (!item) {
          this.dataList.splice(index, 1)
        }
        if (item?.children) {
          this.dataList.splice(index, 1)
        }
      })
      this.dataList.forEach((item, index) => {
        if (item.children) {
          this.dataList.splice(index, 1)
        }
      })
    },
    searchData (val, num, cheNum, commonArr) {
      let dataListA = this.numId(val)
      let dataList = this.numId(num)
      let datalistChe = []
      function findNodeById (nodes, id) {
        for (let node of nodes) {
          if (node?.id === id) {
            return node
          } else if (node.children) {
            let found = findNodeById(node.children, id)
            if (found) {
              return found
            }
          }
        }
        return null
      }
      cheNum.forEach((item) => {
        datalistChe.push(findNodeById(this.treeData, item))
      })
      let numl = []
      let numr = []
      let numc = []
      datalistChe.forEach((item) => {
        numc.push(item.id)
      })
      dataListA.forEach((item) => {
        numl.push(item.id)
      })
      dataList.forEach((item) => {
        numr.push(item.id)
      })
      if (commonArr.length > numc.length) {
        commonArr.length--
        console.log('--')
        let List = []
        this.dataList.forEach((item) => {
          List.push(item.id)
        })
        const filteredArr1 = List.filter((element) => numl.includes(element))
        const removedArr = filteredArr1.filter((item) => !numc.includes(item))
        List.forEach((item, index) => {
          if (removedArr[0] === item) {
            List.splice(index, 1)
            this.$refs.treeData.num = List
            this.emitData(List)
          }
        })
      } else {
        console.log('++')
        let List = []
        this.dataList.forEach((item) => {
          List.push(item.id)
        })
        commonArr.length++
        const addedArr = numc.filter((item) => !List.includes(item))
        List.push(addedArr[0])
        this.$refs.treeData.num = List
        this.emitData(List)
      }
    },
    // 过滤
    numId (num) {
      let dataList = []
      function findNodeById (nodes, id) {
        for (let node of nodes) {
          if (node.id === id) {
            return node
          } else if (node.children) {
            let found = findNodeById(node.children, id)
            if (found) {
              return found
            }
          }
        }
        return null
      }
      num.forEach((item) => {
        dataList.push(findNodeById(this.treeData, item.id))
      })
      dataList.forEach((item, index) => {
        if (!item) {
          dataList.splice(index, 1)
        }
        if (item?.children) {
          dataList.splice(index, 1)
        }
      })
      dataList.forEach((item, index) => {
        if (item?.children) {
          dataList.splice(index, 1)
        }
      })
      return dataList
    },
    onCheck (checkedids) {
      this.selectedNodes = checkedids.map((id) => {
        return this.getNodeByid(this.treeData, id)
      })
    },
    // 滚轮跑到最底层
    bottomSr (targetNode) {
      targetNode.scrollIntoView({ behavior: 'smooth', block: 'nearest', inline: 'center' })
    },
    removeNode (index) {
      this.selectedNodes.splice(index, 1)
    },
    removeAll () {
      this.selectedNodes = []
    },
    clickDeptNode (deptId) {
    },
    handleOk () {
      this.confirmLoading = true
      let ids = []
      this.dataList.forEach(item => {
        ids.push(item.id)
        item.deptName = item.label
      })
      vopackageAdduser(this.dataList).then((res) => {
        getQueryList(ids).then(res => {
          this.$parent.$parent.dataSource = res.data.data
          this.confirmLoading = false
      this.$emit('update:visible', false)
        })
      }).catch(() => {
        this.confirmLoading = false
      })
    },
    handleCancel () {
      this.$emit('update:visible', false)
    },
    /** 查询机构下拉树结构 */
    getTreeselect () {
      treeselect().then((response) => {
        this.treeData = response.data
      })
    }
  }
}
</script>

<style lang="less" scoped>
.tree {
  display: flex;
  margin: 0 auto;
  height: 457px;
  background: #ffffff;
  border: 1px solid #e5e6eb;
  .tree_left {
    width: 456px;
    padding-top: 16px;
    padding-left: 16px;
  }
  .shu {
    width: 1px;
    height: 100%;
    background-color: #e5e6eb;
  }
  .tree_right {
    width: 455px;
    padding-top: 23px;
    padding-left: 23px;
    padding-right: 23px;
  }
  .num {
    display: flex;
    justify-content: space-between;
    p {
      &:nth-child(1) {
        font-size: 14px;
        font-family: Microsoft YaHei;
        font-weight: 400;
        color: #333333;
      }
      &:nth-child(2) {
        cursor: pointer;
        font-size: 14px;
        font-family: Microsoft YaHei;
        font-weight: 400;
        color: #008fe0;
      }
    }
  }
  .listName {
    height: 380px;
    overflow-y: auto;
    .list_num {
      margin-top: 22px;
      display: flex;
      justify-content: space-between;
      span {
        &:nth-child(1) {
          margin-left: 5px;
        }
        &:nth-child(2) {
          cursor: pointer;
          margin-right: 10px;
        }
      }
    }
  }
}
/deep/ .custom-tree-box {
  height: 386px !important;
}
</style>

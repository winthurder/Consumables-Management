<template>

  <el-card class="form-container" shadow="never">

    <el-tree
      :data="menuTreeList"
      show-checkbox
      default-expand-all
      node-key="id"
      ref="tree"
      highlight-current
      :props="defaultProps">
    </el-tree>
    <div style="margin-top: 20px" align="center">
      <el-button @click="handleClear()">清空</el-button>
      <el-button type="primary" @click="handleSave()">保存</el-button>
      <el-button @click="back()">返回</el-button>
    </div>

  </el-card>
</template>

<script>
import {fetchTreeList} from '@/api/menu';
import {allocMenu, listMenuByRole} from '@/api/role';
import {getCookie} from "../../../utils/support";

const defaultListQuery = {
  username: getCookie("username")
};
export default {
  name: "allocMenu",
  data() {
    return {
      menuTreeList: [],
      defaultProps: {
        children: 'children',
        label: 'title'
      },
      roleId: null,
      listQuery: Object.assign({}, defaultListQuery),
    };
  },
  created() {
    this.roleId = this.$route.query.roleId;
    this.treeList();
    this.getRoleMenu(this.roleId);
  },
  methods: {
    back() {
      if (window.history.length <= 1) {
        this.$router.push({path: '/'})
        return false
      } else {
        this.$router.go(-1)
      }
    },
    treeList() {
      fetchTreeList(this.listQuery).then(response => {
        this.menuTreeList = response.data;
      });
    },
    getRoleMenu(roleId) {
      listMenuByRole(roleId).then(response => {
        let menuList = response.data;
        let checkedMenuIds = [];
        if (menuList != null && menuList.length > 0) {
          for (let i = 0; i < menuList.length; i++) {
            let menu = menuList[i];
            if (menu.parentId !== 0) {
              checkedMenuIds.push(menu.id);
            }
          }
        }
        this.$refs.tree.setCheckedKeys(checkedMenuIds);
      });
    },
    handleSave() {
      let checkedNodes = this.$refs.tree.getCheckedNodes();
      let checkedMenuIds = new Set();
      if (checkedNodes != null && checkedNodes.length > 0) {
        for (let i = 0; i < checkedNodes.length; i++) {
          let checkedNode = checkedNodes[i];
          checkedMenuIds.add(checkedNode.id);
          if (checkedNode.parentId !== 0) {
            checkedMenuIds.add(checkedNode.parentId);
          }
        }
      }
      this.$confirm('是否分配菜单？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        let params = new URLSearchParams();
        params.append("roleId", this.roleId);
        params.append("menuIds", Array.from(checkedMenuIds));
        allocMenu(params).then(response => {
          this.$message({
            message: '分配成功',
            type: 'success',
            duration: 1000
          });
          this.$router.back();
        })
      })
    },
    handleClear() {
      this.$refs.tree.setCheckedKeys([]);
    }
  }
}
</script>

<style scoped>

</style>

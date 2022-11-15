<template>
  <div>
      <el-button type="danger" @click="batchDelete">批量删除</el-button>
    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKey"
      ref="menuTree"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            添加
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            删除
          </el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">
            编辑
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      width="30%"
      :close-on-click-modal="false"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      menus: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
      expandedKey: [],
      dialogVisible: false,
      dialogType: "", //edit,add
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        icon: "",
        productUnit: "",
      },
      title: "",
    };
  },
  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        console.log("成功获取菜单数据", data.data);
        this.menus = data.data;
      });
    },
    batchDelete() {
        let checkedNodes = this.$refs.menuTree.getCheckedNodes(false,false);
        let catIds = [];
        console.log(checkedNodes);
        for(let i = 0; i < checkedNodes.length; i++) {
            catIds.push(checkedNodes[i].catId);
        }
        //发送请求
        this.$confirm(`是否批量删除【${catIds}】菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(catIds, false),
          }).then(({ data }) => {
            console.log("批量删除成功...");
            this.$message({
              message: "批量删除成功",
              type: "success",
            });
            this.getMenus();
          });
        })
        .catch(() => {});
    },
    append(data) {
      this.dialogType = "add";
      this.title = "添加分类";
      console.log("添加", data);
      this.dialogVisible = true;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      this.category.catId = null;
      this.category.productUnit = "";
      this.category.sort = 0;
      this.category.showStatus = 1;
      this.category.icon= "";
   

    },
    edit(data) {
      this.dialogType = "edit";
      this.title = "修改分类";
      console.log("要修改的数据", data);
      this.dialogVisible = true;
      //发送请求获取当前节点最新的数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        //请求成功
        console.log("data", data);
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.icon = data.data.icon;
        this.productUnit = data.data.productUnit;
        this.parentCid = data.data.parentCid;
      });
    },
    submitData() {
      if (this.dialogType == "add") {
        this.addCategory();
      }
      if (this.dialogType == "edit") {
        this.editCategory();
      }
    },
    //修改三级分类
    editCategory() {
      var { catId, name, icon, productUnit } = this.category;
      var data = {
        catId: catId,
        name: name,
        icon: icon,
        productUnit: productUnit,
      };
      console.log("提交三级分类的数据", this.category);
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData(data, false),
      }).then(({ data }) => {
        console.log("菜单修改成功...");
        this.$message({
          message: "菜单修改成功",
          type: "success",
        });
        //关闭对话框
        this.dialogVisible = false;
        //刷新数据，显示父节点
        this.getMenus();
        this.expandedKey = [this.category.parentCid];
      });
    },
    //添加三级分类
    addCategory() {
      console.log("提交三级分类的数据", this.category);
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        console.log("菜单添加成功...");
        this.$message({
          message: "菜单添加成功",
          type: "success",
        });
        //关闭对话框
        this.dialogVisible = false;
        //刷新数据，显示父节点
        this.getMenus();
        this.expandedKey = [this.category.parentCid];
      });
    },
    remove(node, data) {
      var ids = [data.catId];
      this.$confirm(`是否删除【${data.name}】菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            console.log("删除成功...");
            this.$message({
              message: "删除成功",
              type: "success",
            });
            this.getMenus();
            this.expandedKey = [node.parent.data.catId];
          });
        })
        .catch(() => {});
    },
  },
  // 生命周期 -创建完成（可以访问当前的this实例）
  created() {
    this.getMenus();
  },
};
</script>

<style>
</style>
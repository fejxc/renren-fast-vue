<template>
  <el-tree :data="menus" :props="defaultProps" node-key="catId" ref="menuTree" @node-click="nodeclick">
  </el-tree>
</template>

<script>
export default {
  name: "Category",

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
      title: "",
    };
  },

  mounted() {},

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
    nodeclick(data,node,component) {
        console.log("子主件category被点击",data,node,component);
        //向父组件发送事件
        this.$emit("tree-node-click",data,node,component);
    }
  },

  // 生命周期 -创建完成（可以访问当前的this实例）
  created() {
    this.getMenus();
  },
};
</script>

<style>
</style>
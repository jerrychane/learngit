<template>
  <div class="fly-panel" style="margin-bottom: 0;">
    <div class="fly-panel-title fly-filter">
      <a :class="{'layui-this':status ==='' && tag ===''}" @click.prevent="search()">综合</a>
      <span class="fly-mid"></span>
      <a :class="{'layui-this':status ==='0'}" @click.prevent="search(0)">未结</a>
      <span class="fly-mid"></span>
      <a :class="{'layui-this':status === '1'}" @click.prevent="search(1)">已结</a>
      <span class="fly-mid"></span>
      <a :class="{'layui-this':status === '' && tag === '精华'}" @click.prevent="search(2)">精华</a>
      <span class="fly-filter-right layui-hide-xs">
        <a :class="{'layui-this':sort === 'created'}" @click.prevent="search(3)">按最新</a>
        <span class="fly-mid"></span>
        <a :class="{'layui-this':sort === 'answer'}" @click.prevent="search(4)">按热议</a>
      </span>
    </div>
    <list-item :lists="lists" :isEnd="isEnd" @nextPage="nextPage()"></list-item>
  </div>
</template>

<script>
import { getList } from "@/api/content";
import ListItem from "./ListItem";
export default {
  name: "list",
  components: {
    ListItem
  },
  data() {
    return {
      isEnd: false,
      status: "",
      tag: "",
      sort: "created",
      page: 0,
      limit: 20,
      catalog: "",
      lists: []
    };
  },
  mounted() {
    this._getLists();
  },
  methods: {
    _getLists() {
      if (this.isEnd) {
        return;
      }
      let options = {
        catalog: this.catalog,
        isTop: 0,
        page: this.page,
        limit: this.limit,
        sort: this.sort,
        tag: this.tag,
        status: this.status
      };
      getList(options)
        .then(res => {
          console.log(res);
          // 对于异常的判断， res.code 非200，我们给用户一个提示
          // 判断是否lists长度为0，如果为零即可直接赋值
          // 当lists长度不为0，后面请求的数据，加入到Lists里面来
          if (res.code === 200) {
            console.log("_getLists -> res.data.length", res.data.length);
            console.log("_getLists -> this.limit", this.limit);
            if (res.data.length < this.limit) {
              this.isEnd = true;
              console.log("_getLists -> this.isEnd", this.isEnd);
            }
            // 判断res.data的长度，如果小于20条，则是最后一页
            if (this.lists.length === 0) {
              this.lists = res.data;
            } else {
              this.lists = this.lists.concat(res.data);
            }
          }
        })
        .catch(err => {
          this.$alert(err.msg);
        });
    },
    nextPage() {
      this.page++;
      this._getLists();
    },
    search(val) {
      switch (val) {
        // 未结帖
        case 0:
          this.status = "0";
          this.tag = "";
          break;
        // 已结帖
        case 1:
          this.status = "1";
          this.tag = "";
          break;
        // 查询'精华'标签
        case 2:
          this.status = "";
          this.tag = "精华";
          break;
        // 按最新，按照日期查询
        case 3:
          this.sort = "created";
          break;
        // 按热议，按照评论数查询
        case 4:
          this.sort = "answer";
          break;
        // 默认综合查询
        default:
          this.status = "";
          this.tag = "";
          break;
          break;
      }
    }
  }
};
</script>

<style lang="scss" scoped>
</style>
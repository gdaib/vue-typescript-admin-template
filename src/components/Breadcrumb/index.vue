<template>
  <el-breadcrumb
    class="app-breadcrumb"
    separator="/"
  >
    <transition-group name="breadcrumb">
      <el-breadcrumb-item
        v-for="(item,index) in levelList"
        :key="item.path"
      >
        <span
          v-if="item.redirect==='noRedirect'||index===levelList.length-1"
          class="no-redirect"
        >{{ item.meta.title }}</span>
        <a
          v-else
          @click.prevent="handleLink(item)"
        >{{ item.meta.title }}</a>
      </el-breadcrumb-item>
    </transition-group>
  </el-breadcrumb>
</template>

<script lang="ts">
import { Vue, Component, Watch } from 'vue-property-decorator';
import { RouteRecord, Route } from 'vue-router';
import pathToRegexp from 'path-to-regexp';

@Component
export default class extends Vue {
  private levelList: RouteRecord[] = [];

  @Watch('$route')
  onRoutesChange() {
    this.getBreadcrumb();
  }

  created() {
    this.getBreadcrumb();
  }

  private getBreadcrumb() {
    // only show routes with meta.title
    let matched = this.$route.matched.filter(
      item => item.meta && item.meta.title
    );
    const first = matched[0];

    if (!this.isDashboard(first)) {
      matched = [
        { path: '/dashboard', meta: { title: 'Dashboard' } } as RouteRecord
      ].concat(matched);
    }

    this.levelList = matched.filter(
      item => item.meta && item.meta.title && item.meta.breadcrumb !== false
    );
  }

  private isDashboard(route: RouteRecord) {
    const name = route && route.name;
    if (!name) {
      return false;
    }
    return name.trim().toLocaleLowerCase() === 'Dashboard'.toLocaleLowerCase();
  }
  private pathCompile(path: string) {
    // To solve this problem https://github.com/PanJiaChen/vue-element-admin/issues/561
    const { params } = this.$route;
    var toPath = pathToRegexp.compile(path);
    return toPath(params);
  }
  private handleLink(item: any) {
    const { redirect, path } = item;
    if (redirect) {
      this.$router.push(redirect);
      return;
    }
    this.$router.push(this.pathCompile(path));
  }
}
</script>

<style lang="scss" scoped>
.app-breadcrumb.el-breadcrumb {
  display: inline-block;
  font-size: 14px;
  line-height: 50px;
  margin-left: 8px;

  .no-redirect {
    color: #97a8be;
    cursor: text;
  }
}
</style>

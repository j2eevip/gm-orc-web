<template>
  <el-breadcrumb class="app-breadcrumb" separator="/">
    <transition-group name="breadcrumb">
      <el-breadcrumb-item v-for="(item, index) in levelList" :key="item.path">
        <span
          v-if="item.redirect === 'noRedirect' || index == levelList.length - 1"
          class="no-redirect"
        >{{ item.meta.title }}</span>
        <a v-else @click.prevent="handleLink(item)">{{ item.meta.title }}</a>
      </el-breadcrumb-item>
    </transition-group>
  </el-breadcrumb>
</template>

<script>
import { compile } from 'path-to-regexp';
import { computed } from 'vue';
import { useRouter, useRoute } from 'vue-router';

export default {
  name: 'Breadcrumb',
  setup() {
    const router = useRouter();
    const route = useRoute();

    const getBreadcrumb = () => {
      // only show routes with meta.title
      let matched = route.matched.filter((item) => item.meta && item.meta.title);
      const first = matched[0];

      if (!isDashboard(first)) {
        matched = [{ path: '/dashboard', meta: { title: 'Dashboard' } }].concat(
          matched
        );
      }

      return matched.filter(
        (item) => item.meta && item.meta.title && item.meta.breadcrumb !== false
      );
    };
    const isDashboard = (route) => {
      const name = route && route.name;
      if (!name) {
        return false;
      }
      return name.trim().toLocaleLowerCase() === 'Dashboard'.toLocaleLowerCase();
    };
    const pathCompile = (path) => {
      // To solve this problem https://github.com/PanJiaChen/vue-element-admin/issues/561
      const { params } = route;
      let toPath = compile(path);
      return toPath(params);
    };
    const handleLink = (item) => {
      const { redirect, path } = item;
      if (redirect) {
        router.push(redirect);
        return;
      }
      router.push(pathCompile(path));
    };

    // computed
    const levelList = computed(() => getBreadcrumb());

    return {
      levelList,
      handleLink,
    };
  },
};
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

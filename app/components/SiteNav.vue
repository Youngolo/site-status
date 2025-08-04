<!-- 导航栏 -->
<template>
  <nav id="nav" :class="{ scroll: statusStore.scrollTop > 0 }" :style="{ color: iconColor }">
    <div class="nav-content">
      <span class="logo">{{ config.public.siteTitle }}</span>
      <n-flex align="center" justify="end">
        <!-- 明暗切换 -->
        <Transition name="fade" mode="out-in">
          <n-button :key="themeIcon" :focusable="false" :color="iconColor" size="large" quaternary circle
            @click="toggleTheme">
            <template #icon>
              <Icon :name="themeIcon" />
            </template>
          </n-button>
        </Transition>
        <!-- 语言 -->
        <n-popselect v-model:value="statusStore.siteLang" :options="langData" trigger="click">
          <n-button :focusable="false" :color="iconColor" size="large" quaternary circle>
            <template #icon>
              <Icon name="icon:language" />
            </template>
          </n-button>
        </n-popselect>
        <!-- 菜单 -->
        <n-dropdown trigger="click" :options="navMenu">
          <n-button :focusable="false" :color="iconColor" size="large" quaternary circle>
            <template #icon>
              <Icon name="icon:menu" />
            </template>
          </n-button>
        </n-dropdown>
      </n-flex>
    </div>
  </nav>
</template>

<script setup lang="ts">
import { NIcon, type DropdownOption } from "naive-ui";
import { Icon } from "#components";
import { langData } from "~/assets/data/text";

const { t } = useI18n();
const colorMode = useColorMode();
const config = useRuntimeConfig();
const statusStore = useStatusStore();
// console.log(config.public.needLogin);
// const { public: configPublic } = useRuntimeConfig();
// const { siteIcp, version, showLink } = configPublic;

// 图标渲染
const renderIcon = (icon: string) => () =>
  h(NIcon, null, () => h(Icon, { name: icon }));

// 导航栏菜单
const navMenu = computed<DropdownOption[]>(() => [
  /*{
    key: "github",
    label: "GitHub",
    icon: renderIcon("icon:github"),
    props: {
      onClick: () => window.open("https://github.com/imsyy/site-status"),
    },
  },*/
  {
    key: "about",
    label: t("nav.about"),
    icon: renderIcon("icon:info"),
    props: {
      onClick: () => {
        window.$dialog.info({
          title: t("nav.about"),
          content: () => h('div', {
            // style: { color: 'red' },
            innerHTML: `<p style="max-width:700px;margin:24px auto 0;padding:0 16px;line-height:1.8;font-size:15px;">
  这是 16C 的公开状态页：<a href="https://www.16c.top/" target="_blank" style="font-weight:500;text-decoration:none;">青漫手记</a>，由<a href="https://uptimerobot.com/" target="_blank" style="color:#0073e6;font-weight:500;text-decoration:none;">UptimeRobot</a> 7×24 小时后台探针提供数据，再经自定义修饰版<a href="https://github.com/imsyy/site-status" target="_blank" style="color:#0073e6;font-weight:500;text-decoration:none;">SiteStatus</a>前端呈现。<br/>
  首页可一览近 60 天整体可用率、事件时间线、各区域实时延迟及故障详情；<br/>
  每 5 分钟自动刷新，支持暗/亮配色、移动端自适应；
</p>`
          }),
          positiveText: "我知道了",
          // negativeText: "取消",
          // transformOrigin: "center"
        });
      }
    }
  },
  {
    key: "logout",
    label: t("nav.logout"),
    show: statusStore.loginStatus && config.public.needLogin,
    icon: renderIcon("icon:logout"),
    props: {
      onClick: () => {
        window.$dialog.warning({
          title: "退出登录",
          content: "确定要退出登录吗?",
          positiveText: "确定",
          negativeText: "取消",
          transformOrigin: "center",
          onPositiveClick: async () => {
            const { code } = await $fetch("/api/logout", {
              method: "POST",
            });
            if (code !== 200) {
              window.$message.error("退出登录失败");
              return;
            }
            window.$message.success("退出登录成功");
            statusStore.loginStatus = false;
            localStorage.removeItem("authToken");
          },
        });
      },
    },
  },
]);

// 模式图标
const themeIcon = computed(() => `icon:${colorMode.preference}-mode`);

// 图标颜色
const iconColor = computed<string | undefined>(() =>
  statusStore.loginStatus && statusStore.scrollTop === 0 ? "#fff" : undefined,
);

// 切换明暗模式
const toggleTheme = () => {
  const themeList = ["light", "dark", "system"];
  const themeValue =
    themeList[(themeList.indexOf(colorMode.preference) + 1) % 3];
  if (themeValue) colorMode.preference = themeValue;
};
</script>

<style lang="scss" scoped>
nav {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  z-index: 100;
  transition:
    background-color 0.3s cubic-bezier(0.4, 0, 0.2, 1),
    box-shadow 0.3s cubic-bezier(0.4, 0, 0.2, 1);

  .nav-content {
    display: flex;
    align-items: center;
    justify-content: space-between;
    max-width: 900px;
    margin: 0 auto;
    padding: 30px 20px;
    transition: padding 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  }

  .logo {
    font-size: 20px;
    font-weight: bold;
    transition: color 0.3s cubic-bezier(0.4, 0, 0.2, 1);

    @media (max-width: 512px) {
      font-size: 16px;
    }
  }

  &.scroll {
    background-color: var(--main-card-color);
    border-bottom: solid 1px var(--mian-border-color);
    box-shadow: 0px 0px 8px 4px var(--main-box-shadow);

    .nav-content {
      padding: 12px 20px;
    }
  }
}
</style>

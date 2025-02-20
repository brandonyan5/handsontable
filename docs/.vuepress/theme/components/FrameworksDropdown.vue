<template>
  <nav class="nav-frameworks nav-links">
    <img :src="imageUrl"/>
    <!-- user links -->
    <nav class="nav-item" >
      <DropdownLink @item-click="onFrameworkClick" :item="item"></DropdownLink>
    </nav>
  </nav>
</template>

<script>
import DropdownLink from '@theme/components/DropdownLink.vue';

/**
 * Sets cookie bound to the docs URL domain and path.
 *
 * @param {string} name Cookie name.
 * @param {string} value Cookie value.
 */
function setCookie(name, value) {
  const hostname = window.location.hostname;
  const urlParts = hostname.split('.');
  let domainWithoutSubdomain = hostname;

  if (urlParts.length > 2) {
    domainWithoutSubdomain = urlParts.slice(1).join('.');
  }
  let str = `${name}=${value}`;

  str += `; Domain=${domainWithoutSubdomain}`;
  str += '; Path=/docs';
  str += `; Expires=${new Date(Date.now() + (365 * 24 * 60 * 60 * 1000)).toUTCString()}`; // 1 year

  document.cookie = str;
}

const frameworkIdToFullName = new Map([
  ['javascript', { name: 'JavaScript' }],
  ['react', { name: 'React' }],
  ['angular', { name: 'Angular', homepage: '/javascript-data-grid/angular-installation/' }],
  ['vue', { name: 'Vue 2', homepage: '/javascript-data-grid/vue-installation/' }],
  ['vue3', { name: 'Vue 3', homepage: '/javascript-data-grid/vue3-installation/' }],
]);

export default {
  name: 'FrameworksDropdown',
  components: {
    DropdownLink
  },
  watch: {
    $route(to) {
      this.detectLegacyFramework(to.fullPath);
    }
  },
  data() {
    return {
      legacyFramework: null
    };
  },
  methods: {
    onFrameworkClick(item) {
      setCookie('docs_fw', item.id === 'react' ? 'react' : 'javascript');
    },
    detectLegacyFramework(path) {
      const frameworkMatch = path.match(/javascript-data-grid\/(vue3|vue|angular)?/);

      this.legacyFramework = frameworkMatch ? frameworkMatch[1] : null;
    },
    getLink(framework) {
      const {
        homepage = `/${framework}${this.$page.frameworkSuffix}/`
      } = frameworkIdToFullName.get(framework);

      if (this.$page.currentVersion === this.$page.latestVersion) {
        return `/docs${homepage}`;
      }

      return `/docs/${this.$page.currentVersion}${homepage}`;
    },
    getFrameworkName(id) {
      return frameworkIdToFullName.get(id).name;
    },
    getFrameworkItems() {
      return Array.from(frameworkIdToFullName.entries()).map(([id, { name }]) => {
        return {
          id,
          text: name,
          link: this.getLink(id),
          target: '_self',
          isHtmlLink: true
        };
      });
    }
  },
  computed: {
    imageUrl() {
      const frameworkWithoutNumber = (this.legacyFramework ?? this.$page.currentFramework).replace(/\d+$/, '');

      return this.$withBase(`/img/pages/introduction/${frameworkWithoutNumber}.svg`);
    },
    item() {
      return {
        text: this.getFrameworkName(this.legacyFramework ?? this.$page.currentFramework),
        items: this.getFrameworkItems(),
      };
    }
  },
  created() {
    this.detectLegacyFramework(this.$route.fullPath);
  }
};
</script>

<style lang="stylus">
.nav-frameworks
  margin-left 1.4rem
  margin-right 1.5rem
  display inline-block
  position relative
  top -1px
  text-transform capitalize

  img
    height 16px
    position relative
    top 3px

  .nav-item
    margin-left 0.25rem

  .dropdown-title {
    text-transform capitalize
  }

  .icon.outbound
    display none

  .dropdown-wrapper
    height 1.8rem

  .dropdown-wrapper .nav-dropdown
    min-width 150px
    height auto !important
    box-sizing border-box
    max-height calc(100vh - 2.7rem)
    overflow-y auto
    position absolute
    top 100%
    background-color #fff
    padding 0.6rem 0
    border 1px solid #ddd
    border-bottom-color #ccc
    text-align left
    border-radius 6px
    white-space nowrap
    margin 0
    z-index 100
.dropdown-wrapper .dropdown-title .arrow, .dropdown-wrapper .mobile-dropdown-title .arrow
  margin-left 0.1rem

</style>

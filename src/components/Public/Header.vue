<template>
    <header>
      <b-navbar type="light" toggleable="md">
          <b-navbar-toggle target="public_nav_collapse">
            <label
              class="channel-toggle">
              <i class="icon-menu4"></i>
            </label>
          </b-navbar-toggle>
          <b-collapse is-nav id="public_nav_collapse" :class="{ visible }" @show="toggleNav(true)" @hide="toggleNav(false)">
            <menu-level id="menu_lvl_1"
                        :pages="pages"
                        :selectedPath="selectedPath"
                        :currentPageID="currentPageID">

              <li slot="collapse" id="public_nav_collapse_0">
                <span>{{ $t('navigation.more') }}</span><ul></ul>
              </li>
            </menu-level>
          </b-collapse>
        <span class="page-title">{{ currentPage.title }}</span>
        <router-link id="public_nav_to_admin_pannel" :to="{ name: 'admin' }" class="nav-link admin-panel">{{ $t('navigation.adminPanel') }}</router-link>
      </b-navbar>
    </header>
</template>
<script>
import { mapGetters } from 'vuex'
import MenuLevel from './MenuLevel'
import navbarCollapse from '@/mixins/navbar_collapse'

export default {
  components: {
    MenuLevel,
  },

  mixins: [ navbarCollapse ],

  props: {
    currentPageID: {
      type: String,
    },
  },

  data () {
    return {
      tree: [],
      collapsedCount: 0,
    }
  },

  computed: {
    ...mapGetters({
      pages: 'page/set',
      getPageByID: 'page/getByID',
    }),
    currentPage () {
      return this.getPageByID(this.currentPageID)
    },

    selectedPath () {
      const tt = (pp, pageID) => {
        if (!pp) {
          return false
        }

        for (let i = 0; i < pp.length; i++) {
          if (pp[i].pageID === pageID) {
            return [pageID]
          }

          let path = tt(pp[i].children, pageID)
          if (path) {
            path.unshift(pp[i].pageID)
            return path
          }
        }

        return false
      }

      return tt(this.tree, this.pageID) || []
    },
  },

  mounted () {
    this.$nextTick(() => {
      const nav = document.getElementById('menu_lvl_1')
      const bb = document.getElementById('public_nav_collapse')
      const collapse = document.getElementById('public_nav_collapse_0')
      const customCollapser = document.getElementById('public_nav_to_admin_pannel')

      setTimeout(() => { this.collapser(nav, bb, collapse, customCollapser) }, 1)
      window.onresize = () => {
        this.collapser(nav, bb, collapse, customCollapser)
      }
    })
  },

  created () {
    this.$crm.pageTree().then((result) => { this.tree = result })
  },

  methods: {
    collapser (nav, bb, collapse, customCollapser = null, buffer = 30) {
      if (!nav || !bb || !collapse) return

      const { children: navChildren = new HTMLCollection() } = nav
      const [ collapseBody ] = collapse.getElementsByTagName('UL')
      if (!collapseBody) return

      const { clientWidth: collapseWidth } = collapse
      let { clientWidth: bbWidth } = bb
      if (customCollapser) bbWidth -= customCollapser.clientWidth

      // Check if overflow possible
      if (nav.clientWidth + buffer >= bbWidth) {
        let c = null
        // -2; skip last element (the 'more' dropdown)
        for (let i = navChildren.length - 2; i >= 0; i--) {
          c = navChildren.item(i)

          const { clientWidth, offsetLeft } = c
          let elPos = clientWidth + offsetLeft + buffer

          // Initially collapse has visibility: hidden, so it's width is excluded untill visible
          if (this.collapsedCount) elPos += collapseWidth
          if (elPos >= bbWidth) {
            c.dataset.collapsed = true
            c.dataset.clientWidth = clientWidth

            // Initial show
            if (this.collapsedCount <= 0) {
              this.collapsedCount = 0
              collapse.style.visibility = 'visible'
            }
            this.collapsedCount++

            if (!collapseBody.firstChild) {
              collapseBody.append(c)
            } else {
              collapseBody.insertBefore(c, collapseBody.firstChild)
            }
          } else {
            return
          }
        }
      } else {
        // Check if any can be uncollapsed
        const collapsedNodes = document.querySelectorAll('[data-collapsed]')

        for (let i = 0; i < collapsedNodes.length; i++) {
          const cn = collapsedNodes[i]
          if ((parseInt(cn.dataset.clientWidth) || cn.clientWidth) + nav.clientWidth + buffer <= bbWidth) {
            delete cn.dataset.collapsed
            nav.insertBefore(cn, collapse)
            this.collapsedCount--

            if (this.collapsedCount <= 0) {
              this.collapsedCount = 0
              collapse.style.visibility = 'hidden'
            }
          } else {
            return
          }
        }
      }
    },
  },
}
</script>

<style lang="scss" scoped>
@import "@/assets/sass/components/header.scss";

.page-title {
  left: 50px;
  top: 12px;
  position: absolute;
  font-size: 15px;
  font-family: $crustsemibold;
  max-width: calc(100vw - 150px);
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

#public_nav_collapse {
  #public_nav_collapse_0 {
    visibility: hidden;

    a {
      overflow: hidden;
      text-overflow: ellipsis;
    }

    & > ul {
      overflow: hidden;

      & > li {
        & > ul {
          display: inline-block;
        }
      }
    }

    ul li ul {
      width: 100%;
    }

    &:hover > ul {
      max-width: 200px;

      & > li > ul {
        visibility: visible;
      }
    }
  }
}

@media (max-width: 768px) {
  #public_nav_collapse_0 {
    width: 100%;

    & > span {
      display: none;
    }

    & > ul {
      padding-left: 0;
      max-width: unset;
    }
  }
}
</style>

<template>
  <div
    style="overflow-x:hidden; scrollbar-width: none;"
    class="w-[292px] fixed top-0 pt-[15px] z-30 h-screen transition-position bg-[#f4f5f7] font-SfProDisplayNormal text-sm"
  >
    <AsideMenuSkeleton v-if="status == 'loading'" />
    <div v-if="status == 'success'">
      <div class="my-[10px]">
        <template
          v-for="(menuGroup, index) in menu"
          :key="`b-${index}`"
        >
          <pre>{{ menuGroup }}</pre>
          <router-link :to="'/' + menuGroup.path">
            <aside-menu-list
              :menu="menuGroup"
              @menu-click="menuClick"
            />
          </router-link>
        </template>
      </div>
    </div>
  </div>
</template>

<script>
import AsideMenuList from '@/components/AsideMenu/AsideMenuList.vue'
import AsideMenuSkeleton from '@/components/AsideMenu/AsideMenuSkeleton.vue'
import 'v-calendar/dist/style.css'
import { UID_TO_ACTION } from '@/store/helpers/functions'

import warn from '@/icons/warn.js'
import { mdiMenu } from '@mdi/js'

export default {
  components: {
    AsideMenuSkeleton,
    AsideMenuList
  },
  props: {
    menu: {
      type: Array,
      default: () => []
    }
  },
  emits: ['closeSubMenu'],
  data () {
    return {
      newDayTimerID: 0,
      dateToday: '',
      mdiMenu,
      currentDay: null,
      visitedDay: null,
      userParentId: null,
      warn,
      showFreeModal: false,
      modalOneActive: false,
      lastVisitedDate: this.navStack && this.navStack.length && this.navStack[this.navStack.length - 1] && this.navStack[this.navStack.length - 1].uid && this.navStack[this.navStack.length - 1].uid === '901841d9-0016-491d-ad66-8ee42d2b496b' && this.navStack[this.navStack.length - 1].param ? new Date(this.navStack[this.navStack.length - 1].param) : new Date(),
      currentSettingsTab: 'account'
    }
  },
  computed: {
    status () {
      return this.$store.state.navigator.status
    },
    lastTab () {
      return this.$store.state.navigator.lastTab
    },
    isAsideMobileExpanded () {
      return this.$store.state.isAsideMobileExpanded
    },
    isPropertiesMobileExpanded () {
      return this.$store.state.isPropertiesMobileExpanded
    },
    navStack () {
      return this.$store.state.navbar.navStack
    },
    user () {
      return this.$store.state.user.user
    },
    storeNavigator () {
      return this.$store.getters.sortedNavigator
    }
  },
  mounted () {
    let currDate = new Date()
    currDate.setHours(0, 0, 0, 0)
    this.newDayTimerID = setInterval(() => {
      const newDate = new Date()
      newDate.setHours(0, 0, 0, 0)
      if (currDate.getTime() !== newDate.getTime()) {
        currDate = newDate
        this.onNewDay()
      }
    }, 1000)
  },
  beforeUnmount () {
    clearInterval(this.newDayTimerID)
  },
  methods: {
    dateToLabelFormat (calendarDate) {
      const day = calendarDate.getDate()
      const month = calendarDate.toLocaleString('default', { month: 'short' })
      const weekday = calendarDate.toLocaleString('default', { weekday: 'short' })
      return day + ' ' + month + ', ' + weekday
    },
    // TODO: clean up messy logic
    menuClick (event, item) {
      localStorage.setItem('lastTab', 'directory')
      this.$emit('closeSubMenu')

      console.log('directory', item)

      // Если уже находимся на этой вкладке игнорировать дальнейший код
      if (this.checkOnWhichTab(item)) {
        return
      }
      this.userParentId = null
      this.visitedDay = ''
      if (item.uid === '901841d9-0016-491d-ad66-8ee42d2b496b') {
        this.dateToday = new Date()
      }

      if (item.path === 'new_private_boards' || item.path === 'new_private_projects') {
        this.$store.state.navigator.submenu.status = true
        this.$store.state.navigator.submenu.path = item.path
        return
      } else {
        this.$store.state.navigator.submenu.status = false
      }

      if (this.isPropertiesMobileExpanded) {
        this.$store.dispatch('asidePropertiesToggle', false)
      }
      if (this.isAsideMobileExpanded) {
        this.$store.dispatch('asideMobileToggle', false)
      }

      if (['account', 'tarif', 'option', 'karma', 'support'].includes(item.type)) {
        this.$store.state.navigator.currentSettingsTab = item.type
        const navElem = {
          name: item.label,
          key: 'greedSource',
          value: { uid: item.uid, param: null },
          greedPath: item.type
        }
        localStorage.setItem('currentSettingsTab', item.type)
        this.$store.commit('updateStackWithInitValue', navElem)
        return
      }

      // do it now
      if (item.uid === '2cf6b167-6506-4b05-bc34-70a8d88e3b25') {
        window.ym(89796698, 'reachGoal', 'doitnow')
        if (this.user.tarif !== 'alpha' && this.user.tarif !== 'trial') {
          this.showFreeModal = true
          return
        }
        const navElem = {
          name: item.label,
          key: 'greedSource',
          value: { uid: item.uid, param: null },
          greedPath: 'doitnow'
        }
        this.$store.commit('updateStackWithInitValue', navElem)
        this.$store.commit('basic', { key: 'mainSectionState', value: 'greed' })
        this.$store.commit('basic', { key: 'greedPath', value: 'doitnow' })
        return
      }

      // other
      if (item.uid === '757be87d-c269-40e0-b224-6b2bb0e4f97d') {
        const navElem = {
          name: item.label,
          key: 'greedSource',
          value: { uid: item.uid, param: null },
          greedPath: 'other'
        }
        this.$store.commit('updateStackWithInitValue', navElem)
        this.$store.commit('basic', { key: 'mainSectionState', value: 'greed' })
        this.$store.commit('basic', { key: 'greedPath', value: 'other' })
        // отображаем навбар в прочее
        this.$store.state.onboarding.hideNavBar = false
        return
      }

      // colors & tags
      if (item.uid === 'ed8039ae-f3de-4369-8f32-829d401056e9' || item.uid === '00a5b3de-9474-404d-b3ba-83f488ac6d30') {
        this.$store.commit('basic', { key: 'mainSectionState', value: 'greed' })
        this.$store.commit('basic', { key: 'greedPath', value: item.path })
        const navElem = {
          name: item.label,
          key: 'greedSource',
          greedPath: item.path,
          value: this.storeNavigator[item.path]?.items
        }
        this.$store.commit('updateStackWithInitValue', navElem)
        this.$store.commit('basic', { key: 'greedSource', value: this.storeNavigator[item.path]?.items })
        return
      }

      // Tasks list source
      if (UID_TO_ACTION[item.uid] && item.type === 'uid') {
        this.$store.dispatch(UID_TO_ACTION[item.uid])
        const navElem = {
          name: item.label,
          key: 'taskListSource',
          value: { uid: item.uid, param: new Date() },
          typeVal: new Date(),
          type: 'date'
        }
        this.$store.commit('updateStackWithInitValue', navElem)
        if (item.uid === '901841d9-0016-491d-ad66-8ee42d2b496b') { this.lastVisitedDate = new Date() } // desktop check
        this.$store.commit('basic', { key: 'taskListSource', value: { uid: item.uid, param: null } })
        this.$store.commit('basic', { key: 'mainSectionState', value: 'tasks' })
      // Grid source (projects, employees, colors, tags)
      } else {
        this.$store.commit('basic', { key: 'mainSectionState', value: 'greed' })
        this.$store.commit('basic', { key: 'greedPath', value: item.path })
        if (item.path === 'new_private_projects' || item.path === 'new_emps' || item.path === 'new_delegate' || item.path === 'new_private_boards') {
          const navElem = {
            name: item.label,
            key: 'greedSource',
            greedPath: item.path,
            value: this.storeNavigator[item.path]
          }
          this.$store.commit('updateStackWithInitValue', navElem)
          this.$store.commit('basic', { key: 'greedSource', value: this.storeNavigator[item.path] })
        } else {
          console.log(item, 'item')
          const navElem = {
            name: item.label,
            key: 'greedSource',
            greedPath: item.path,
            value: this.storeNavigator[item.path]?.items
          }
          this.$store.commit('updateStackWithInitValue', navElem)
          this.$store.commit('basic', { key: 'greedSource', value: this.storeNavigator[item.path]?.items })
        }
      }
    },
    checkOnWhichTab (item) {
      const lastNavStack = this.navStack[this.navStack.length - 1]
      if (lastNavStack?.value?.uid === item.uid ||
        lastNavStack?.uid === item.uid ||
        lastNavStack?.name === item.label ||
        (lastNavStack.name && item.name && lastNavStack?.name === item.name)) {
        return true
      }
    }
  }
}
</script>

<template>
  <div class="w-full pt-[30px]">
    <div>
      <div class="flex items-center justify-between w-full">
        <p
          class="font-['Roboto'] text-[#424242] text-[19px] leading-[22px] font-bold"
        >
          Уведомления
        </p>
        <div class="flex">
          <Icon
            :path="listView.path"
            :width="listView.width"
            :height="listView.height"
            :box="listView.viewBox"
            class="cursor-pointer hover:text-gray-800 mr-2"
            :class="{
              'text-gray-800': !isGridView,
              'text-gray-400': isGridView
            }"
            @click="updateGridView(false)"
          />
          <Icon
            :path="gridView.path"
            :width="gridView.width"
            :height="gridView.height"
            :box="gridView.viewBox"
            class="cursor-pointer hover:text-gray-800 mr-2"
            :class="{
              'text-gray-800': isGridView,
              'text-gray-400': !isGridView
            }"
            @click="updateGridView(true)"
          />
        </div>
      </div>
      <div
        class="grid gap-2 mt-3 grid-cols-1"
        :class="{
          'md:grid-cols-2 lg:grid-cols-4': isGridView,
          'lg:grid-cols-2': isPropertiesMobileExpanded && isGridView
        }"
      >
        <template
          v-for="notify in notificationtasks"
          :key="notify.uid"
        >
          <ListBlocItem
            :title="notify.name"
            @click="gotoReglamentContent(notify)"
          >
            <Icon
              :box="notifyIcon.viewBox"
              :path="notifyIcon.path"
              height="24"
              width="24"
              w="-mt-[2px]"
              h="-ml-[2px]"
              class="text-[#606061]"
            />
          </ListBlocItem>
        </template>
      </div>
    </div>
  </div>
</template>

<script>
import { NOTIFICATION_TASKS_GENERATE, NOTIFICATION_TASKS_CLEAR } from '@/store/actions/notification_tasks'
import { setLocalStorageItem } from '@/store/helpers/functions'
import ListBlocItem from '@/components/Common/ListBlocItem.vue'
import Icon from '@/components/Icon.vue'
import gridView from '@/icons/grid-view.js'
import listView from '@/icons/list-view.js'

export default {
  components: {
    ListBlocItem,
    Icon
  },
  data () {
    return {
      notificationtasks: [],
      listView,
      gridView,
      notifyIcon: {
        path: [
          'M13 4.73762C8.44731 4.73762 4.73768 8.44725 4.73768 13C4.73768 17.5527 8.44731 21.2624 13 21.2624C17.5528 21.2624 21.2624 17.5527 21.2624 13C21.2624 8.44725 17.5528 4.73762 13 4.73762ZM19.0704 13.7554C19.4084 13.7554 19.6732 14.0504 19.6039 14.3813C19.057 16.9939 16.9977 19.0523 14.3846 19.5978C14.0556 19.6665 13.7622 19.4033 13.7622 19.0672C13.7622 18.756 13.51 18.5037 13.1988 18.5037H12.8081C12.4969 18.5037 12.2446 18.756 12.2446 19.0672C12.2446 19.4033 11.9513 19.6665 11.6222 19.5978C9.00797 19.052 6.94799 16.9921 6.4022 14.3778C6.33351 14.0488 6.59674 13.7554 6.93287 13.7554C7.24404 13.7554 7.4963 13.5031 7.4963 13.192V12.8044C7.4963 12.4915 7.24262 12.2378 6.92969 12.2378C6.59166 12.2378 6.32693 11.9428 6.39619 11.6119C6.94308 8.99933 9.00239 6.94094 11.6155 6.3954C11.9445 6.3267 12.2379 6.58993 12.2379 6.92606C12.2379 7.23724 12.4901 7.48949 12.8013 7.48949H13.192C13.5032 7.48949 13.7555 7.23724 13.7555 6.92606C13.7555 6.58993 14.0488 6.3267 14.3779 6.3954C16.9921 6.94118 19.0521 9.00116 19.5979 11.6154C19.6666 11.9445 19.4033 12.2378 19.0672 12.2378C18.756 12.2378 18.5038 12.4901 18.5038 12.8012V13.1888C18.5038 13.5017 18.7575 13.7554 19.0704 13.7554Z',
          'M24.95 16.0014C25.5023 16.0014 25.95 15.5537 25.95 15.0014V10.9986C25.95 10.4463 25.5023 9.99855 24.95 9.99855H24.8128C24.3614 9.99855 23.9703 9.6945 23.8235 9.26764C23.6725 8.82851 23.4953 8.40014 23.292 7.98254C23.0946 7.57709 23.1563 7.08617 23.4752 6.7673L23.577 6.66553C23.9675 6.27501 23.9675 5.64184 23.577 5.25132L20.7487 2.42306C20.3582 2.03253 19.725 2.03253 19.3345 2.42306L19.2327 2.52482C18.9139 2.84369 18.4229 2.90546 18.0175 2.70806C17.5999 2.50474 17.1715 2.32757 16.7324 2.17655C16.3055 2.02974 16.0015 1.63867 16.0015 1.18726V1.04999C16.0015 0.497703 15.5538 0.0499878 15.0015 0.0499878H10.9986C10.4463 0.0499878 9.99861 0.497703 9.99861 1.04999V1.18727C9.99861 1.63867 9.69456 2.02974 9.2677 2.17655C8.82857 2.32757 8.4002 2.50474 7.9826 2.70806C7.57715 2.90546 7.08623 2.84369 6.76736 2.52482L6.66672 2.42418C6.27575 2.03322 5.64172 2.03372 5.25138 2.42531L2.42762 5.25806C2.03815 5.64877 2.03865 6.28107 2.42874 6.67115L2.53163 6.77404C2.8505 7.09291 2.91226 7.58384 2.71486 7.98928C2.51155 8.40689 2.33438 8.83525 2.18335 9.27439C2.03655 9.70125 1.64547 10.0053 1.19407 10.0053H1.05005C0.497764 10.0053 0.0500488 10.453 0.0500488 11.0053V15.0082C0.0500488 15.5604 0.497764 16.0082 1.05005 16.0082H1.18733C1.63873 16.0082 2.02981 16.3122 2.17661 16.7391C2.32763 17.1782 2.5048 17.6066 2.70812 18.0242C2.90552 18.4296 2.84376 18.9205 2.52488 19.2394L2.42312 19.3412C2.03259 19.7317 2.03259 20.3649 2.42312 20.7554L5.25138 23.5837C5.6419 23.9742 6.27507 23.9742 6.66559 23.5837L6.76736 23.4819C7.08623 23.163 7.57715 23.1013 7.9826 23.2987C8.4002 23.502 8.82857 23.6791 9.2677 23.8302C9.69456 23.977 9.99861 24.368 9.99861 24.8194V24.95C9.99861 25.5023 10.4463 25.95 10.9986 25.95H15.0015C15.5538 25.95 16.0015 25.5023 16.0015 24.95V24.8127C16.0015 24.3613 16.3055 23.9702 16.7324 23.8234C17.1715 23.6724 17.5999 23.4952 18.0175 23.2919C18.4229 23.0945 18.9139 23.1563 19.2327 23.4751L19.3345 23.5769C19.725 23.9674 20.3582 23.9674 20.7487 23.5769L23.577 20.7487C23.9675 20.3581 23.9675 19.725 23.577 19.3344L23.4752 19.2327C23.1563 18.9138 23.0946 18.4229 23.292 18.0174C23.4953 17.5998 23.6725 17.1715 23.8235 16.7323C23.9703 16.3055 24.3614 16.0014 24.8128 16.0014H24.95ZM22.8339 14.4832C22.8339 14.4836 22.8339 14.484 22.8338 14.4845L22.7058 15.0841C22.4562 16.2645 21.9909 17.3774 21.3366 18.3891C21.1438 18.6882 21.1857 19.0813 21.4374 19.3329L21.5092 19.4047C21.8534 19.749 21.8534 20.3071 21.5092 20.6514L20.6582 21.5024C20.3139 21.8466 19.7558 21.8466 19.4115 21.5024L19.3398 21.4306C19.0881 21.1789 18.695 21.137 18.3959 21.3298C17.3842 21.984 16.2713 22.4494 15.0909 22.699C14.7409 22.7737 14.4906 23.083 14.4906 23.441V23.542C14.4906 24.03 14.095 24.4257 13.607 24.4257H12.4066C11.9185 24.4257 11.5229 24.03 11.5229 23.542V23.441C11.5229 23.083 11.2727 22.7737 10.9226 22.699C9.74231 22.4494 8.62942 21.984 7.6177 21.3298C7.3186 21.137 6.92548 21.1789 6.67383 21.4306L6.60204 21.5024C6.25779 21.8466 5.69965 21.8466 5.3554 21.5024L4.50441 20.6514C4.16016 20.3071 4.16016 19.749 4.50441 19.4047L4.5762 19.3329C4.82785 19.0813 4.86982 18.6882 4.67697 18.3891C4.02273 17.3774 3.55734 16.2645 3.30778 15.0841C3.23305 14.734 2.92378 14.4838 2.56582 14.4838H2.4587C1.96658 14.4838 1.56763 14.0849 1.56763 13.5928V12.3998C1.56763 11.9117 1.96324 11.5161 2.45125 11.5161H2.55233C2.9103 11.5161 3.21956 11.2659 3.29429 10.9158C3.54385 9.73551 4.00924 8.62262 4.66349 7.6109C4.85646 7.31159 4.81506 6.9183 4.564 6.66574L4.49094 6.59224C4.14638 6.24562 4.14721 5.68555 4.49281 5.33996L5.34191 4.49086C5.68616 4.14661 6.2443 4.14661 6.58855 4.49086L6.66034 4.56265C6.91199 4.8143 7.30511 4.85627 7.60421 4.66342C8.61593 4.00918 9.72882 3.54379 10.9092 3.29423C11.2592 3.2195 11.5094 2.91023 11.5094 2.55227V2.45119C11.5094 1.96318 11.9051 1.56757 12.3931 1.56757H13.5935C14.0815 1.56757 14.4772 1.96318 14.4772 2.45119V2.55227C14.4772 2.91023 14.7274 3.2195 15.0774 3.29423C16.2578 3.54379 17.3707 4.00918 18.3824 4.66342C18.6815 4.85627 19.0746 4.8143 19.3263 4.56265L19.3981 4.49086C19.7423 4.14661 20.3004 4.14661 20.6447 4.49086L21.4957 5.34185C21.8399 5.6861 21.8399 6.24424 21.4957 6.58849L21.4239 6.66028C21.1722 6.91193 21.1303 7.30505 21.3231 7.60415C21.9774 8.61587 22.4428 9.72876 22.6923 10.9091C22.767 11.2592 23.0763 11.5094 23.4343 11.5094H23.5354C24.0234 11.5094 24.419 11.905 24.419 12.393V13.6846C24.419 14.1223 24.0642 14.4771 23.6265 14.4771H22.84C22.8367 14.4771 22.8339 14.4798 22.8339 14.4832Z',
          'M13.7555 9.76248C13.7555 9.34528 13.4173 9.00707 13 9.00707C12.5828 9.00707 12.2446 9.34528 12.2446 9.76249V12.9028C12.2446 13.168 12.35 13.4223 12.5375 13.6099L14.0758 15.1482C14.372 15.4443 14.8521 15.4443 15.1483 15.1482C15.4444 14.8521 15.4444 14.3719 15.1483 14.0758L14.0484 12.9759C13.8608 12.7883 13.7555 12.534 13.7555 12.2688V9.76248Z'
        ],
        viewBox: '0 0 30 30',
        width: 30,
        height: 30
      }
    }
  },
  computed: {
    notificationTasks () {
      return this.$store.state.notificationtasks.notificationtasks
    },
    storeNavigator () {
      return this.$store.state.navigator.navigator
    },
    isGridView () {
      return this.$store.state.isGridView
    },
    isPropertiesMobileExpanded () {
      return this.$store.state.isPropertiesMobileExpanded
    }
  },
  mounted () {
    this.$store.commit('basic', { key: 'mainSectionState', value: 'greed' })
    this.$store.commit('basic', { key: 'greedPath', value: 'notifications' })
    const navElem = {
      name: 'Цвета',
      key: 'greedSource',
      greedPath: 'notifications',
      value: this.storeNavigator.notifications?.items
    }
    this.$store.commit('updateStackWithInitValue', navElem)
    this.$store.commit('basic', { key: 'greedSource', value: navElem.value })
    this.$store.dispatch(NOTIFICATION_TASKS_GENERATE).then(() => {
      this.notificationtasks = this.notificationTasks
    })
  },
  unmounted () {
    this.$store.dispatch(NOTIFICATION_TASKS_CLEAR)
  },
  methods: {
    gotoReglamentContent (notify) {
      this.$store.commit('updateStackWithInitValue', {
        name: 'Регламенты',
        key: 'greedSource',
        greedPath: 'reglaments',
        value: this.storeNavigator.reglaments?.items
      })
      this.$store.commit('basic', { key: 'greedSource', value: this.storeNavigator.reglaments?.items })
      this.$store.commit('basic', { key: 'mainSectionState', value: 'greed' })
      this.$store.commit('basic', { key: 'greedPath', value: 'reglaments' })
      const reglament = this.$store.state.reglaments.reglaments[notify.uid]
      this.$store.commit('basic', {
        key: 'reglamentSource',
        value: { uid: '92413f6c-2ef3-476e-9429-e76d7818685d', param: notify.uid }
      })
      const navElem = {
        name: this.name,
        key: 'greedSource',
        uid: this.uid,
        global_property_uid: '92413f6c-2ef3-476e-9429-e76d7818685d',
        greedPath: 'reglament_content',
        value: []
      }
      this.$store.commit('pushIntoNavStack', navElem)
      this.$store.commit('basic', { key: 'greedSource', value: reglament })
      this.$store.commit('basic', { key: 'mainSectionState', value: 'greed' })
      this.$store.commit('basic', { key: 'greedPath', value: 'reglament_content' })
    },
    updateGridView (value) {
      this.$store.commit('basic', { key: 'isGridView', value: value })
      setLocalStorageItem('isGridView', value)
    }
  }
}
</script>

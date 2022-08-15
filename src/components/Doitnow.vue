<template>
  <div
    v-if="displayModal"
    class="max-w-xl mx-auto"
  >
    <div
      class="flex flex-col"
    >
      <img
        class="mx-auto mt-10"
        width="320"
        height="314"
        src="@/assets/images/emptydoitnow.png"
        alt="Empty task image"
      >
      <p class="font-bold p-3">
        Работайте только с одной конкретной задачей и не отвлекайтесь на другие
      </p>
      <ul class="list-decimal pl-[30px]">
        <li class="p-3 text-sm">
          Очередь выдает по одной задаче в один момент времени
        </li>
        <li class="p-3 text-sm">
          У вас нет возможности выбрать задачу и вы не знаете, какая будет следующей
        </li>
        <li class="p-3 text-sm">
          В Очередь попадают непрочитанные и просроченные задачи и поручения, а также поручения и задачи на сегодня
        </li>
      </ul>
      <button
        class="bg-[#FF912380] px-2 rounded-[8px] text-black text-sm mr-1 hover:bg-[#F5DEB3] w-[156px] h-[51px] mr-auto ml-auto mt-[20px]"
        @click="okToModal"
      >
        Понятно
      </button>
    </div>
  </div>
  <div
    v-else-if="tasksCount && !isLoading"
    class="flex items-center mt-[70px] mb-5 justify-between"
  >
    <!-- header -->
    <div class="flex items-center">
      <div
        class="font-Roboto font-medium text-sm bg-gray-200 px-2.5 mx-5 py-2 rounded-lg flex"
      >
        В очереди задач: {{ tasksCount }}
      </div>
    </div>
    <button
      class="border border-slate-600 py-3 px-4 rounded-lg mr-5 hover:bg-gray-300 text-sm bg-opacity-70 font-medium flex w-[181px] items-center justify-center"
      @click="nextTask"
    >
      <span class="pr-2">Следующая задача</span>
      <Icon
        :height="arrowForw.height"
        :width="arrowForw.width"
        :view="arrowForw.viewBox"
        :path="arrowForw.path"
      />
    </button>
  </div>
  <DoitnowSkeleton
    v-if="isLoading"
    class="mt-20"
  />
  <transition :name="taskTransition">
    <div
      v-if="!(tasksCount === 0 && !isLoading) && !displayModal"
      class="px-5"
    >
      <a
        class="dark:bg-gray-700 cursor-pointer dark:text-gray-100 rounded-lg text-[14px] breadcrumbs text-[#7E7E80] font-medium"
        target="_blank"
        :href="`${currentLocation}/task/${firstTask?.uid}`"
      >
        Открыть задачу
      </a>
    </div>
  </transition>
  <transition :name="taskTransition">
    <div>
      <DoitnowTask
        v-if="!displayModal && tasksCount && !isLoading && !notify"
        :key="firstTask.uid"
        :task="firstTask"
        :childrens="childrens"
        :sub-tasks="subTasks"
        :colors="colors"
        :tags="tags"
        :user="user"
        :task-messages="taskMessages.slice().reverse()"
        :employees="employees"
        :projects="projects"
        @clickTask="onClickTask"
        @nextTask="nextTask"
        @changeValue="changeValue"
        @readTask="readTask"
      />
      <DoitnowNotificationTask
        v-if="!displayModal && tasksCount && !isLoading && notify"
        :name="firstTask.name"
        :uid="firstTask.uid"
      />
    </div>
  </transition>
  <DoitnowEmpty
    v-if="(tasksCount === 0 && !isLoading)"
    @clickPlanning="goToNextDay"
  />
</template>

<script>
import { copyText } from 'vue3-clipboard'
import * as FILES from '@/store/actions/taskfiles.js'
import * as MSG from '@/store/actions/taskmessages.js'
import * as TASK from '@/store/actions/tasks.js'

import DoitnowEmpty from '@/components/Doitnow/DoitnowEmpty.vue'
import DoitnowTask from '@/components/Doitnow/DoitnowTask.vue'
import DoitnowSkeleton from '@/components/Doitnow/DoitnowSkeleton.vue'
import Icon from '@/components/Icon.vue'

import arrowForw from '@/icons/arrow-forw-sm.js'
import initWebSync from '@/websync/index.js'
import initInspectorSocket from '@/inspector/index.js'
import { PUSH_COLOR } from '@/store/actions/colors'
import { USER_VIEWED_MODAL } from '@/store/actions/onboarding.js'

import { NAVIGATOR_REQUEST } from '@/store/actions/navigator'
import { USER_REQUEST } from '@/store/actions/user'
import DoitnowNotificationTask from './Doitnow/DoitnowNotificationTask.vue'

export default {
  components: {
    DoitnowEmpty,
    DoitnowSkeleton,
    DoitnowTask,
    Icon,
    DoitnowNotificationTask
  },
  setup () {
    return {
      arrowForw
    }
  },
  data: () => ({
    unreadTasks: [],
    overdueTasks: [],
    todayTasks: [],
    readyTasks: [],
    unreadDelegateByMe: [],
    unreadDelegateToMe: [],
    readyTasksReaded: [],
    readyTasksUnreaded: [],
    openedTasks: [],
    slidesCopy: [],
    projectTasks: [],
    unsortedTasks: [],
    overdueReaded: [],
    tasksLoaded: false,
    childrens: []
  }),
  computed: {
    tasksCount () {
      return (
        this.slidesCopy.length +
        this.unreadTasks.length +
        this.overdueTasks.length +
        this.readyTasks.length +
        this.todayTasks.length +
        this.notifies.length
      )
    },
    currentLocation () {
      return window.location.origin
    },
    firstTask () {
      if (this.slidesCopy.length && this.justRegistered) {
        return this.slidesCopy[0]
      }
      if (this.notifies.length) {
        return this.notifies[0]
      }
      if (this.unreadTasks.length) {
        return this.unreadTasks[0]
      }
      if (this.readyTasks.length) {
        return this.readyTasks[0]
      }
      if (this.overdueTasks.length) {
        return this.overdueTasks[0]
      }
      if (this.todayTasks.length) {
        return this.todayTasks[0]
      }
      return null
    },
    slides () {
      return this.$store.state.slides.slides
    },
    taskMessages () {
      return this.$store.state.taskfilesandmessages.messages
    },
    taskFiles () {
      return this.$store.state.taskfilesandmessages.files
    },
    employees () {
      return this.$store.state.employees.employees
    },
    projects () {
      return this.$store.state.projects.projects
    },
    colors () {
      return this.$store.state.colors.colors
    },
    tags () {
      return this.$store.state.tasks.tags
    },
    isLoading () {
      return this.$store.state.tasks.status === 'loading'
    },
    user () {
      return this.$store.state.user.user
    },
    subTasks () {
      return this.$store.state.tasks.subtasks.tasks
    },
    taskTransition () {
      return this.tasksLoaded ? 'slide-in-fade-out' : ''
    },
    displayModal () {
      return !this.$store.state.onboarding?.visitedModals?.includes('doitnow') && this.$store.state.onboarding?.showModals
    },
    notifies () {
      return this.$store.state.notificationTasks.notificationTasks
    },
    notify () {
      return !!this.firstTask.notify
    },
    justRegistered () {
      return this.$store.state.onboarding.justRegistered
    }
  },
  watch: {
    firstTask (newtask, oldtask) {
      if (newtask && newtask.uid && !newtask.notify) {
        this.$store.dispatch(TASK.GET_TASK_CHILDRENS, newtask.uid)
          .then((resp) => {
            this.childrens = resp.data.tasks
          })
        this.$store.commit(TASK.SELECT_TASK, newtask)
        this.$store.dispatch(MSG.MESSAGES_REQUEST, newtask.uid)
          .then(() => {
            this.$store.dispatch(FILES.FILES_REQUEST, newtask.uid)
              .then(() => {
                this.$store.dispatch(MSG.INSPECTOR_MESSAGES_REQUEST, newtask.uid)
                  .then(() => {
                    this.$store.commit(FILES.MERGE_FILES_WITH_MESSAGES)
                  })
              })
          })
        this.$store.dispatch(MSG.INSPECTOR_MESSAGES_REQUEST, newtask.uid)
        this.$store.dispatch(TASK.SUBTASKS_REQUEST, newtask.uid)
      }
    }
  },
  mounted: function () {
    const navLoaded = this.$store.state.navigator.hasLoadedOnce
    const userLoaded = this.$store.state.user.hasLoadedOnce
    // this.$store.dispatch('NOTIFICATION_TASKS_GENERATE')
    // сначала запрашиваем пользователя, потом регламенты, потом навигатор
    if (!userLoaded || !navLoaded) {
      this.$store.dispatch(USER_REQUEST).then(() => {
        // запрос регламентов
        const data = {
          organization: this.$store?.state?.user?.user?.owner_email,
          user_uid: this.$store?.state?.user?.user?.current_user_uid
        }
        let reglaments = []
        this.$store.dispatch('REGLAMENTS_REQUEST', data).then(resp => {
          reglaments = resp.data
        }).finally(() => {
          // запрос навигатора
          this.$store.dispatch(NAVIGATOR_REQUEST).then((resp) => {
            this.$store.state.navigator.navigator.reglaments = {
              uid: 'fake-uid',
              items: reglaments
            }
            try {
              initWebSync()
              initInspectorSocket()
            } catch (e) {}
          }).then(() => {
            this.$store.dispatch('NOTIFICATION_TASKS_GENERATE')
          })
        })
      })
    }
    if (userLoaded && navLoaded) {
      this.$store.dispatch('NOTIFICATION_TASKS_GENERATE')
    }
    if (this.justRegistered) {
      this.slidesCopy = [...this.slides]
    }
    if (!this.displayModal) {
      this.loadAllTasks()
    }
    this.$store.dispatch('fullScreenToggle', 'add')
  },
  unmounted: function () {
    this.$store.commit('NOTIFICATION_TASKS_CLEAR')
  },
  methods: {
    loadAllTasks: function () {
      this.$store.dispatch(TASK.DOITNOW_TASKS_REQUEST)
        .then((result) => {
          // сортировка непрочитанных
          for (let i = 0; i < result[0].length; i++) {
            // Поручено мной
            if (result[0][i].uid_customer === this.user?.current_user_uid) {
              this.unreadDelegateByMe.unshift(result[0][i])
            } else {
              // Поручено мне
              if (result[0][i].uid_performer === this.user?.current_user_uid) {
                this.unreadDelegateToMe.unshift(result[0][i])
              } else {
                // Готово к сдаче
                if (result[0][i].status === 5) {
                  this.readyTasksUnreaded.push(result[0][i])
                } else {
                  // Доступ
                  if (result[0][i].emails.includes(this.user?.current_user_email) || (result[0][i].uid_project !== '00000000-0000-0000-0000-000000000000')) {
                    this.openedTasks.push(result[0][i])
                  }
                }
              }
            }
          }
          // Сортировка просроченных
          for (let i = 0; i < result[1].length; i++) {
            if (result[1][i].readed) {
              this.overdueReaded.push(result[1][i])
            }
          }
          // Готово к сдаче
          this.$store.dispatch(TASK.READY_FOR_COMPLITION_TASKS_REQUEST)
            .then((resp) => {
              if (resp.data.anothers_markers.length) {
                this.$store.commit(PUSH_COLOR, resp.data.anothers_markers)
              }
              if (resp.data.anothers_tags.length) {
                this.$store.commit(TASK.ADD_TASK_TAGS, resp.data.anothers_tags)
              }
              for (let i = 0; i < resp.data.tasks; i++) {
                if (resp.data.tasks[i].readed) {
                  this.readyTasksReaded.push(resp.data.tasks[i])
                }
              }
            })
          // Отправляем в главный массив (непрочитанное) отсортированные массивы по очереди
          this.unreadTasks = [...this.unreadDelegateByMe, ...this.unreadDelegateToMe,
            ...this.readyTasksUnreaded, ...this.projectTasks, ...this.unsortedTasks]
          // Отправляем в главный массив (просрочено) отсортированные данные
          this.overdueTasks = [...this.overdueReaded]
          // Отправляем в главный массив (готовые) отсортированные данные
          this.readyTasks = [...this.readyTasksReaded]
          this.todayTasks = [...result[2]]
          this.openedTasks = [...this.openedTasks]
          // удаляем из массивов задачи со статусом "завершено"
          this.unreadTasks = this.unreadTasks.filter(task => (task.status !== 1) && (task.status !== 8))
          this.overdueTasks = this.overdueTasks.filter(task => (task.status !== 1) && (task.status !== 8))
          this.readyTasks = this.readyTasks.filter(task => (task.status !== 1) && (task.status !== 8))
          this.todayTasks = this.todayTasks.filter(task => (task.status !== 1) && (task.status !== 8))
          this.openedTasks = this.openedTasks.filter(task => (task.status !== 1) && (task.status !== 8))
        })
        .then(() => {
          this.tasksLoaded = true
        })
    },
    linkToTask () {
      copyText(`${window.location.origin}/task/${this.firstTask.uid}`, undefined, (error, event) => {
        if (error) {
          console.log(error)
        } else {
          console.log(event)
        }
      })
    },
    okToModal () {
      this.$store.commit(USER_VIEWED_MODAL, 'doitnow')
      this.slidesCopy = [...this.slides]
      this.loadAllTasks()
    },
    readTask: function () {
      this.$store.dispatch(TASK.CHANGE_TASK_READ, this.firstTask.uid)
    },
    dateToLabelFormat: function (calendarDate) {
      const day = calendarDate.getDate()
      const month = calendarDate.toLocaleString('default', { month: 'short' })
      const weekday = calendarDate.toLocaleString('default', {
        weekday: 'short'
      })
      return day + ' ' + month + ', ' + weekday
    },
    nextTask: function () {
      if (this.slidesCopy.length && this.justRegistered) {
        this.slidesCopy.shift()
        return
      }
      if (this.notifies.length) {
        this.notifies.shift()
        return
      }
      this.readTask()
      if (this.unreadTasks.length) {
        this.unreadTasks.shift()
        return
      }
      if (this.readyTasks.length) {
        this.readyTasks.shift()
        return
      }
      if (this.overdueTasks.length) {
        this.overdueTasks.shift()
        return
      }
      if (this.todayTasks.length) {
        this.todayTasks.shift()
        return
      }
      if (this.openedTasks.length) {
        this.openedTasks.shift()
      }
    },
    changeValue: function (objWithValues) {
      for (const elem in objWithValues) {
        this.firstTask[elem] = objWithValues[elem]
      }
    },
    goToNextDay: function () {
      const tomorrow = new Date(new Date().setDate(new Date().getDate() + 1))

      this.$store.dispatch('asidePropertiesToggle', false)
      this.$store.commit('basic', { key: 'mainSectionState', value: 'tasks' })
      this.$store.dispatch(TASK.TASKS_REQUEST, tomorrow)
      // hardcoded and messy
      const navElem = {
        name: this.dateToLabelFormat(tomorrow),
        key: 'taskListSource',
        value: { uid: '901841d9-0016-491d-ad66-8ee42d2b496b', param: tomorrow }
      }
      this.$store.commit('updateStackWithInitValue', navElem)
      this.$store.commit('basic', {
        key: 'taskListSource',
        value: { uid: '901841d9-0016-491d-ad66-8ee42d2b496b', param: tomorrow }
      })
      this.$store.commit(TASK.CLEAN_UP_LOADED_TASKS)
    },
    onClickTask: function (task) {
      this.$store.commit('basic', { key: 'propertiesState', value: 'task' })
      this.$store.dispatch(TASK.SELECT_TASK, task)
      this.$store.dispatch('asidePropertiesToggle', true)
    }
  }
}
</script>

<style scoped>
.slide-in-fade-out-enter-from {
  transform: translateX(45px);
}

.slide-in-fade-out-enter-active {
  transition-delay: .4s;
  transition-property: opacity, transform;
  transition-duration: .4s;
}

.slide-in-fade-out-enter-from, .slide-in-fade-out-leave-to {
  opacity: 0;
}

.slide-in-fade-out-leave-active {
  transition-property: opacity, transform;
  transition-duration: .8s;
}

.slide-in-fade-out-leave-to {
  position: absolute;
  width: 100%;
}
</style>

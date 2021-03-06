<template lang="pug">
div#app
  Sidebar
  TaskList
  LoginModal
  TaskDetailModal
  AddFolderModal
  AddTaskModal
  SettingsModal
</template>

<script>
import { mapState, mapActions, mapMutations } from 'vuex'
import initShortCut from './common/event'
import Notification from './common/notification'
import LoginModal from '@/components/LoginModal'
import Sidebar from '@/components/Sidebar'
import TaskList from '@/components/TaskList/'
import TaskDetailModal from '@/components/TaskDetailModal'
import AddFolderModal from '@/components/AddFolderModal'
import AddTaskModal from '@/components/AddTaskModal'
import SettingsModal from '@/components/SettingsModal'
import { ipcRenderer } from 'electron'

const notify = new Notification()

export default {
  name: 'ms-todo',

  components: {
    Sidebar,
    TaskList,
    LoginModal,
    TaskDetailModal,
    AddFolderModal,
    AddTaskModal,
    SettingsModal
  },

  computed: {
    ...mapState([
      'token',
      'theme',
      'tasks',
      'language',
      'currentTask',
      'currentFolder',
      'showCompleteTask',
      'showPlannedFolder',
      'showImportanceFolder'
    ])
  },

  mounted () {
    this.init()
    initShortCut()
    // Set theme if theme setting is auto
    if (this.theme === 'auto') {
      const isDark = ipcRenderer.sendSync('get-theme')
      document.body.setAttribute('data-theme', `theme-${isDark ? 'dark' : 'light'}`)
    }
    // Update data when window focus
    if (process.env.NODE_ENV !== 'development') {
      window.onfocus = () => this.init(false)
    }
    // Update data every 30 minute
    // setInterval(() => {
    //   this.init(false)
    // }, 1000 * 60 * 30)
  },

  beforeDestroy () {
    // Remove all listener before destroy
    ipcRenderer.removeAllListeners()
    // Remove all notification timer
    notify.clear()
  },

  watch: {
    tasks: {
      handler (newValue) {
        notify.create(newValue)
      },
      deep: true
    },
    currentTask: {
      handler (newValue) {
        ipcRenderer.send('update-touchbar', {
          ...this.currentTask,
          showCompleteTask: this.showCompleteTask
        })
      },
      deep: true
    },
    currentFolder: {
      handler (newValue) {
        this.updateState({currentTask: {}})
      },
      deep: true
    },
    theme: {
      handler (newValue) {
        document.body.setAttribute('data-theme', `theme-${newValue}`)
        ipcRenderer.sendSync('update-setting', {theme: newValue})
      },
      immediate: true
    },
    language (newValue) {
      ipcRenderer.sendSync('update-setting', {language: newValue})
    },
    showCompleteTask (newValue) {
      ipcRenderer.sendSync('update-setting', {showCompleteTask: newValue})
    },
    showPlannedFolder (newValue) {
      ipcRenderer.sendSync('update-setting', {showPlannedFolder: newValue})
    },
    showImportanceFolder (newValue) {
      ipcRenderer.sendSync('update-setting', {showImportanceFolder: newValue})
    }
  },

  methods: {
    ...mapMutations({
      updateState: 'UPDATE_STATE'
    }),
    ...mapActions({
      getUserPhoto: 'GET_USER_PHOTO',
      getTaskFolders: 'GET_TASK_FOLDERS',
      getTasks: 'GET_TASKS',
      updateTask: 'UPDATE_TASK'
    }),
    async init (showLoading) {
      let loading
      if (showLoading !== false) {
        loading = this.$loading({
          lock: true,
          text: this.$t('base.loading'),
          background: 'rgba(0, 0, 0, 0.7)'
        })
      }
      try {
        await this.getTaskFolders()
        await this.getTasks()
      } finally {
        loading && loading.close()
      }
    }
  }
}
</script>

<style lang="stylus">
html
body
  *
    font-family Segoe UI,SegoeUI,Segoe WP,Helvetica Neue,Helvetica,Tahoma,Arial,sans-serif

#app
  width 100vw
  height 100vh
  display flex
</style>

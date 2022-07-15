<template>
  <div :key="task.gid" class="task-item" v-on:dblclick="onDbClick">
    <!-- <div class="task-name" :title="taskFullName">
      <span>{{ taskFullName }}</span>
    </div> -->
    <mo-task-item-actions mode="LIST" :task="task" />
    <div class="task-progress">
      <mo-task-progress-info :task="task" />
      <mo-task-progress
        :completed="Number(task.completedLength)"
        :total="Number(task.totalLength)"
        :status="taskStatus"
      />
    </div>
  </div>
</template>

<script>
  import { checkTaskIsSeeder, getTaskName } from '@shared/utils'
  import { TASK_STATUS } from '@shared/constants'
  import { openItem, getTaskFullPath } from '@/utils/native'
  import TaskItemActions from './TaskItemActions'
  import TaskProgress from './TaskProgress'
  import TaskProgressInfo from './TaskProgressInfo'

  export default {
    name: 'mo-task-item',
    components: {
      [TaskItemActions.name]: TaskItemActions,
      [TaskProgress.name]: TaskProgress,
      [TaskProgressInfo.name]: TaskProgressInfo
    },
    props: {
      task: {
        type: Object
      }
    },
    computed: {
      taskFullName () {
        return getTaskName(this.task, {
          defaultName: this.$t('task.get-task-name'),
          maxLen: -1
        })
      },
      taskName () {
        return getTaskName(this.task, {
          defaultName: this.$t('task.get-task-name')
        })
      },
      isSeeder () {
        return checkTaskIsSeeder(this.task)
      },
      taskStatus () {
        const { task, isSeeder } = this
        if (isSeeder) {
          return TASK_STATUS.SEEDING
        } else {
          return task.status
        }
      }
    },
    methods: {
      onDbClick () {
        const { status } = this.task
        const { COMPLETE, WAITING, PAUSED } = TASK_STATUS
        if (status === COMPLETE) {
          this.openTask()
        } else if ([WAITING, PAUSED].includes(status) !== -1) {
          this.toggleTask()
        }
      },
      async openTask () {
        const { taskName } = this
        this.$msg.info(this.$t('task.opening-task-message', { taskName }))
        const fullPath = getTaskFullPath(this.task)
        const result = await openItem(fullPath)
        if (result) {
          this.$msg.error(this.$t('task.file-not-exist'))
        }
      },
      toggleTask () {
        this.$store.dispatch('task/toggleTask', this.task)
      }
    }
  }
</script>

<style lang="scss">
.task-item {
  position: relative;
  padding: 4px 0 0 0;
  background-color: $--task-item-background;
  transition: $--border-transition-base;
  &:hover {
    background: #fafafa;
    .task-item-actions {
      display: block;
    }
    .task-progress-info-right {
      opacity: 0;
    }
  }
  .task-item-actions {
    position: absolute;
    z-index: 2;
    top: 0;
    right: 0;
    display: none;
  }
}
.selected .task-item {
  background: #e0e7ff;
}
.task-name {
  color: #505753;
  word-break: break-all;
  font-weight: bold;
  &> span {
    font-size: 12px;
    overflow : hidden;
    text-overflow: ellipsis;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
  }
}
</style>

<template>
  <div class="notification">
    <v-card>
      <v-card-title>
        <notification-edit
          ref="notificationEdit"
          @pipelineSaved="onSave"
        />
        <v-btn
          class="ma-2"
          @click="onNavigateBack()"
        >
          <v-icon
            dark
            right>
              mdi mdi-arrow-left
          </v-icon>
        </v-btn>
        <v-btn
          class="ma-2"
          color="success"
          @click="onCreateNotification()"
        >
          Add notification
          <v-icon
            dark
            right
          >
            mdi mdi-alarm
          </v-icon>
        </v-btn>
        <v-btn
          class="ma-2"
          @click="onLoadNotifications()"
        >
          <v-icon dark>
            mdi mdi-sync
          </v-icon>
        </v-btn>
      </v-card-title>

      <v-data-table
        :headers="headers"
        :items="selectedPipeline.notifications"
        class="elevation-1"
      >
        <v-progress-linear
          slot="progress"
          indeterminate
        />
        <template v-slot:item.notificationId="{ item }">
          {{ item.notificationId }}
        </template>
        <template v-slot:item.type="{ item }">
          {{ item.type }}
        </template>
        <template v-slot:item.condition="{ item }">
          {{ item.condition }}
        </template>
        <template v-slot:item.action="{ item }">
          <v-btn
            depressed
            small
            class="ma-2"
            @click="onEditNotification(item)"
          >
            Edit
            <v-icon
              dark
              right
            >
              mdi-pencil
            </v-icon>
          </v-btn>
          <v-btn
            depressed
            small
            class="ma-2"
            @click="onDeleteNotification(item)"
          >
            Delete
            <v-icon
              dark
              right
            >
              mdi-delete
            </v-icon>
          </v-btn>
        </template>
      </v-data-table>
    </v-card>
  </div>
</template>

<script lang="ts">
import Component from 'vue-class-component'
import Vue from 'vue'
import { Action, State } from 'vuex-class'
import Pipeline from '@/pipeline/pipeline'
import NotificationConfig from '@/pipeline/notifications/notificationConfig'
import NotificationEditDialog from '@/pipeline/notifications/notificationEditDialog'
import NotificationEdit from '@/pipeline/notifications/NotificationEdit.vue'
import { Ref } from 'vue-property-decorator'

const namespace = { namespace: 'pipeline' }

@Component({
  components: {
    NotificationEdit: NotificationEdit
  }
})
export default class PipelineNotifications extends Vue {
  @Action('loadPipelineById', namespace) private loadPipelineByIdAction!: (id: number) => void

  @Action('addNotification', namespace)
  private addNotificationAction!: (notification: NotificationConfig) => Promise<Pipeline>

  @Action('removeNotification', namespace)
  private removeNotificationAction!: (notification: NotificationConfig) => Promise<Pipeline>

  @Action('updateNotification', namespace)
  private updateNotificationAction!: (notification: NotificationConfig) => Promise<Pipeline>

  @State('selectedPipeline', namespace) private selectedPipeline!: Pipeline

  @Ref('notificationEdit')
  private notificationEdit!: NotificationEditDialog

  private headers = [
    { text: 'Id', value: 'notificationId' },
    { text: 'Type', value: 'type' },
    { text: 'Condition', value: 'condition' },
    { text: 'Actions', value: 'action' }
  ]

  private isEdit = false
  private pipelineId = -1

  private created () {
    console.log('Notification Overview created!')
    this.pipelineId = this.$route.params.pipelineId as unknown as number
    this.loadPipelineByIdAction(this.pipelineId)
  }

  private onCreateNotification () {
    this.isEdit = false
    this.notificationEdit.openDialog()
  }

  private onEditNotification (notification: NotificationConfig) {
    this.isEdit = true
    this.notificationEdit.openDialog(notification)
  }

  private onDeleteNotification (notification: NotificationConfig) {
    this.removeNotificationAction(notification)
  }

  private onLoadNotifications () {
    this.loadPipelineByIdAction(this.pipelineId)
  }

  private onNavigateBack() {
    this.$router.push({ name: 'pipeline-overview' })
  }

  onSave (editedNotification: NotificationConfig) {
    if (this.isEdit) { // edit
      this.updateNotificationAction(editedNotification)
    } else { // create
      this.addNotificationAction(editedNotification)
    }
  }
}
</script>

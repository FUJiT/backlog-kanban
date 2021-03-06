<template>
  <div class="task-list">
    <client-only>
      <draggable class="status-list" group="statusList">
        <section v-for="status in statusList" :key="status.id">
          <TskCardList
            :status="status"
            :task-list="getTaskListByStatus(status.id)"
            class="tsk-card-list"
          />
        </section>
      </draggable>
    </client-only>

    <section class="status-list-add">
      <b-button
        type="is-success"
        class="status-list-add-button"
        @click="showForm()"
        >リストを追加</b-button
      >
      <div v-show="isShowForm" class="status-list-add-form">
        <form @submit.prevent="addStatusList">
          <b-input
            id="statusListName"
            v-model="statusListName"
            v-validate="'required|max:20'"
            name="statusListName"
            maxlength="20"
          ></b-input>
          <div class="status-list-add-form-button">
            <b-button type="is-success" native-type="submit">追加</b-button>
            <b-button type="is-light" @click="hideForm()">キャンセル</b-button>
          </div>
        </form>
      </div>
    </section>
  </div>
</template>

<script lang="ts">
import { Vue, Component } from 'nuxt-property-decorator';
import { statusListInterface } from '~/store/statusList/type';
import { taskInterface } from '~/store/task/type';
import TskCardList from '~/components/organisms/TskCardList.vue';

@Component({
  components: {
    TskCardList,
  },
})
class TskList extends Vue {
  isShowForm: boolean = false;
  statusListName: string = '';

  get statusList(): statusListInterface[] {
    return this.$store.state.statusList.statusList;
  }

  get taskList(): taskInterface[] {
    return this.$store.state.task.taskList;
  }

  /**
   * 状態ごとのタスクを取得
   *
   * @param statusId ステータスID
   */
  getTaskListByStatus(statusId: number): taskInterface[] {
    return this.taskList
      .filter(
        (element: taskInterface) =>
          element.status_id === statusId && !element.isArchive
      )
      .sort((a, b) => b.position - a.position);
  }

  /**
   * ステータスリストを新規追加
   */
  addStatusList(): void {
    this.$validator.validateAll().then((result) => {
      if (result) {
        const statusList: statusListInterface = {
          id: this.$store.getters['statusList/nextId'],
          name: this.statusListName.trim(),
          position: this.$store.getters['statusList/nextPosition'],
        };
        this.$store.dispatch('statusList/asyncAdd', statusList);

        this.hideForm();
      }
    });
  }

  showForm(): void {
    this.isShowForm = true;
  }

  hideForm(): void {
    this.isShowForm = false;
    this.statusListName = '';
  }
}

export default TskList;
</script>

<style lang="scss" scoped>
.task-list {
  display: flex;
  align-items: flex-start;
  margin: 10px 0 0 10px;
}

.status-list {
  display: flex;
}

.status-list-add {
  margin: 5px;

  &-button {
    margin-bottom: 10px;
  }
}

.tsk-card-list {
  margin-right: 10px;
  padding: 10px;
  width: 280px;
}
</style>

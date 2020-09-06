<template>
  <div class="card-group">
    <!-- Контенйнер с названием группы  -->
    <div class="title-container" @click="showDropDown = !showDropDown && groupObj.selectedTasks.length">
      <div class="title">
        <img src="../../../assets/down.png" id="arrow" :class="{'rotate': showDropDown}">
        <span>{{groupObj.title}}</span>
      </div>
      <img src="../../../assets/pencil.png" alt="Edit" id="edit" @click.stop="editGroup()">
      <img src="../../../assets/delete_4219.png" alt="Del" id="delete" @click.stop="deleteGroup()">
    </div>

    <!-- Список задач, входящих в группу -->
    <div class="tasks-list" v-if="showDropDown">
      <!-- Контейнер для строки задачи-->
      <div class="task-container" v-for="task in groupObj.selectedTasks" :key="'task' + task.id">
        <div class="circle" :class="{'successful': task.status}"></div>
        <div class="task">
          {{task.title}}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "CardGroup",
  props: ['groupObj'],
  data() {
    return {
      showDropDown: false
    }
  },
  methods: {
    /* Удалить группу */
    editGroup() {
      this.$emit('editGroup', this.groupObj);
    },
    /* Удалить группу */
    deleteGroup() {
      this.$emit('deleteGroup');
    }
  },
}
</script>

<style lang="scss" scoped>
.card-group {
  display: flex;
  width: 100%;
  height: max-content;
  margin-bottom: 5px;
  flex-direction: column;
  font-size: 14px;
  transition: all 0.3s;
}

.title-container {
  position: relative;
  display: flex;
  align-items: center;

  .title {
    display: flex;
    align-items: center;
    height: 35px;
    width: 100%;
    padding-left: 10px;
    padding-right: 50px;
    background-color: #cdb6bc;
    color: #fff;
    cursor: pointer;

    &:hover {
      background-color: #b58890;
    }

    #arrow {
      height: 15px;
      margin-right: 15px;
      transition: transform .3s;
    }

    .rotate {
      transform: rotateX(180deg);
    }
  }

  #delete, #edit {
    width: 20px;
    height: 20px;
    margin-left: 20px;
    cursor: pointer;
  }
}

.tasks-list {
  width: calc(100% - 90px);
  margin-left: 10px;

  .task-container {
    display: flex;
    align-items: center;
    height: 30px;
    padding-left: 10px;
    padding-right: 15px;
    background-color: #babbce;

    &:not(:last-child) {
      border-bottom: 2px solid #fff;
    }

    .circle {
      width: 10px;
      height: 10px;
      margin-right: 10px;
      border-radius: 50%;
      background-color: #8b1010;
    }

    .successful {
      background-color: #4CAF50;
    }
  }
}
</style>
<template>
  <div class="card-task">
    <!-- Вверхняя часть карточки-->
    <div class="header">
      <!-- Название карточки -->
      <div class="title-container">
        <div class="task-title">
          <span>{{taskObj.title ? taskObj.title : 'Без названия'}}</span>
        </div>
        <div class="group-title">
          <span>{{taskObj.group.title ? taskObj.group.title : 'Без группы'}}</span>
        </div>
      </div>
    </div>
    <!-- Блок с котентом -->
    <div class="content-container">
      <!-- Описание -->
      <div class="description">
        <span>Описание:</span>
        <textarea v-model="taskObj.description"></textarea>
      </div>
      <div class="status-btn" :class="{'successful': taskObj.status}" @click="changeStatus()">
        <span v-if="taskObj.status">Выполнено</span>
        <span v-else>Не выполнено</span>
      </div>
    </div>

    <div class="delete-btn" @click="deleteTask()">
      <img src="../../../assets/delete_4219.png" alt="Del">
    </div>
  </div>
</template>

<script>
export default {
  name: "CardTask",
  props: ['taskObj'],
  methods: {
    // Удалить задачу
    deleteTask() {
      this.$emit('deleteTask');
    },
    // Изменить статус
    changeStatus() {
      this.$emit('changeStatus');
    }
  }
}
</script>

<style lang="scss" scoped>

@mixin flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

@mixin overflow-block {
  display: block;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

$pepper: #AF4425;
$cinnamon: #662E1C;
$cream: #EBDCB2;
$caramel: #c9a66b;
$successful: #4CAF50;
.card-task {
  display: flex;
  flex-direction: column;
  width: 270px;
  height: 330px;
  border: 2px solid $caramel;
  transition: all 0.3s ease;
  background-color: $cream;
  color: #08090b;
  margin-right: 50px;
  margin-bottom: 50px;
  transition: all 0.3s;

  &:hover {
    border: 2px solid #f3b366;
    transform: scale(1.01);
  }

  .title-container {

    .task-title, .group-title  {
      @include overflow-block;
      padding: 5px 10px;
    }

    .task-title {
      font-size: 16px;

    }

    .group-title {
      font-size: 12px;
      border-bottom: 1px solid $caramel;
    }
  }

  .content-container {
    display: flex;
    flex: 1;
    padding: 10px 10px;
    flex-direction: column;

    .description {
      font-size: 14px;
      display: flex;
      flex-direction: column;

      textarea {
        padding: 5px;
        height: 150px;
        resize: none;
        background-color: #f8e8bc;
        border-color: $caramel;
        outline : none;
        overflow:auto;
      }

    }

    .status-btn {
      @include flex-center;
      align-self: center;
      height: 25px;
      width: 50%;
      margin-top: 20px;
      border-radius: 5px;
      background-color: #bfbfb6;
      cursor: pointer;
      font-size: 14px;
    }

    .successful {
      background-color: $successful;
      color: #fff;
    }
  }

  .delete-btn {
    margin-left: auto;
    margin-right: 5px;
    cursor: pointer;
    img {
      width: 25px;
    }
  }



}

</style>
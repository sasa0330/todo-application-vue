<template>
  <div id="app">
    <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.14.0/css/all.css"
      integrity="sha384-HzLeBuhoNPvSl5KYnjx0BT+WB0QEEqLprO+NBkkk5gbc67FTaL7XIGa2w1L0Xbgc" crossorigin="anonymous" />
    <div v-if="isOpenModal" class="modal">
      <div class="modal__back_layer" @click="editCalcel()"></div>
      <div class="modal__main_layer">
        <div class="add_todo">
          <textarea type="text" v-model="inputTodoText" placeholder="edit me" class="add_todo__input" id="inputTodo" />
          <div class="add_todo__btn" v-if="isEditIndex == -1">
              <div @click="editCalcel()" class="add_todo__btn--cancel">キャンセル</div>
              <div @click="addItem('add')" class="add_todo__btn--add">追加</div>
          </div>
          <div class="add_todo__btn" v-if="isEditIndex > -1">
              <div @click="editCalcel()" class="add_todo__btn--cancel">編集キャンセル</div>
              <div @click="editItem(index, 'confirm')" class="add_todo__btn--add">編集</div>
          </div>
        </div>
      </div>
    </div>
    <div class="modal_btn" v-if="!isOpenModal">
      <div @click="isOpenModal = true" class="modal_btn__text">＋</div>
    </div>  
    <div class="todo_list todo_list--empty" v-show="!isExistTodo">
      右下から追加してね
    </div>
    <div class="todo_list todo_list--exist" v-show="isExistTodo">
      <div v-for="(todo, index) in todos" :key="index" class="todo_list__contents">
        <div class="todo_text">
          <div class="todo_text__wrap_text" @click="doneItem(index)">
            <span class="todo_text__text" v-if="!todo.isDone">{{ todo.item }}</span>
            <span class="todo_text__text--done" v-if="todo.isDone" style="text-decoration: line-through;">{{ todo.item }}</span>
          </div>
        </div>
        <div class="todo_btn">
          <div class="todo_btn__left" @click="deleteItem(index)">
              <i class="fas fa-trash-alt"></i>
          </div>
          <div class="todo_btn__right" @click="editItem(index, 'edit')">
              <i class="fas fa-pencil-alt"></i>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data: () => {
    return {
      todos: [],  //TODOリスト
      inputTodoText: "",  //TODOを追加するときのテキスト
      isEmptyInputTodoText: true,  //TODOの入力状態監視用
      isOpenModal: false,  //TODO入力モーダル開閉用
      isEditIndex: -1 //編集中TODOのIndex。何もしていないときは-1
    };
  },
  created: function () {
    let jsonObj = localStorage.getItem("todosKey");
    let jsObj = JSON.parse(jsonObj);
    for (let i in jsObj) {
      let todo = {
        item: jsObj[i].item,
        isDone: jsObj[i].isDone,
      };
      this.todos.push(todo);
    }
  },
  computed:{
    //todoリストの中身があればリスト部分を表示
    isExistTodo:function(){
      return this.todos.length > 0;
    }
  },
  watch: {
    //addItem()したときにテキストが空かを監視する
    inputTodoText: function () {
        //空ならtrue
        this.isEmptyInputTodoText = this.inputTodoText == "";
    }
  },
  methods: {
    /**
     * TODO追加
     * @param {string} addState - 編集ボタンの状態
     */
    addItem(addState) {
      //TODO追加キャンセル時
      if(addState == "cancel"){
        this.isOpenModal = false;
        return false;
      }
      //テキストボックスに何も値がないときは追加させない
      if(this.isEmptyInputTodoText){
        return false;
      }
      let todo = {
        item: this.inputTodoText,
        isDone: false
      };
      this.todos.push(todo);
      this.inputLocalStrage();
      this.inputTodoText = "";
      this.isOpenModal = false;
    },
    /**
     * TODO削除
     * @param {number} index - 削除するTODOのindex
     */
    deleteItem(index) {
      this.todos.splice(index, 1);
      this.inputLocalStrage();
    },
    /**
     * TODO編集
     * @param {number} index - 編集中のTODOのindex
     * @param {string} editState - 編集ボタンの状態
     */
    editItem(index, editState) {
      switch (editState) {
        case "edit": {
          this.isOpenModal = true;
          this.isEditIndex = index;
          this.inputTodoText = this.todos[index].item;
          break;
        }
        case "confirm": {
          this.todos[this.isEditIndex].item = this.inputTodoText;
          this.editCalcel();
          break;
        }
       }
      this.inputLocalStrage();
    },
    /**
     *実行済みTODOの打消し線
     * @param {number} index - 打ち消すTODOのindex
     */
    doneItem(index) {
      if (this.todos[index].isDone) {
        this.todos[index].isDone = false;
      } else {
        this.todos[index].isDone = true;
      }
      this.inputLocalStrage();
    },
    /**
     *追加や編集を辞めて初期化するメソッド
     */
    editCalcel(){
      //やりたいこと：テキスト入力されていたら、警告したい。
      this.isOpenModal = false;
      this.inputTodoText = "";
      this.isEditIndex = -1;
    },
    /**
     * ローカルストレージにデータ格納
     */
    inputLocalStrage() {
      let obj = JSON.stringify(this.todos);
      localStorage.setItem("todosKey", obj);
    },
  },
};
</script>

<style lang="scss">
.reset-btn-style {
  background-color: transparent;
  border: none;
  cursor: pointer;
  outline: none;
  padding: 0;
  appearance: none;
}

.base-input-text {
  font-size: 17px;
}

.main-title {
  text-align: center;
}

/* モーダル */
.modal{
    position: fixed;
    width: 100vw;
    height: 100vh;
    top: 0;
    left: 0;
  &__back_layer {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background-color: #DDDDDD33;
  }
  &__main_layer {
    position: fixed;
    width: calc(100vw - 20px);
    height: 40vh;
    top: 60vh;
    left: 0;
    padding: 10px;
    background-color: #FFF;
  }
}

.modal_btn{
  position: fixed;
  bottom: 10px;
  right: 10px;
  border-radius: 50%;
  border: solid 1px #DDD;
  background-color: #FFF;
  width: 50px;
  height: 50px;
  &__text{
    display: flex;
    align-items: center;
    justify-content: center;
    height: 100%;
    width: 100%;
  }
}

/* モーダルの中身 */
.add_todo {
  height: 40px;
  &__input {
    @extend .base-input-text;
    width: 100%;
    height: 100px;
    padding: 3px;
    margin-bottom: 30px;
    display: block;
    border: solid 1px #DDD;
  }
  &__btn {
    @extend .reset-btn-style;
    width: 100%;
    height: 40px;
    display: flex;
    justify-content: space-between;
    &--add{
      width: 45%;
      height: 40px;
      display: flex;
      justify-content: center;
      align-items: center;
      background: #76ff03;
      color: #fff;
      font-weight: bold;
    }
    &--cancel{
      width: 45%;
      height: 40px;
      display: flex;
      justify-content: center;
      align-items: center;
      background: #DDD;
      color: #000;
      font-weight: bold;
    }
    &:active {
      /*ボタンを押したとき*/
      border: none;
      -webkit-transform: translate(3px, 3px);
      transform: translate(3px, 3px);
    }
  }
}

/* TODO一覧部分 */
.todo_list {
  padding: 0 10px;
  width: calc(100vw - 20px);
  &__contents {
    border-bottom: solid 1px #999;
    padding: 10px;
    display: flex;
  }
  &--empty{
    height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
  }
}

.todo_text {
  width: 80%;
  word-break: break-all;
  &-wrap_text {
    width: 100%;
    display: block;
    &--done_check:checked + &__text {
      text-decoration: line-through;
    }
  }

  .input-todo-edit {
    @extend .base-input-text;
    width: 90%;
    height: 40px;
  }
}

.todo_btn {
  width: 20%;
  display: flex;
  justify-content: space-between;
  align-items: center;

  &__left {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 30px;
    height: 30px;
    border: solid 1px #999;
    border-radius: 5px;
  }
  &__right {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 30px;
    height: 30px;
    border: solid 1px #999;
    border-radius: 5px;
  }
}
</style>
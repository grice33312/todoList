<script setup>
import {ref , computed, onMounted , } from "vue" ;
  const newTask = ref("");
  
  const todoList = ref([
    {id : 202406111200 , taskName : "這是一個新任務" , completed : false , isEdit : false} ,
    {id : 202406111204 , taskName : "請添加新任務" , completed : false , isEdit : false}
  ])

  const addToList = ()=>{
    const timeStamp = Math.floor(Date.now());
    const name = newTask.value.trim();
    if(name){
      todoList.value.push({id : timeStamp , taskName : name , completed : false})
    };
    newTask.value = "" ;
  }

  const deleteTask = (index)=>{
    todoList.value.splice(index , 1) ;
  }

  const headerTabs = ref([
    {value:"allItem" , name : "全部"},
    {value:"processing" , name : "進行中"},
    {value:"done" , name:"已完成"}
  ])

  const visibilityTab = ref("allItem");

  const setVisibilityTab = (value)=>{
    visibilityTab.value = value ;
  }

  const filteredList = computed(()=>{
    let allTab = headerTabs.value[0].value ; 
    let processingTab = headerTabs.value[1].value ; 
    if(visibilityTab.value === allTab){
      return todoList.value
    }else if(visibilityTab.value === processingTab){
      return todoList.value.filter((item)=>{
        return !item.completed
      })
    }else{
      return todoList.value.filter((item)=>{
        return item.completed
      })
    }
  })
  const cacheTaskName = ref("");
  
  const cacheTask = ref({});

  const editTask = (item)=>{
    item.isEdit =  true;
    cacheTaskName.value = item.taskName ;
    cacheTask.value = item ;
  }

  const deleteAllTask = ()=>{
    todoList.value = []
  }

  const cancelEdit = (item)=>{
    item.isEdit = false ;
    cacheTask.value = {};
  }

  const doneEdit = (item)=>{
    item.isEdit = false ; 
    item.taskName =  cacheTaskName.value; 
    cacheTask.value = {};
    cacheTaskName.value = "";
  }
  
  const lostFocus = (item)=>{
    if(item.isEdit){
      doneEdit(item)
    }
  }
  
  const vFocus = {
    mounted:(el)=>el.focus()
  }

  const uncompletedPrompt = computed(()=>{
    let message = "現在沒有任務唷" ;
    let counter = 0 ; 
    if(todoList.value.length <= 0){
      return
    };
    todoList.value.forEach((item)=>{
      counter = !item.completed ? counter + 1 : counter ;
    });
    if(counter > 0){
      message = `現在還有 ${counter} 筆任務未完成，繼續加油! ` 
    }else{
      message = "完工了"
    }
    return message
  })  


</script>

<template>
    <div class="container my-3">
      <div class="input-group mb-3">
          <span class="input-group-text" id="basic-addon1">待辦事項</span>
          <input type="text" placeholder="準備要做的任務" class="form-control" v-model="newTask" @keyup.enter="addToList">
          <button type="button" class="btn btn-primary" @click="addToList">新增</button>
      </div>
    </div>
  <div class="card text-center">
    <div class="card-header" >
      <ul class="nav nav-tabs card-header-tabs" >
        <li class="nav-item" v-for="(item , index) in headerTabs">
          <a href="#" class="nav-link" :class="{active: visibilityTab === item.value}" @click="setVisibilityTab(item.value)">{{item.name}}</a>
        </li>
      </ul>
    </div>
    <ul class="list-group list-group-flush text-left">
      <li class="list-group-item" v-for="(item , index) in filteredList">
        <div class="d-flex justify-content-between my-1" @dblclick="editTask(item)" v-if="!item.isEdit">
          <div class="form-check" >
            <input type="checkbox" class="form-check-input" :id="item.id" v-model="item.completed" >
            <label :for="item.id"  class="form-check-label" :class="{completed:item.completed}" >
                {{ item.taskName }}
            </label>
          </div >
          <button type="button" class="btn-close  aria-label=Close" @click="deleteTask(index)">
          </button>
        </div >
        <input type="text" class="form-control" v-focus v-else v-model="cacheTaskName" @keyup.enter="doneEdit(item)" @keyup.esc="cancelEdit(item)" @blur="lostFocus(item)" >
      </li>
    </ul>
    <div class="card-footer d-flex justify-content-between">
      <span>{{uncompletedPrompt}}</span>
      <a href="#" @click="deleteAllTask">清除所有任務</a>
    </div>
  </div>
</template>

<style scoped>
  .completed{
    text-decoration: line-through;
  }
</style>
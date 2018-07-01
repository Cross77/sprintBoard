<template>
  <div>
    <nav class="navbar navbar-inverse">
      <div class="container-fluid">
        <!-- Brand and toggle get grouped for better mobile display -->
        <div class="navbar-header">
          <a class="navbar-brand" href="#">SprintBoard</a>
        </div>

        <!-- Collect the nav links, forms, and other content for toggling -->
        <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
          <ul class="nav navbar-nav">
            <li><a href="#" @click="create">Create task</a></li>
            <li><a href="#" @click="load">Load board</a></li>
            <li><a href="#" @click="save">Save board</a></li>
            <li><a href="#" @click="remove">Format board</a></li>
            <li><a href="#" @click="orderList">Order list</a></li>
            <li><a href="#" @click="editable=!editable"><span v-if="editable">Block editable</span> <span v-else>Blocked</span></a></li>
            <li><a href="#" @click="debug=!debug"><span v-if="debug">Active debug</span> <span v-else>Deactived debug</span></a></li>
          </ul>
        </div><!-- /.navbar-collapse -->
      </div><!-- /.container-fluid -->
    </nav>
    <div class="container-fluid">
      <div class="row">
        <div class="col-md-8">

          <div class="row">
            <div class="col-md-4 todo-list">
              <h1>
                TODO
                <span class="badge">{{ list.length }}</span>
                <span class="badge">{{ count1 }}</span>
              </h1>
                <draggable class="list-group" element="ul" v-model="list" :options="dragOptions" :move="onMove" @start="isDragging=true" @end="isDragging=false"> 
                  <transition-group type="transition" :name="'flip-list'">
                    <li class="list-group-item" v-for="element in list" :key="element.order" @click="active = element" :class="{active: element == active}">
                      <i :class="element.fixed? 'fa fa-anchor' : 'glyphicon glyphicon-pushpin'" @click=" element.fixed=! element.fixed" aria-hidden="true"></i>
                      {{element.name}}
                      <span class="badge">{{element.storyPoint}}</span>
                    </li> 
                  </transition-group>
              </draggable>
            </div>

            <div class="col-md-4 inProgress-list">
              <h1>
                IN PROGRESS
                <span class="badge">{{ list2.length }}</span>
                <span class="badge">{{ count2 }}</span>
              </h1>
              <draggable element="span" v-model="list2" :options="dragOptions" :move="onMove"> 
                  <transition-group name="no" class="list-group" tag="ul">
                    <li class="list-group-item" v-for="element in list2" :key="element.order" @click="active = element" :class="{active: element == active}"> 
                      <i :class="element.fixed? 'fa fa-anchor' : 'glyphicon glyphicon-pushpin'" @click=" element.fixed=! element.fixed" aria-hidden="true"></i>
                      {{element.name}}
                      <span class="badge">{{element.storyPoint}}</span>
                    </li> 
                  </transition-group>
              </draggable>
            </div>

            <div class="col-md-4 done-list">
              <h1>
                DONE
                <span class="badge">{{ list3.length }}</span>
                <span class="badge">{{ count3 }}</span>
              </h1>
              <draggable element="span" v-model="list3" :options="dragOptions" :move="onMove"> 
                  <transition-group name="no" class="list-group" tag="ul">
                    <li class="list-group-item" v-for="element in list3" :key="element.order" @click="active = element" :class="{active: element == active}"> 
                      <i :class="element.fixed? 'fa fa-anchor' : 'glyphicon glyphicon-pushpin'" @click=" element.fixed=! element.fixed" aria-hidden="true"></i>
                      {{element.name}}
                      <span class="badge label-warning">{{element.storyPoint}}</span>
                    </li> 
                  </transition-group>
              </draggable>
            </div>
          </div>

          <div class="row" v-if="debug">
            <div  class="list-group col-md-4">
              <pre>{{listString}}</pre>
            </div>
            <div  class="list-group col-md-4">
              <pre>{{list2String}}</pre>
            </div>
            <div  class="list-group col-md-4">
              <pre>{{list3String}}</pre>
            </div>
          </div>
        </div>
        
        <div class="col-md-4">
          
            <div class="form-group form-group-lg panel panel-default">
              <div class="panel-heading">
                <h3 class="panel-title">Task details</h3>
              </div>
              <div class="panel-body" v-if="active != -1">
                <h6>Task name</h6> <input type="text" v-model="active.name" class="form-control">
                <h6>Storypoint</h6> <input type="number" v-model="active.storyPoint" class="form-control">
                <h6>Description</h6>
                <textarea v-model="active.description" class="form-control"></textarea> 
                <hr>
                <button type="button" class="btn btn-danger" @click="removeItem(active)">Remove</button>
                <button type="button" class="btn btn-default" @click="active = -1">Close</button>
              </div>
              <div class="panel-body" v-else>
                Select task
              </div>
            </div>

        </div>
      </div>
    </div>
  </div>
</template>

<script>
import draggable from 'vuedraggable'
const message = [ 'vue.draggable', 'draggable', 'component', 'for', 'vue.js 2.0', 'based' , 'on', 'Sortablejs' ]

export default {
  name: 'hello',
  components: {
    draggable,
  },
  data () {
    return {
      //list: message.map( (name,index) => {return {name, order: index+1, fixed: false, description: '', storyPoint: 0}; }),
      list: [],
      list2:[],
      list3: [],
      active: -1,
      editable:true,
      isDragging: false,
      delayedDragging:false,
      debug: false
    }
  },
  mounted() {
    if(localStorage.getItem('board')) this.load();
  },
  methods:{
    create(){
      this.list.push({name: 'New ticket', order: this.count + 1, fixed: false, description: '', storyPoint: 1});
    },
    load(){
      try{
        const data = JSON.parse(localStorage.getItem('board'));
        this.list = data.todo;
        this.list2 = data.inProgress;
        this.list3 = data.done;
      }catch(e){
        alert('Error. Open console');
        console.log('error', e);
      }
    },
    removeItem(item){
      this.active = -1;
      var idx = -1;
      idx = this.list.findIndex(e => e==item)
      if(idx != -1){
        this.list.splice(idx, 1);
      }else{
        idx = this.list2.findIndex(e => e==item)
        if(idx != -1){
          this.list2.splice(idx, 1);
        }else{
          idx = this.list3.findIndex(e => e==item)
          if(idx != -1){
            this.list3.splice(idx, 1);
          }else{
            alert('ERROR');
          }
        }
      }
    },
    remove(){
      localStorage.removeItem('board');
      this.list = [];
      this.list2 = [];
      this.list3 = [];
      alert('Removed!');
    },
    save(){
      localStorage.setItem('board', JSON.stringify({
        todo: this.list,
        inProgress: this.list2,
        done: this.list3
      }));
      alert('Saved!');
    },
    orderList () {
      this.list = this.list.sort((one,two) =>{return one.order-two.order; })
    },
    onMove ({relatedContext, draggedContext}) {
      const relatedElement = relatedContext.element;
      const draggedElement = draggedContext.element;
      return (!relatedElement || !relatedElement.fixed) && !draggedElement.fixed
    }
  },
  computed: {
    count1(){
      var i = 0;
      this.list.forEach(e => {
        i += Number(e.storyPoint);
      });
      return i;
    },
    count2(){
      var i = 0;
      this.list2.forEach(e => {
        i += Number(e.storyPoint);
      });
      return i;
    },
    count3(){
      var i = 0;
      this.list3.forEach(e => {
        i += Number(e.storyPoint);
      });
      return i;
    },
    count(){
      return this.list.length + this.list2.length + this.list3.length;
    },
    dragOptions () {
      return  {
        animation: 0,
        group: 'description',
        disabled: !this.editable,
        ghostClass: 'ghost'
      };
    },
    listString(){
      return JSON.stringify(this.list, null, 2);  
    },
    list2String(){
      return JSON.stringify(this.list2, null, 2);  
    },
    list3String(){
      return JSON.stringify(this.list3, null, 2);  
    }
  },
  watch: {
    isDragging (newValue) {
      if (newValue){
        this.delayedDragging= true
        return
      }
      this.$nextTick( () =>{
           this.delayedDragging =false
      })
    }
  }
}
</script>
<style>
body{
  font-family: Roboto, -apple-system, BlinkMacSystemFont, 'Segoe UI', Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  background-image: url('https://wallpapercave.com/wp/zwaIWUh.jpg');
  background-size: cover;
}
.flip-list-move {
  transition: transform 0.5s;
}

.no-move {
  transition: transform 0s;
}

.ghost {
  opacity: .5;
  background: #C8EBFB;
}

.list-group {
  min-height: 20px;
  border: 3px solid rgba(255,255,255,0.4);
  border-radius: 5px;
}

.list-group-item {
  cursor: move;
}

.list-group-item i{
  cursor: pointer;
}
h1{
  color: #fff !important;
  font-family: Roboto !important;
  font-weight: 100 !important;
}
h6{
  color: #999 !important;
}
h5{
  color: #fff !important;
}
.todo-list .list-group-item{
  background-color: #999;
  color: #fff;
}
.inProgress-list .list-group-item{
  background-color: #3f51b5;
  color: #fff;
}
.done-list .list-group-item{
  background-color: #4CAF50;
  color: #fff;
}
.list-group-item.active, .list-group-item.active:hover, .list-group-item.active:focus {
    color: #333 !important;
    background-color: #fff !important;
    border-color: #fff !important;
}
</style>

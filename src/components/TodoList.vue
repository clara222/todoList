<template>
  <v-app>
    <v-layout row>
      <v-flex xs12 sm6 offset-sm3>
        <v-card>
          <v-toolbar color="teal" dark>
            <v-toolbar-title>{{todoList.length}}个任务，{{doneListNum}}个已完成</v-toolbar-title>
          </v-toolbar>
          <!--list-->
          <v-list subheader>
            <v-list-tile avatar v-for="(item,index) in todoList" :key="index">
              <v-list-tile-action>
                <v-checkbox v-model="item.isComplete"  color="grey darken-1"></v-checkbox>
              </v-list-tile-action>
              <v-list-tile-content>
                <v-list-tile-title :style="{'textDecoration': item.isComplete?'line-through':'none'}">{{item.ListName}}</v-list-tile-title>
              </v-list-tile-content>
              <v-list-tile-action>
                <v-btn icon ripple @click.native.stop="onEditBtnIsClicked(item.ListName,index)">
                  <v-icon color="grey lighten-1">more_vert</v-icon>
                </v-btn>
              </v-list-tile-action>
            </v-list-tile>
          </v-list>
          <v-btn fab dark color="indigo" @click.native.stop="onAddBtnIsClicked()">
            <v-icon dark>add</v-icon>
          </v-btn>
          <!--dialog-->
          <v-dialog v-model="dialog" max-width="460">
            <v-card>
              <v-card-title class="headline">任务编辑</v-card-title>
                <v-text-field
                  v-model="userInput"
                  style="padding: 20px;"
                  placeholder="这里是任务内容的编辑区，请注意字数的限制."
                ></v-text-field>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="red darken-1" flat="flat" @click.native="onDelBtnIsClicked()" v-if="del">删除</v-btn>
                <v-btn color="green darken-1" flat="flat" @click.native="onSaveBtnIsClicked()">保存</v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-card>
      </v-flex>
    </v-layout>
  </v-app>
</template>

<script>
  import axios from 'axios'
  import {mapGetters} from 'vuex'
export default {
  name: 'todolist',
  data () {
    return {
      todoList: [],
      dialog: false,
      userInput: '',
      del: true,
      doneListNum: 0,
      idx: 0
    }
  },
  created () {
    this.getListData()
  },
  computed: {
    ...mapGetters(['todoLists','doneTodoList'])
  },
  watch: {
    'todoLists': function (n ,o) {
      this.todoList = n
    },
    'doneTodoList': function (n, o) {
      this.doneListNum = n.length
    },
    'dialog': function (n, o) {
      if (!n) {
        this.userInput = ''
      }
    }
  },
  methods: {
    onAddBtnIsClicked () {
      this.del = false
      this.dialog = true
    },
    onEditBtnIsClicked (v,index) {
      this.del = true
      this.dialog = true
      this.userInput = v
      this.idx = index
    },
    onSaveBtnIsClicked () {
      this.dialog = false
      if (this.userInput !== '') {
      	if (!this.del) {
      		var list = {
            "ListName": this.userInput,
            "isComplete": false
          }
          this.$store.dispatch('addList', list)
        } else {
      		var editItem = {
      			index: this.idx,
            ListName: this.userInput
          }
          this.$store.dispatch('editList', editItem)
        }
      } else {
      	alert('输入不能为空！')
      }
    },
    onDelBtnIsClicked () {
      this.dialog = false
      this.$store.dispatch('deleteList', this.idx)
    },
    async getListData () {
    	try {
    		var res = await axios.get('/static/json/list.json')
        this.$store.dispatch('changeList', res.data.result)
      }catch (err) {
    		console.log(err)
      }
    }
  }

}
</script>


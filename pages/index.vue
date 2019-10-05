<template>
  <div class="container">
    <div>
      <div v-if="!playing">
        <button @click="loadGame(game)" v-for="game in games" :key="game.id">Game:  {{ game.id }} </button @click="loadGame(game)">
      </div>
<hr>
<!--<button @click="createGame">Create Game</button>-->
<div v-if="!playing" class="d-flex flex-column">
  <div>
    <label>Rows</label>
    <input type="text" v-model="rows" placeholder="rows">
  </div>
  <div>
    <label>Cols</label>
    <input type="text" v-model="cols" placeholder="cols">
  </div>
  <div>
    <label>Mines</label>
    <input type="text" v-model="mines" placeholder="mines">
  </div>
  <button  @click="createLoadGame">Create Game</button>
</div>
<button v-if="playing" @click="closeGame">Close</button>


<div v-if="playing" class="d-flex">
<div v-if="false">
  <pre >{{game}}</pre>
</div>
<div>

<h1 v-if="gameover">GAME OVER</h1>
<h1 v-if="win">YOU WIN</h1>
<h4 v-if="game.time">{{game.time}} seconds</h4>

<div class="d-flex flex-column" >
  <div class="d-flex" v-for="(row,key) in grid" :key="key">
    <div  class="cell" :class="{revealed : col.revealed, boom : col.revealed && col.mine}" v-for="(col,key) in row" :key="key"
      @contextmenu.prevent="flag(col)"
      @click="reaveal(col)"
    >

      <b v-if="col.mine && col.revealed">X</b> <span v-if="col.around">{{col.around}}</span> <small>{{col.mark}}</small>
    </div>
  </div>
</div>

</div>
</div>
    </div>
  </div>
</template>

<script>
import Logo from '~/components/Logo.vue'

export default {
  components: {
    Logo
  },
  data(){
    return {
      cols:10,
      rows:10,
      mines:10,
      games:[],
      game:null,
      grid:[],
      gameover:false,
      win:false,
      playing:false
    }
  },
  mounted(){
   /* this.$axios.get('http://127.0.0.1:8000/api/test')
    this.$axios.post('http://127.0.0.1:8000/api/test')
    this.$axios.put('http://127.0.0.1:8000/api/test')
    this.$axios.delete('http://127.0.0.1:8000/api/test')*/
    this.loadGames();
  },
  methods:{
    closeGame(){
      //stop timer
      this.playing = false
      this.$axios.get('http://127.0.0.1:8000/api/game/close/'+this.game.id).then( ({data}) => {

      })
    },
    flag(cell) {
      if(this.gameover || this.win) return
        this.$axios.get('http://127.0.0.1:8000/api/game/flag/'+cell.id).then( ({data}) => {
        this.grid[data.x][data.y] = data;
      })
     },
    reaveal(cell){
      if(this.gameover || this.win) return
      this.$axios.get('http://127.0.0.1:8000/api/game/reveal/'+cell.id).then( ({data}) => {
        this.grid = data.grid;
        if(data.status == 'boom') this.gameover = true;
        if(data.iwin) this.win = true;
      })

    },
    loadGames(){
      this.$axios.get('http://127.0.0.1:8000/api/games').then( ({data}) => {
        this.games = data;
      })
    },
    loadGame(game){
      this.$axios.get('http://127.0.0.1:8000/api/game/'+game.id).then( ({data}) => {
      this.game = data.game;
      this.grid = data.grid;
      this.gameover = data.game.result === 'loose'
      this.win = data.game.result === 'win'
      this.playing = true;
    })
    },
    createGame(){
      this.$axios.post('http://127.0.0.1:8000/api/game/create',{
        mines:this.mines,
        cols:this.cols,
        rows:this.rows
      }).then( ({data}) => {
        this.loadGames();
       this.game = data
      })
    },
    createLoadGame(){
      this.$axios.post('http://127.0.0.1:8000/api/game/create-n-load',{
        mines:this.mines,
        cols:this.cols,
        rows:this.rows
      }).then( ({data}) => {
        this.loadGames();
        this.game = data.game;
        this.grid = data.grid;
        this.gameover = false
        this.playing = true;
      })
    }
  }
}
</script>

<style>
.revealed{
  background: #acafb3;
}
.boom{
  background: red;
}
.cell{
  border: 1px solid #000;
  width: 50px;
  height: 50px;
}
.d-flex{
  display: flex;
}
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>

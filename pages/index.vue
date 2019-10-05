<template>
  <div class="container">
    <div>
      <div>
        <button @click="loadGame(game)" v-for="game in games" :key="game.id">Game:  {{ game.id }} </button @click="loadGame(game)">
      </div>
<hr>
<button @click="createGame">Create Game</button>
<button @click="createLoadGame">Create N Load Game</button>
<pre>{{game}}</pre>

<h1 v-if="gameover">GAME OVER</h1>

<div class="d-flex flex-column">
  <div class="d-flex" v-for="(row,key) in grid" :key="key">
    <div  class="cell" :class="{revealed : col.revealed}" v-for="(col,key) in row" :key="key"
      @contextmenu.prevent="flag(col)"
      @click="reaveal(col)"
    >

      <b v-if="col.mine">X</b> {{col.around}} <small>{{col.mark}}</small>
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
      games:[],
      game:null,
      grid:[],
      gameover:false
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
    flag(cell) {
      if(this.gameover) return
        this.$axios.get('http://127.0.0.1:8000/api/game/flag/'+cell.id).then( ({data}) => {
        this.grid[data.x][data.y] = data;
      })
     },
    reaveal(cell){
      if(this.gameover) return
      this.$axios.get('http://127.0.0.1:8000/api/game/reveal/'+cell.id).then( ({data}) => {
        this.grid = data.grid;
        if(data.status == 'boom') this.gameover = true;
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
    })
    },
    createGame(){
      this.$axios.post('http://127.0.0.1:8000/api/game/create',{
        mines:20,
        cols:10,
        rows:10
      }).then( ({data}) => {
        this.loadGames();
       this.game = data
      })
    },
    createLoadGame(){
      this.$axios.post('http://127.0.0.1:8000/api/game/create-n-load',{
        mines:20,
        cols:10,
        rows:10
      }).then( ({data}) => {
        this.loadGames();
        this.game = data.game;
        this.grid = data.grid;
      })
    }
  }
}
</script>

<style>
.revealed{
  background: #acafb3;
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

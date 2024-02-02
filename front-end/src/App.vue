<template>
  <div class="app">
    <audio controls class="audio IWIN">
      <source src="./assets/win.mp3" type="audio/mpeg">
    </audio>
    <audio controls class="audio ILOSE">
      <source src="./assets/lose.mp3" type="audio/mpeg">
    </audio>
    <header>Genieh Game</header>
    <div class="message" v-if="showAlert">
      <div class="content">{{ this.messaage }}</div>
      <div class="ok" @click="reset, showAlert = false, stopMusic">Ok, To Main Menu</div>
    </div>
    <div class="controls" v-if="controls">
      <div class="up key" @click="moveUp"> up </div>
      <div class="do">
        <div class="left key" @click="moveLeft"> left </div>
        <div class="down key" @click="moveDown"> down </div>
        <div class="right key" @click="moveRight"> right </div>
      </div>
    </div>
    <div class="mainMenu" v-if="mainMenu">
      <div class="survive mode" @click="setMode('survive')">Survive Mode</div>
      <div class="collect mode">
        <div @click="setMode('collect')">Collect Mode</div>
        <select name="Level" id="level" v-model="level">
          <option value="easy">Easy</option>
          <option value="medium">Medium</option>
          <option value="hard">Hard</option>
        </select>
      </div>
    </div>
    <div class="board" v-if="win && mode=='survive'">
      <div class="row" v-for="(row, i) in board" :key="row">
        <div class="col" v-for="(col, j) in row" :key="col">
          <div class="block" v-if="board[j][i] == 0"></div>
          <div class="way" v-if="board[j][i] == 1">
            <img src="./assets/coin.png" class="player" v-if="player[0] == j && player[1] == i">
            <img src="./assets/dolar.jpeg" class="player" v-if="checkEvil(j,i)">
            <img src="./assets/start.png" class="star" v-if="j == start[0] && i == start[1]">
            <img src="./assets/end.png" class="star" v-if="j == end[0] && i == end[1]">
          </div>
        </div>
      </div>
    </div>
    <div class="score" v-if="win && mode=='collect'">Your Score is {{ score }}</div>
    <div class="board" v-if="win && mode=='collect'">
      <div class="row" v-for="(row, i) in board" :key="row">
        <div class="col" v-for="(col, j) in row" :key="col">
          <div class="block" v-if="board[j][i] == 0"></div>
          <div class="way" v-if="board[j][i] == 1">
            <img src="./assets/player.png" class="player" v-if="player[0] == j && player[1] == i">
            <img src="./assets/evil.png" class="player" v-if="checkEvil(j,i)">
            <img src="./assets/coin.png" class="star" v-if="checkStar(j, i)">
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return{
    controls: false,
      level: 'easy',
      messaage: '',
      showAlert: false,
      mode: '',
      mainMenu: true,
      win: true,
      board: [
        [0, 1, 1, 1, 1, 1, 1, 1, 1, 1],
        [0, 1, 0, 0, 1, 0, 0, 0, 0, 1],
        [0, 1, 1, 1, 1, 0, 1, 1, 1, 1],
        [0, 1, 0, 0, 1, 1, 1, 0, 1, 0],
        [0, 1, 0, 0, 0, 0, 1, 0, 1, 0],
        [1, 1, 1, 1, 1, 0, 1, 0, 1, 0],
        [1, 0, 0, 1, 1, 0, 1, 0, 1, 0],
        [1, 1, 0, 0, 1, 0, 1, 0, 1, 0],
        [0, 1, 1, 1, 1, 1, 1, 0, 1, 0],
        [0, 1, 0, 0, 0, 0, 1, 1, 1, 0],
      ],
      score: 0,
      stars: [],
      player: [9,1],
      evils: [
      ],
      start: [9,1],
      end: [0, 9],
      ways: [],
      rows: 10,
      cols: 10,
    }
  },
  mounted(){
    // get way blocks
    this.getWay()
    if(this.win && this.mode != ''){
      
    }
  },
  methods: {
    stopMusic(){
      document.querySelector(".IWIN").pause();
      document.querySelector(".ILOSE").pause();
    },
    setMode(mode){
      this.controls = true
      this.win = true;
      this.mode = mode;
      this.mainMenu = false;
      if(this.mode == 'survive'){
        this.evils[0] = [this.end[0], this.end[1]]
        setInterval(() => this.moveEvil(0), 500);
      }
      this.player = [this.start[0], this.start[1]]
      addEventListener("keyup", this.handleMove)
      // setInterval(this.checkLose, 500);
      if(mode == 'collect'){
        this.getEvils()
        this.getCoins()
      }
    },
    getWay(){
      for(let i = 0; i < 10; i++){
        for(let j = 0; j < 10; j++){
          let points = [j,i];
          if(this.board[j][i] == 1)this.ways.push(points);
        }
      }
    },
    getEvils(){
      this.evils = []
      this.evils.push([0,1])
      if(this.level != 'easy')this.evils.push([2,8])
      if(this.level == 'hard')this.evils.push([9,8])
      for(let i = 0; i < this.evils.length; i++){setInterval(() => this.moveEvil(i), 500);}
      // for(let i = 0; i < 1; i++){
      //   let p = this.getRandomInt(0, this.ways.length-1)
      //   while(this.ways[p][0] ==this.player[0] && this.ways[p][1] == this.player[1]){
      //     p = this.getRandomInt(0, this.ways.length-1);
      //   }
      //   this.evils.push(this.ways[p])
      // }
    },
    getCoins(){
      for(let i = 0; i < 10; i++){
        let p = this.getRandomInt(0, this.ways.length-1)
        while(this.ways[p][0] ==this.player[0] && this.ways[p][1] == this.player[1]){
          p = this.getRandomInt(0, this.ways.length-1);
        }
        this.stars.push(this.ways[p])
      }
    },
    getRandomInt(min, max) {
      min = Math.ceil(min);   // Round up the minimum value
      max = Math.floor(max);  // Round down the maximum value
      return Math.floor(Math.random() * (max - min + 1)) + min;
    },
    handleMove(event){
      if(!this.win)return
      if (event.key === 'ArrowUp') {
        this.moveUp();
      } else if (event.key === 'ArrowDown') {
        this.moveDown()
      } else if (event.key === 'ArrowLeft') {
        this.moveLeft()
      } else if (event.key === 'ArrowRight') {
        this.moveRight()
      }
    },
    moveUp(){
      let bit = this.player[0] - 1;
      if(bit >= 0 && this.board[bit][this.player[1]] == 1 && bit < 10){
        this.player[0] = bit
        if(this.mode == 'collect')this.checkPoint();
        if(this.mode == 'survive')this.checkWin();
        this.checkLose()
        // document.querySelector(".player").style.transform = "rotate(90deg)"
      }
      // console.log(this.player)
    },
    moveDown(){
      if(this.player[0] == 9)return;
      let bit = this.player[0]+1;
      if(bit >= 0 && this.board[bit][this.player[1]] == 1 && bit < 10){
        this.player[0] = bit
        if(this.mode == 'collect')this.checkPoint();
        if(this.mode == 'survive')this.checkWin();
        this.checkLose()
        // document.querySelector(".player").style.transform = "rotate(-90deg)"
      }
      // console.log(this.player)
    },
    moveLeft(){
      let bit = this.player[1] - 1;
      if(bit >= 0 && this.board[this.player[0]][bit] == 1 && bit < 10){
        this.player[1] = bit
        if(this.mode == 'collect')this.checkPoint();
        if(this.mode == 'survive')this.checkWin();
        this.checkLose()
        // document.querySelector(".player").style.transform = "rotate(0deg)"
      }
      // console.log(this.player)
    },
    moveRight(){
      let bit = this.player[1] + 1;
      if(bit >= 0 && this.board[this.player[0]][bit] == 1 && bit < 10){
        this.player[1] = bit
        if(this.mode == 'collect')this.checkPoint();
        if(this.mode == 'survive')this.checkWin();
        this.checkLose()
        // document.querySelector(".player").style.transform = "rotate(0deg) scaleX(-1)"
      }
      // console.log(this.player)
    },
    checkStar(x,y){
      for(let i = 0; i < this.stars.length; i++){
        if(this.stars[i][0] == x && this.stars[i][1] == y)return true
      }
      return false;
    },
    checkPoint(){
      if(this.checkStar(this.player[0], this.player[1])){
        this.score++;
        this.stars = this.stars.filter(s => !(s[0] == this.player[0] && s[1] == this.player[1]))
        if(this.stars.length == 0){
          this.win = false;
          this.showMessage("Good Pharoah Wins the Game 🥳🥳🥳")
          this.reset();
        }
      }
    },
    checkEvil(x,y){
      for(let i = 0; i < this.evils.length; i++){
        if(this.evils[i][0] == x && this.evils[i][1] == y)return true;
      }
      return false
    },
    checkLose(){
      if(this.checkEvil(this.player[0], this.player[1]) && this.win){
        this.win = false
        if(this.mode == 'survive'){
          this.showMessage("Dollar Wins as usual😭😭")
          document.querySelector(".ILOSE").play();
        }
        if(this.mode == 'collect'){
          this.showMessage("Bad Pharoah Wins 😥")
        }
        this.reset();
      }
    },
    checkWin(){
      if(this.mode != 'survive')return;
      if(this.player[0] == this.end[0] && this.player[1] == this.end[1]){
        this.showMessage("Finally!!..Genieh is Survived Successifully 🥳🥳")
        document.querySelector(".IWIN").play();
        this.reset()
      }
    },
    reset(){
      this.controls = false;
      this.stars = [];
      // this.evils = [];
      this.mode = '';
      this.mainMenu = true;
      this.win = false;
    },
    moveEvil(i){
      if(i >= this.evils.length)return;
      console.log("hi")
      let x = this.evils[i][0]
      let y = this.evils[i][1]
      
      let down = false;
      let up = false;
      let left = false;
      let right = false;
      
      if((x+1) < 10 && this.board[x+1][y] == 1){
        down = true
      }
      // check right
      if((y+1) < 10 && this.board[x][y+1] == 1){
        right = true
      }
      // check up
      if((x-1) > 0 && this.board[x-1][y] == 1){
        up = true
      }
      // check left
      if((y-1) > 0 && this.board[x][y-1] == 1){
        left = true
      }
      let playerx = this.player[0];
      let playery = this.player[1];
      if(playerx > x && down)this.evils[i][0] = x+1;
      else if(playery > y && right)this.evils[i][1] = y+1;
      else if(playerx < x && up)this.evils[i][0] = x-1;
      else if(playery < y && left)this.evils[i][1] = y-1;
      else{
        if(left)this.evils[i][1] = y-1;
        else if(right)this.evils[i][1] = y+1;
        else if(up)this.evils[i][0] = x-1;
        else if(down)this.evils[i][0] = x+1;
      }
      this.checkLose()
    },
    showMessage(message){
      this.messaage = message
      this.showAlert = true
    }
  }
}
</script>
<style scoped>

.app{
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100vw;
  overflow-x: hidden;
  flex-direction: column;
  background: rgb(33, 33, 33);
  height: 100vh;
  user-select: none;
}
header{
  position: absolute;
  color: white;
  top: 20px;
  font-size: 50px;
  font-weight: bold;
}
.mainMenu{
  display: flex;
  align-items: center;
  justify-content:center;
  width: 200px;
  height: 142px;
  flex-direction: column;
  background: #171717;;
  gap: 2px;
  position: relative;
}
.mode{
  width: 200px;
  height: 70px;
  display: flex;
  justify-content: center;
  align-items: center;
  font-weight: bold;
  color: white;
  cursor: pointer;
}
.collect{
  position: relative;
}
.survive:hover{
  background: rgb(95, 95, 95);
}
.collect div{
  width: 200px;
  height: 70px;
  display: flex;
  align-items: center;
  justify-content: center;
}
.collect div:hover{
  background: rgb(95, 95, 95);
}
#level:hover{
  background: rgb(95, 95, 95);
}
select option {
  background-color: rgb(95, 95, 95);
}
#level{
  background: transparent;
  color: white;
  position: absolute;
  right: 0;
  top: 0;
  cursor: pointer;
}

.controls{
  display: flex;
  flex-direction: column;
  gap: 1px;
  width: 150px;
  height: 100px;
  align-items: center;
  justify-content: center;
  position: absolute;
  left: 50%;
  bottom: 0;
  transform: translateX(-50%)
}
.do{
  display:flex;
  gap: 1px;
}
.key{
  display: flex;
  align-items: center;
  justify-content: center;
  width: 50px;
  height: 50px;
  background: red;
  color: white;
  cursor: pointer;
  font-weight: bold;
}
.key::selection{
  background:transparent
}
.key:hover{
  background: brown;
}
.board{
  width: 500px;
  height: 500px;
  background: rebeccapurple;
  display: grid;
  grid-template-columns: repeat(10, 50px);
  border: 5px solid black;
}
.board .row{
  width: 500px;
  height: 50px;
}
.col{
  width: 50px;
  height: 50px;
  overflow: hidden;
}
.block{
  width: 50px;
  height: 50px;
  background:linear-gradient(#a49d83 50%, #0000 0) 75% 0/2px 50% repeat-y, linear-gradient(#0000 50%, #a49d83 0) 25% 0/2px 50% repeat-y, linear-gradient(#d8ce5f calc(100% - 2px), #a49d83 0) 0 0/100% calc(100%/4);
}
.way{
  width: 50px;
  height: 50px;
  background-image: url(./assets/grass.jpg);
  display: flex;
  align-items: center;
  justify-content: center;
}
.player{
  width: 40px;
  height: 40px;
  transform: rotate(0deg);
  border-radius: 50%;
}
.star{
  width: 40px;
  height: 40px;
}
.message {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%,-50%);
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  z-index: 100;
  color: white;
  height: 150px;
  background: #2b2828;
  padding: 20px;
}
.message .content {
  padding: 5px;
  margin-bottom: 10px;
  /* max-width: 300px; */
  font-weight: bold;
  color: #fff38d;
  font-size: 25px;
}
.message .ok{
  margin-top: 10px;
  font-size: 15px;
  background: #9b9b9b;
  color: black;
  padding: 5px;
  border-radius: 5px;
  font-weight: bold;
  cursor: pointer;
}
.message .ok:hover{
  background: #c4c4c4;
}
.audio{
  position: absolute;
  width: 100px;
  height: 50px;
  background: white;
  z-index: 100;
  display: none;
}
@media screen and (max-width: 600px) {
  .board{
    transform: scale(0.75);
  }
}
</style>
<template>
  <div class="card">
    <canvas id="myCanvas" tabindex="1" @keydown="onKeyPress"/>
  </div>
  <div>
    <Button label="play" @click="pollData"/>
    <div class="flex gap-3"><span>Score: {{score}}</span></div>
  </div>
</template>

<script>
import { ref } from 'vue';
import Button from 'primevue/button';

export default {
  name: 'SnakeGame',
  props: {
    msg: String
  },
  components: {
    Button
  },
  data: () => ({
    RIGHT: 'right',
    DOWN: 'down',
    LEFT: 'left',
    UP: 'up',
    canvas: null,
    snakeBody: ref([{x:0, y:0}]), //Тело змейки,
    direction: ref('right'),
    applePosition: ref(0,0), //Яблоко, массив, 0 элемент - x, 1 элемент - y.
    polling: null,
    interval: 240,
    c: null,
    head: null,
    headCoord: null,
    tail: null,
    WITH_SNAKE: 30,
    GAP: 2,
    score: ref(0)
  }),
  mounted() {
    this.c = document.getElementById("myCanvas")
    this.canvas = this.c.getContext('2d')
    this.c.width = Math.round(1800/2)
    this.c.height =  Math.round(900/2)
  },
  methods: {
    addScore(){
      this.score = this.score + 100
    },
    getRand(max){
      let k = Math.random() * max
      return (Math.round(k/this.WITH_SNAKE)) * this.WITH_SNAKE
    },
    newApplePosition(){
      this.applePosition = [this.getRand(this.c.width), this.getRand(this.c.height)]
    },
    async newSnakeFirstPosition(){
      this.snakeBody = await [{x: 0,y: 0}]
    },
    startGame(){
      this.newApplePosition()
      this.newSnakeFirstPosition().then(()=>
      this.initSnake())
    },
    async initSnake(){
      for (let i = 0; i < 2; i++) {
        if (this.snakeBody!=null) {
          this.head = this.snakeBody[0]
          this.headCoord = {x: this.head.x, y: this.head.y}
          this.tail = this.snakeBody.slice(-1)[0]
          this.headCoord.x = this.tail.x + this.WITH_SNAKE
          this.headCoord.y = this.getSameCoord(this.tail.y)
          this.snakeBody.push(this.headCoord)
        }
    }
    },
    playGame(){
      if (this.applePosition[0] + this.WITH_SNAKE >= this.c.width ||
          this.applePosition[1] + this.WITH_SNAKE >= this.c.height) {
        this.newApplePosition()
      }
      this.canvas.clearRect(0, 0, this.c.width, this.c.height)
      this.canvas.fillStyle = 'red'
      this.canvas.fillRect(...this.applePosition, this.WITH_SNAKE - this.GAP, this.WITH_SNAKE - this.GAP)
      this.canvas.fillStyle = 'green'
      ////console.log("snakeBody", this.snakeBody)
      if (Array.isArray(this.snakeBody)){
      for (let i = 0; i < this.snakeBody.length; i++) {
        //здесь ф-я crash
          let last = this.snakeBody.length - 1
          if (
              this.snakeBody[i].x == this.snakeBody[last].x &&
              this.snakeBody[i].y == this.snakeBody[last].y &&
              i < last
              )
            {
            this.snakeBody.splice(0, last)
            this.snakeBody = [{x:0, y:0}]
            this.initSnake()
            this.direction = this.RIGHT
            }
          //}
      }}
      if (this.snakeBody!=null) {
          this.head = this.snakeBody[0]
      this.headCoord = {x: this.head.x, y: this.head.y}
      this.tail = this.snakeBody.slice(-1)[0]
      if (this.direction == this.RIGHT){
        this.headCoord.x = this.tail.x + this.WITH_SNAKE
        this.headCoord.y = this.getSameCoord(this.tail.y)
      }
      if (this.direction == this.DOWN){
        this.headCoord.y = this.tail.y + this.WITH_SNAKE
        this.headCoord.x = this.getSameCoord(this.tail.x)
      }
      if (this.direction == this.LEFT){
        this.headCoord.x = this.tail.x - this.WITH_SNAKE
        this.headCoord.y = this.getSameCoord(this.tail.y)
      }
      if (this.direction == this.UP){
        this.headCoord.y = this.tail.y - this.WITH_SNAKE
        this.headCoord.x = this.getSameCoord(this.tail.x)
      }
      this.snakeBody.push(this.headCoord)
      this.snakeBody.splice(0,1)
      let xMax = this.getSameCoord(this.c.width-this.WITH_SNAKE)
      let yMax = this.getSameCoord(this.c.height-this.WITH_SNAKE)
      if (Array.isArray(this.snakeBody))
        for (let i = 0; i < this.snakeBody.length; i++) {
          //console.log('xMax: ' + xMax)
        if (this.direction == this.RIGHT){
        if (this.snakeBody[i].x > xMax){
            this.snakeBody[i].x = 0
        }
        }
        if (this.direction == this.DOWN){
          if (this.snakeBody[i].y > yMax){
              this.snakeBody[i].y = 0
          }
        }
        if (this.direction == this.LEFT){
          if (this.snakeBody[i].x < 0) {
              this.snakeBody[i].x = xMax
          }
        }
        if (this.direction == this.UP){
          if (this.snakeBody[i].y < 0){
              this.snakeBody[i].y = yMax
          }
        }
        if (this.snakeBody[i].x == this.applePosition[0] && this.snakeBody[i].y == this.applePosition[1]){
          this.snakeEatApple()
          }
          this.canvas.fillRect(this.snakeBody[i].x, this.snakeBody[i].y, this.WITH_SNAKE - this.GAP, this.WITH_SNAKE - this.GAP)
        }
    }
    },
    getSameCoord(param){
      return param
      //return Math.round((param/this.WITH_SNAKE)*this.WITH_SNAKE)
      //console.log('param', param)
    },
    crashSnake(){
      let last = this.snakeBody.length - 1
      if (
          this.snakeBody[0].x == this.snakeBody[last].x &&
          this.snakeBody[0].y == this.snakeBody[last].y //&&
          //i < last
          ){
        this.snakeBody.splice(0, last)
        this.snakeBody = [{x:0, y:0}]
        this.direction = this.RIGHT
      }
    },
    screenBorders(i){
      if (this.direction == this.RIGHT){
        if (this.snakeBody[i].x > this.getSameCoord(this.c.with)){
            this.snakeBody[i].x = 0
        }
      }
      if (this.direction == this.DOWN){
        if (this.snakeBody[i].y > this.getSameCoord(this.c.height)){
            this.snakeBody[i].y = 0
        }
      }
      if (this.direction == this.RIGHT){
        if (this.snakeBody[i].x < 0) {
            this.snakeBody[i].x = this.getSameCoord(this.c.with)
        }
      }
      if (this.direction == this.DOWN){
        if (this.snakeBody[i].y < 0){
            this.snakeBody[i].y =  this.getSameCoord(this.c.height)
        }
      }
    },
    pollData(){
      this.startGame()
      //this.c.addEventListener("keypress", this.onKeyDown)
      this.polling = setInterval(() => {
        this.playGame()
      }, this.interval)
    },
    snakeEatApple(){
      this.newApplePosition()
      this.snakeBody.unshift({x: this.headCoord.x - this.WITH_SNAKE,
                              y: this.tail.y})
      this.addScore()
    },
    onKeyPress(event) {
      //console.log("move!!!!!!!!!!")
      //console.log('event', event)
      let key = event.keyCode
      if ([38, 39, 40, 37].indexOf(key) >= 0){
        event.preventDefault()
      }
      if (key == 39 && this.direction != this.LEFT){
        this.direction = this.RIGHT
      }
      if (key == 40 && this.direction != this.UP){
        this.direction = this.DOWN
      }
      if (key == 37 && this.direction != this.RIGHT){
        this.direction = this.LEFT
      }
      if (key == 38 && this.direction != this.DOWN){
        this.direction = this.UP
      }
    },
  },
  beforeUnmount(){
    clearInterval(this.polling)
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
#myCanvas {
  border: 1px solid grey;
}
</style>

<template>
  <div id="app">
    <!-- secao para p play -->
    <template  v-if="!gameOver && !newLevel">
      <mm-header :pontos="pontos" :health="health"/>
      <mm-question :num1="num1" :num2="num2" />
      <mm-answer :sugestoes="sugestoes" :questaoRespondida="questaoRespondida" />
      <mm-timer :temporestante="time" :settings="settings" :getProgressClass="getProgressClass" :messageClass="messageClass" :getBonus="getBonus" :notification="notification" :result="result" />
    </template>

    <!-- estado da acao -->
    <template v-if="gameOver">
      <mm-stats :pontos="pontos" :totalTempo="totalTempo" :initGame="initGame" />
    </template>

    <!-- mensagem de alerta para novo nivel -->
      <div v-show="newLevel" class="new-level alert alert-success text-center" role="alert">
        <h4 class="alert-heading">Nível {{ gameLevel+1 }}</h4>
      </div>
  
  </div>
</template>

<script>
import Header from './components/Header.vue'
import Question from './components/Question.vue'
import Answer from './components/Answer.vue'
import Timer from './components/Timer.vue'
import Stats from './components/Stats.vue'

import { eventBus } from "./main.js"

export default {
  name: 'app',
  data () {
    return {
      num1: 0,
      num2: 0,
      playerResponda: "",
      notification:"",
      pontos: 0,
      questaoRespondida: false,
      time: 0,
      health: 0,
      result:"",
      gameOver:false,
      totalTempo:0,
      sugestoes:[],
      tries:0,
      gameLevel:0,
      newLevel:false,

      timerInertval: null,

      settings:{
        timeOut: 20,
        health:3,
        pauseAfterQuestion:2000,
        numSugestao:10,

        levels:[
          {
            numMin: 1,
            numMax: 10,
          },
          {
            numMin: 10,
            numMax: 50,
          },
          {
            numMin: 50,
            numMax: 100,
          },
          {
            numMin: 100,
            numMax: 200,
          },
        ],
      }

    }
  },
  components:{
    "mm-header":   Header,
    "mm-question": Question,
    "mm-answer":   Answer,
    "mm-timer":    Timer,
    "mm-stats":    Stats,
  
  },
  created:function(){
    self = this

    eventBus.$on("userAnswered",function(answer){
      self.sendResult(answer)
    })
  },
  methods:{
    getRandomInt: function(min, max, except) {
      /*retorna um numero aleatorio entre o valor minimo e maximo */
      var numMin,numMax
      var currentLevel = this.settings.levels[ this.gameLevel ]

      if( min === undefined || min == 0  ){
        numMin = currentLevel.numMin
      }else{
        numMin = min
      }

      if( max === undefined || max == 0 ){
        numMax = currentLevel.numMax
      }else{
        numMax = max
      }

      var ret

      do{
        ret = Math.floor(Math.random() * (numMax - numMin)) + numMin
      }while( (except != undefined && except.includes(ret) ) )

      return ret
    },
    sendResult: function(ans){

      this.tries++

      this.playerResponda = ans

      if( this.playerResponda == "" && this.time > 0 ){
        return false
      }

      clearInterval(this.timerInertval)
      
      var theAnswer = this.theGoodAnswer()

      if( this.playerResponda == theAnswer ){
        this.goodAnswer()
      }else if( this.playerResponda == "" && this.time == 0 ){
        this.timeOutAnswer()
      }else{
        this.badAnswer(theAnswer)
      }

      this.questaoRespondida = true

      var self = this
      setTimeout(function(){
        if( !self.gameOver ){
          self.newQuestion()
        }
      }, self.settings.pauseAfterQuestion)
    },
    goodAnswer: function(){
      this.result = "Bom"
      this.notification = "Boa resposta"

      this.totalTempo += (this.settings.timeOut - this.time)

      this.pontos += ( 1 + this.getBonus() )
    },
    getBonus:function(){
      return this.time < (this.settings.timeOut/2) ? (this.time < (this.settings.timeOut/4) ? 0 : 1 ) : 2
    },
    badAnswer: function( ans ){
      this.result = "Ruim"
      this.notification = "Errou na resposta! A reposta correta é ["+ans+"]"
      this.health --
    },
    timeOutAnswer: function(){
      this.result = "Empatou"
      this.notification = "Acabou seu tempo!"
      this.health --
    },
    initGame: function(){
      this.health = this.settings.health
      this.pontos = 0
      this.gameOver = false
      this.totalTempo = 0

      this.newQuestion()
    },
    newQuestion: function(){
      this.num1 = this.getRandomInt()
      this.num2 = this.getRandomInt()
      
      this.assignSuggestions()

      this.playerResponda = ""
      this.questaoRespondida = false
      this.notification = ""
      this.time = this.settings.timeOut
      this.setTimer()
    },
    setTimer: function(){
      self = this
      this.timerInertval = setInterval(function(){
        self.time--
      },1000)
    },
    endGame: function(){
      this.gameOver = true
      clearInterval(this.timerInertval)
    },
    messageClass:function(){

      switch(this.result){

        case "Bom":
        return "alert-success"
        break;

        case "Ruim":
        return "alert-danger"
        break;

        case "Empatou":
        return "alert-primary"
        break;

        default:
        break;
      }
    },
    getProgressClass:function(){
      switch (this.getBonus()){
        case 0:
        return 'bg-danger'
        break;

        case 1:
        return 'bg-warning'
        break;

        case 2:
        return 'bg-success'
        break;

        default:
        break;

      }
    },
    theGoodAnswer:function(){
      return this.num1 + this.num2
    },
    assignSuggestions:function(){
      var arrLn = this.settings.numSugestao

      var currentLevel = this.settings.levels[ this.gameLevel ]

      var min = currentLevel.numMin
      var theAnswer = this.theGoodAnswer()
      var max = 2 * theAnswer

   
      var ret = [theAnswer]

      for (var i = arrLn - 2; i >= 0; i--) {
        ret.push( this.getRandomInt( min, max, ret ) )
      }

      /* ordem aleatoria */
      ret = ret.sort(() => Math.random() * 2 - 1).sort(() => Math.random() * 2 - 1)

      this.sugestoes = ret
    },
  },
  computed:{

  },
  mounted: function(){
    this.initGame()
  },

  watch:{
    time: function (tm, oldVal) {
      if( tm == 0 ){
        this.sendResult("")
      }
    },
    health: function(hlt, oldVal){
      if( hlt == 0 ){
        this.endGame()
      }
    },
    tries: function(tries){
      /* Change levels */
      if( tries <= 5 ){
        this.gameLevel = 0
      }else if( tries <= 10 ){
        this.gameLevel = 1        
      }else if( tries <= 15 ){
        this.gameLevel = 2        
      }else if( tries <= 20 ){
        this.gameLevel = 3        
      }
    },
    gameLevel:function(lvl){
      self = this
      
      self.newLevel = true

      setTimeout(function(){
        self.newLevel = false
      },self.settings.pauseAfterQuestion)
    }
  }
}


</script>

<style type="text/css">
body {
  padding-top: 60px;
}
.operation div {
  display: inline-block;
}
.new-level .alert-heading{
  font-size: 4rem;  
}
</style>
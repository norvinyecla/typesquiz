<template>
  <div id="game">
    <h1>
      Pokémon Types Quiz
    </h1>
    <div class="main" v-show="turns <= maxTurns">
      <div class="score">
        <h2>Your Score: {{ score }} / {{ maxTurns }}</h2>
      </div>
      <h2>{{ current.name }}</h2>

      <img :src="current.sprite" height=150 width=150 />

      <div class="button-set">
        <div class="button" v-bind:class="choices[0]" @click="select(choices[0])">
          {{ choices[0] }}
        </div>
        <div class="button" v-bind:class="choices[1]" @click="select(choices[1])">
          {{ choices[1] }}
        </div>
        <div class="button" v-bind:class="choices[2]" @click="select(choices[2])">
          {{ choices[2] }}
        </div>
      </div>
      <div class="current-selection">
        <h5 v-bind:class="[ currentSelection == null ? isInvisibleClass : '' ]">
          You chose {{ currentSelection }}
        </h5>
      </div>
      <span>
        <div class="button primary" @click="submit">
          Submit
        </div>
      </span>
    </div>
    <div class="final" v-show="turns > maxTurns">
      <h5 class="total-score-message">
        Your score is {{ score }}/{{ maxTurns }}
        <br/>
        {{ message }}
      </h5>
      <div class="button primary" @click="start">
        Play Again
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Game',

  data() {
    return {
      dash: null,
      ct: null,
      score: 0,
      current: {
        sprite: null,
        name: null,
        types: null
      },
      P: null,
      t: null,
      choices: [ 
        null, 
        null, 
        null 
      ],
      answer: null,
      currentSelection: null,
      turns: 0,
      maxTurns: 10,
      isInvisibleClass: 'is-invisible',
      message: ''
    }
  },
  methods: {
    getRandomNumber() {
      return this.dash.random(1, 802)
    },
    submit() {
      if (this.current.types[0] == this.currentSelection || this.current.types[1] == this.currentSelection) {
        alert('You are correct!')
        this.score++
      } else {
        alert('Sorry! It\'s ' + this.answer + '!')
      }

      this.message = this.getMessage(this.score)

      if (this.turns <= this.maxTurns) {
        this.reload()
      } else {
        this.hide()
      }
    },
    select(selected) {
      this.currentSelection = selected
    },
    start() {
      this.turns = 0
      this.currentSelection = null
      this.current = {}
      this.score = 0
      this.ct = 0
      this.reload()
    },
    reload() {
      this.currentSelection = null
      this.ct = this.getRandomNumber()
      this.fetch()
    },
    generateChoices(types) {
      var answer = null
      if (types.length == 2) {
        const id = this.dash.random(0, 1)
        answer = types[id]
      } else {
        answer = types[0]
      }
      this.answer = answer

      const allTypes = [ 'normal', 'fire', 'fighting', 'water', 'flying', 'grass', 'poison', 'electric', 'ground', 'psychic',
'rock', 'ice', 'bug', 'dragon', 'ghost', 'dark', 'steel', 'fairy' ]
      
      var shuffled = this.dash.shuffle(allTypes)
      while (shuffled[0] == answer || shuffled[1] == answer) {
       shuffled = this.dash.shuffle(allTypes)
      }
      var choices = [ shuffled[1], answer, shuffled[0] ]

      return this.dash.shuffle(choices)
    },
    fetch() {
      var t = this
      this.turns = this.turns + 1
      this.P.resource('/api/v2/pokemon/' + this.ct + '/')
      .then(function(response) {
        t.current.name = t.dash.capitalize(response.name)
        t.current.sprite =  response.sprites.front_default, 

        t.current.types = [ response.types[0].type.name ]
        if (response.types.length > 1) {
          t.current.types.push(response.types[1].type.name)
        }

         t.choices = t.generateChoices(t.current.types)
      })
    },
    getMessage(score) {
      var message = ''
      if (score < 3) {
        message = 'Tough luck, try harder!'
      } else if ( score >= 3 && score < 5 ) {
        message = 'Not bad!'
      } else if (score >= 5 && score < 8 ) {
        message = 'Good job!'
      } else if (score >= 8 && score < 10) {
        message = 'You are amazing in this!'
      } else if (score == 10 ) {
        message = 'You are a Pokémon master!'
      }

      return message
    }
  },

  mounted() {
    const dash = require('lodash')
    const Pokedex = require('pokeapi-js-wrapper')
    const P = new Pokedex.Pokedex({
      protocol: 'https'
    })
    this.P = P
    this.dash = dash
    this.ct = this.getRandomNumber()
    
    this.fetch(this.ct)
  }
}

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
@media all {
  #game {
    width: 70%;
  }
}
@media all and (max-width: 991px) {
  #game {
    width: 75%;
  }
}

@media all and (max-width: 768px) {
  #game {
    width: 85%;
  }
}
@media all and (max-width: 575px) {
  #game {
    width: 95%;
  }
}
h1 {
  font-weight: bolder;
}
#game {
  background-color: #f4f4f4;
  margin: auto;
  padding: 25px;
}
.score {
  margin-bottom: 10px;
}
.button {
  color: white;
  text-shadow: 0 .10em 0.10em rgba(0,0,0,0.25) !important;
}
.button:hover {
  filter: brightness(80%);
  color: white !important;
  overflow: hidden
}
.primary {
  background-color: lightskyblue;
  color: white !important;
  border: darkblue;
}
.primary:hover {
  zoom: 1;
}
.is-invisible {
  visibility: hidden
}
.button-set {
  margin: 5px;
}
.current-selection{
  margin-top: 15px;
  margin-bottom: 15px;
}
.final {
  margin-top: 20px;
}
.total-score-message {
  margin-top: 10px;
  margin-bottom: 10px;
}
.normal { 
  background-color: #A8A77A; 
}
.fire { 
  background-color: #EE8130; 
}
.water { 
  background-color: #6390F0; 
}
.electric { 
  background-color: #F7D02C; 
}
.grass { 
  background-color: #7AC74C; 
}
.ice { 
  background-color: #96D9D6; 
}
.fighting { 
  background-color: #C22E28; 
}
.poison { 
  background-color: #A33EA1; 
}
.ground { 
  background-color: #E2BF65; 
}
.flying { 
  background-color: #A98FF3; 
}
.psychic { 
  background-color: #F95587; 
}
.bug { 
  background-color: #A6B91A; 
}
.rock { 
  background-color: #B6A136; 
}
.ghost { 
  background-color: #735797; 
}
.dragon { 
  background-color: #6F35FC; 
}
.dark { 
  background-color: #705746; 
}
.steel { 
  background-color: #B7B7CE; 
}
.fairy { 
  background-color: #D685AD; 
}
</style>

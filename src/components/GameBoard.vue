<template>
  <div class="gameboard">
    <p>Enter a username below to get a random public repo name for your game.</p>
    <ErrorMessage v-if="errorMessage" v-bind:message="errorMessage"></ErrorMessage>
    <form v-on:submit.prevent="getName" v-if="!gameStart">
      <label>Github Username:
        <input type="text" v-model="username" id="username"/>
      </label>
      
      <input type="submit" value="Get Repo Name">
    </form>
    <div v-if="gameOver">
      <p v-if="gameWon">You Won!</p>
      <p v-else>You Lost.</p>
      <button v-on:click="gameReset">Play again?</button>
    </div>
    <div v-if="gameStart">
      <p>Repo name: {{ repo }}</p>
      <p>Correct Guesses: {{correctGuesses.join("")}}</p>
      <p>Wrong Guesses: {{wrongGuesses}}</p>
      <form v-on:submit.prevent="guessCharacter" v-if="!gameOver">
        <label>Guess a Letter:
          <input type="text" v-model="currentGuess" id="currentGuess"/>
        </label>
        <input type="submit" value="Guess Letter">
        <p>Previous Guesses: {{prevGuesses.join(', ')}}</p>
      </form>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import ErrorMessage from './ErrorMessage';
export default {
  name: "GameBoard",
  components: {
    ErrorMessage
  },
  data: () => {
    return {
      username: "",
      repo: "",
      gameStart: false,
      currentGuess: "",
      prevGuesses: [],
      errorMessage: "",
      correctGuesses: [],
      wrongGuesses: 0,
      gameWon: null,
      gameOver: false
    };
  },
  methods: {
    getName: function() {
      const vm = this;
      axios
        .get(`https://api.github.com/users/${vm.username}/repos`)
        .then(response => {
          vm.repo = response.data[0].name;
          vm.gameStart = true;
          for (let i = 0; i < vm.repo.length; i++) {
            vm.correctGuesses.push("_");
          }
        });
    },
    guessCharacter: function() {
      const vm = this;
      if (vm.errorMessage) {
        vm.errorMessage = "";
      }
      const theGuess = vm.currentGuess.toLowerCase().trim();
      const validCharacter = this.checkCharacter(theGuess);
      if (theGuess == false) {
        vm.errorMessage = "Please add a letter.";
        return vm.errorMessage;
      }
      // If valid, push to array.
      if (validCharacter) {
        vm.prevGuesses.push(theGuess);
        this.isGuessCorrect(theGuess);
      }
      if (vm.wrongGuesses >= 6) {
        vm.gameOver = true;
        vm.gameWon = false;
      }

      if (vm.correctGuesses.join("") === vm.repo) {
        vm.gameOver = true;
        vm.gameWon = true;
      }
      vm.currentGuess = "";
    },
    checkCharacter: function(guess) {
      const vm = this;
      // Check if it has been guessed before.
      if (vm.prevGuesses.includes(guess)) {
        vm.errorMessage = `You have already guessed: ${guess}.`;
        return false;
      }
      // Check if guess is length of one.
      if (guess.length > 1) {
        vm.errorMessage = `You guessed: ${guess}. Please guess only 1 letter each time.`;
        return false;
      }
      return true;
    },
    isGuessCorrect: function(guess) {
      const vm = this;
      const gameString = vm.repo.split("");
      let correctGuess = false;
      for (let i = 0; i < gameString.length; i++) {
        if (guess === gameString[i].toLowerCase()) {
          vm.correctGuesses[i] = guess;
          correctGuess = true;
        }
      }
      if (!correctGuess) {
        vm.errorMessage = `Sorry ${guess} is not in the current repo name.`;
        vm.wrongGuesses++;
        return false;
      }
    },
    gameReset: function() {
      this.username = "";
      this.repo = "";
      this.gameStart = false;
      this.currentGuess = "";
      this.prevGuesses = [];
      this.errorMessage = "";
      this.correctGuesses = [];
      this.wrongGuesses = 0;
      this.gameWon = null;
      this.gameOver = false;
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
div {
  border: 1px solid #212121;
}
</style>

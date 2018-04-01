<template>
  <div class="gameboard">
    <button v-on:click="showRules = !showRules">{{showRules ? "Hide Rules" : "Show Rules"}}</button>
    <div class="game-rules" v-if="showRules">
      <h2>Game Rules:</h2>
      <ul>
        <li>Can guess incorrectly 6 times before losing.</li>
        <li>Any letter, special character or number that is a valid repo name is fair game. (This includes characters like underscores, hyphens, and periods.)</li>
        <li>All letters will be tested and converted to lowercase.</li>
        <li>Only one character per guess.</li>
        <li>Guessing more than one character or a character already guessed won't count against total wrong guesses.</li>
      </ul>
      
    </div>
    <ErrorMessage v-if="errorMessage" v-bind:message="errorMessage"></ErrorMessage>
    <form v-on:submit.prevent="getName" v-if="!gameStart">
      <p>To start, enter the username or organization name. The game will grab a random repo and you have to guess it.</p>
      <label>Github Username:
        <input type="text" v-model="username" id="username"/>
      </label>
      
      <input type="submit" value="Get Repo Name">
    </form>
    <div v-if="gameOver">
      <p v-if="gameWon">You Won!</p>
      <p v-else>You Lost.</p>
      <p>You can visit <span class="repo-link-name">{{repo}}</span> @ <a :href="'https://github.com/' + username + '/' + repo" target="_blank">https://github.com/{{username}}/{{repo}}</a></p>
      <button v-on:click="gameReset">Play again?</button>
    </div>
    <div v-if="gameStart">
      <p><img class="github-avatar" :src="userAvatar" :alt="username + ' Github avatar'"/> <span class="repo-name">{{correctGuesses.join("")}}</span></p>
      <p>Wrong Guesses: {{wrongGuesses}} / 6</p>
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
      gameOver: false,
      showRules: true,
      userAvatar: ""
    };
  },
  methods: {
    getName: function() {
      const vm = this;
      axios
        .get(`https://api.github.com/users/${vm.username}/repos`)
        .then(response => {
          const randomIndex = this.randomRepo(response.data.length);
          vm.repo = response.data[randomIndex].name;
          vm.gameStart = true;
          vm.userAvatar = response.data[randomIndex].owner.avatar_url;
          for (let i = 0; i < vm.repo.length; i++) {
            vm.correctGuesses.push("*");
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
    },
    randomRepo: max => {
      const min = 0;
      return Math.floor(Math.random() * (max - min + 1)) + min;
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.gameboard {
  border: 1px solid #212121;
  padding: 20px 10px;
}
.game-rules {
  text-align: left;
}
.github-avatar {
  width: 100px;
  border-radius: 50%;
}
.repo-name {
  letter-spacing: 5px;
}
.repo-link-name {
  font-weight: bold;
}
</style>

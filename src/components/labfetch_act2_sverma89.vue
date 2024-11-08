<script>
export default {
  props: {
      numberOfPlayers: Number,
      numberOfCategories: Number,
  },
  data() {
    return {
      listOfTopics: [], 
      player_scores: new Array(this.numberOfPlayers).fill(0),
      isQuestionClicked: false,
      questionRow: 0,
      questionCol: 0,
      player_turn: 1,
      prev_player: 1,
      processedCells: 0, 
      selectedAnswer: null,    
      answeredQuestionStatus: "",
      difficulty: ["easy", "medium", "hard"],
      isQuestionInProgress: false,
      selectedQuestion: [],
      gridValues: [new Array(this.numberOfCategories).fill("100$"), new Array(this.numberOfCategories).fill("200$"), new Array(this.numberOfCategories).fill("300$")],
      textColor: [new Array(this.numberOfCategories).fill("#FFD700"),new Array(this.numberOfCategories).fill("#FFD700"), new Array(this.numberOfCategories).fill("#FFD700")],
      isGameFinished: false,
      gameResult: "",
      isTopicLoaded: false,
      doubleJeopardyCells: [],
      currentAmount: 0,
      isQuestionLoaded: false,
      isFinalJeopardy: false,
      finalPlayers: [],
      finalJeopadyTopic: [],
      finalJeopadyQuestion: [],
      finalJeopadyPlayerIndex: 0,
      finalWagerAmounts: [],
      selectedJeopardyAnswer: null,
      isCorrect: [],
      finalJeopadyAnswers: [],
      playerOrder: []

    }
  },
  computed: {
    selectedQuestionParsing() {
      return this.selectedQuestion[0]["question"]; 
    },
    finalJoepadyquestionParsing(){
      return this.finalJeopadyQuestion[0]["question"];
    }
  },
 async mounted() {
    await this.fetchApplicationData(); 
    this.isTopicLoaded = true;
  },
  methods: {
    async fetchFinalJeopardy(){
      var isRes = false;
      var listOfTopics = [];
      while(!isRes){
        var response = await fetch("https://opentdb.com/api_category.php");
        var data = await response.json();
        if(data["response_code"] == 5){
          continue;
        }
        isRes = true;
        listOfTopics = data["trivia_categories"];
        listOfTopics = this.shuffle(listOfTopics);
      }
      var isQuesFound = false;
      var ind = 0;
      while(!isQuesFound){
      var response = await fetch(" https://opentdb.com/api.php?amount=1&category="+listOfTopics[ind]["id"]+"&difficulty=hard&type=multiple");
      
      var data = await response.json();
      if(data["response_code"] != 5){
        this.finalJeopadyTopic = listOfTopics[ind];
        this.finalJeopadyQuestion = data["results"];
        break;
      }
      response = await fetch(" https://opentdb.com/api.php?amount=1&category="+listOfTopics[ind]["id"]+"&difficulty=medium&type=multiple");
      
      data = await response.json();
      if(data["response_code"] != 5){
        this.finalJeopadyTopic = listOfTopics[ind];
        this.finalJeopadyQuestion = data["results"];
        break;
      }

      ind++;
      }
      this.finalJeopadyQuestion[0]["choices"] = this.finalJeopadyQuestion[0]["incorrect_answers"];
      this.finalJeopadyQuestion[0]["choices"].push(this.finalJeopadyQuestion[0]["correct_answer"]);
      this.finalJeopadyQuestion[0]["choices"] = this.shuffle(this.finalJeopadyQuestion[0]["choices"]);
    },
    async assignDoubleJeopardyCells() {
      let count = 0;
      while (count < 2) {
        const randomRow = Math.floor(Math.random() * 3);
        const randomCol = Math.floor(Math.random() * this.numberOfCategories);
        const cell = this.numberOfCategories*randomRow + randomCol;

        if (!this.doubleJeopardyCells.includes(cell)) {
          this.doubleJeopardyCells.push(cell);
          count++;
        }
      }
    },
    async questionAnswered(){
      if(this.selectedAnswer == this.selectedQuestion[0]["correct_answer"]){
        this.answeredQuestionStatus = "Correct!";
        this.gridValues[this.questionRow][this.questionCol] = "P"+this.player_turn;
        this.textColor[this.questionRow][this.questionCol] = "green";
        this.player_scores[this.player_turn-1] += this.currentAmount;
        
        setTimeout(() => {
        this.isQuestionClicked = false; 
        this.isQuestionLoaded = false;
        this.isQuestionInProgress = false;
        this.selectedAnswer = null;
        this.answeredQuestionStatus = "";
        this.selectedQuestion = [];
      }, 1000);
        
      }
      else{
        this.answeredQuestionStatus = "Incorrect!";
        this.gridValues[this.questionRow][this.questionCol] = "P"+this.player_turn;
        this.textColor[this.questionRow][this.questionCol] = "red";
        this.player_scores[this.player_turn-1] -= this.currentAmount;
        this.player_turn +=1;
        if(this.player_turn == this.numberOfPlayers+1){
          this.player_turn = 1;
        }
        setTimeout(() => {
        this.isQuestionClicked = false; 
        this.isQuestionLoaded = false;
        this.selectedAnswer = null;
        this.answeredQuestionStatus = "";
        this.selectedQuestion = [];
      }, 1000);

        setTimeout(() => {
        this.prev_player = this.player_turn;
      }, 1500);
        
      }

      this.processedCells += 1;

      if(this.processedCells == 3*this.numberOfCategories){
        var winners = [];
        for(var i = 0; i < this.numberOfPlayers; i++){
          if(this.player_scores[i] > 0){
            winners.push(i+1);
          }
        }
        if(winners.length == 0){
          this.gameResult = "No one won :(";
          this.isGameFinished = true;
        }
        else if(winners.length == 1 ){
          this.gameResult = "Player "+(winners[0])+" won!!!";
          this.isGameFinished = true;
        }
        else{
          this.isFinalJeopardy = true;
          this.finalPlayers = winners;
          this.finalPlayers = this.finalPlayers.map((index, i) => [index, this.player_scores[index-1]]) .sort((a, b) => b[1] - a[1]) .map((pair) => pair[0]); 
 
          for(var ind in this.finalPlayers){
            this.finalWagerAmounts.push(await this.promptWagerFinalJeopardy(this.finalPlayers[ind]));
          }
        }
        
      }

    },
    async handleCellClick(rowIndex, colIndex) {
      const cellId = this.numberOfCategories*rowIndex + colIndex;
      if(this.isQuestionClicked){
        alert("Please answer the current question before clicking on another one");
        return;
      }
      
      else if (this.textColor[rowIndex][colIndex] != "#FFD700") {
       
        alert("The question was already answered");
        return;
      } 
      else{
        this.questionRow = rowIndex;
        this.questionCol = colIndex;
        this.currentAmount = (this.questionRow+1)*100;
        if (this.doubleJeopardyCells.includes(cellId) && this.currentAmount <= this.player_scores[this.player_turn-1]) {
          this.currentAmount = await this.promptWager(this.currentAmount);
        }
        this.isQuestionClicked = true;
        this.selectedQuestion = await this.fetchQuestion(this.listOfTopics[this.questionCol]["id"], this.difficulty[this.questionRow]);
        this.isQuestionLoaded = true; 
      }
      
      
      
  },
    async fetchTopics() {
      var isRes = false;
      while(!isRes){
        const response = await fetch("https://opentdb.com/api_category.php");
        const data = await response.json();
        if(data["response_code"] == 5){
          continue;
        }
        isRes = true;
        const idsToRemove = [21, 27, 30, 32, 13];
        var filteredList = data["trivia_categories"].filter(item => !idsToRemove.includes(item.id));
        var shuffledList = this.shuffle(filteredList);
        this.listOfTopics = shuffledList.slice(0, this.numberOfCategories); 
      }
    },
    async fetchApplicationData(){
      await this.fetchTopics();
      await this.assignDoubleJeopardyCells();
      this.fetchFinalJeopardy();
    },
    shuffle(array) {
      for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1)); 
      [array[i], array[j]] = [array[j], array[i]];
    }
    return array;
  },
  async fetchQuestion(id, difficulty){
    var isRes = false;
    while(isRes == false){
      const response = await fetch(" https://opentdb.com/api.php?amount=1&category="+id+"&difficulty="+difficulty+"&type=boolean");
      
      const data = await response.json();
      if(data["response_code"] == 5){
        continue;
      }
      isRes = true;
      return data["results"];
    }
    return []
  },

  promptWager(defaultValue) {
  return new Promise(async (resolve) => {
    let isValid = false;
    let wagerValue = null;

    while (!isValid) {
      const wager = prompt(`Double Jeopardy wager: (if you click on cancel then $${defaultValue}) will be used`);
      
     
      if (wager === null) {
        resolve(defaultValue);
        return;
      }

      wagerValue = parseInt(wager, 10);

      if (!isNaN(wagerValue) && wagerValue > 0 && wagerValue <= this.player_scores[this.player_turn-1]) {
        isValid = true; 
      } else {
        alert(`Invalid wager. Please enter a value up to your balance of $${this.player_scores[this.player_turn-1]}.`);
      }
    }
    resolve(wagerValue);
  });
},

  promptWagerFinalJeopardy(playerIndex){
    return new Promise(async (resolve) => {
    let isValid = false;
    let wagerValue = null;

    while (!isValid) {
      const wager = prompt(`Final Jeopardy wager for Player ${playerIndex}: (if you click on cancel then $${this.player_scores[playerIndex-1]}) will be used`);
      
      
      if (wager === null) {
        resolve(this.player_scores[playerIndex-1]);
        return;
      }

      wagerValue = parseInt(wager, 10);

      if (!isNaN(wagerValue) && wagerValue > 0 && wagerValue <= this.player_scores[playerIndex-1]) {
        isValid = true; 
      } else {
        alert(`Invalid wager. Please enter a value up to your balance of $${this.player_scores[playerIndex-1]}.`);
      }
    }
    resolve(wagerValue);
  });
  },

  jeopardyQuestionAnswered(){
    this.finalJeopadyAnswers.push(this.selectedJeopardyAnswer);
    if(this.selectedJeopardyAnswer == this.finalJeopadyQuestion[0]["correct_answer"]){
        this.isCorrect.push(1);
    }
    else{
      this.isCorrect.push(-1);
    }
    this.selectedJeopardyAnswer = null;
    if(this.finalJeopadyPlayerIndex == this.finalPlayers.length-1){
      this.isGameFinished = true;
      var finalScores = this.player_scores;
      for(i in this.finalPlayers){
        finalScores[this.finalPlayers[i]-1] += this.finalWagerAmounts[i]*this.isCorrect[i];
      }
      var maxScore = Math.max(...finalScores);
      var finalWinners = [];
      for(var i in this.finalPlayers){
        if(finalScores[this.finalPlayers[i]-1] == maxScore){
          finalWinners.push(this.finalPlayers[i]);
        }
      }
      if(finalWinners.length == 1){
        this.gameResult = "Player "+finalWinners[0]+" won!!";
      }
      else{
        this.gameResult = "There's a tie between players "+finalWinners;
      }
    }
    else{
      this.finalJeopadyPlayerIndex++;
    }
  }
  
  }
}
</script>

<template>
  <div class="scoreboard" v-if="!isFinalJeopardy">
    <div v-for="(player, index) in player_scores" :key="player">
      Player {{ index+1 }} score: {{ player_scores[index] }}
    </div>
  </div>
  <table v-if="!isFinalJeopardy">
    <tbody>
    <tr v-if="isTopicLoaded">
      <th v-for="(topic, index) in listOfTopics">{{listOfTopics[index]["name"]}}</th>
      </tr>
    <tr v-for="(row, rowIndex) in [0, 1, 2]" :key="rowIndex">
        <td
          v-for="(col, colIndex) in gridValues[rowIndex]" :key="colIndex" v-on:click="handleCellClick(rowIndex, colIndex)"
          :style="{ color: textColor[rowIndex][colIndex] }"
        >
          {{ gridValues[rowIndex][colIndex] }}
        </td>
      </tr>
      </tbody>
  </table>
  <div class="game-status" v-if="!isFinalJeopardy">
  <div class="player-action" v-if="isQuestionClicked">
    <span class="player-name">Player {{prev_player}}</span> selects <span class="category">{{listOfTopics[questionCol]["name"]}}</span> for <span class="amount">{{currentAmount}}$</span>:
  </div>
  <div v-if="!isQuestionLoaded && isQuestionClicked">Please wait while the question is loading</div>
  <div class="question" v-if="isQuestionLoaded">
  <span v-if="doubleJeopardyCells.includes(questionRow*numberOfCategories+questionCol)" style="color:red"> It's double jeopardy</span>
    <div v-html="selectedQuestionParsing"></div>
    <label class="answer-choice">
      <input type="radio" value="True" v-model="selectedAnswer"> True
    </label>
    <label class="answer-choice">
      <input type="radio" value="False" v-model="selectedAnswer"> False
    </label>
  </div>
  <div class="submit-section"  v-if="isQuestionLoaded">
    <button class="submit-button" @click="questionAnswered()">Submit</button>
  </div>
  <div class="result"  v-if="isQuestionLoaded">
    {{answeredQuestionStatus}}
  </div>
  <div class="next-player" v-if="!isGameFinished">
    <span class="next-action">Player {{player_turn}} to select</span>
  </div>
  <div class="game-result" v-if="isGameFinished">
    <span class="next-action">{{gameResult}}</span>
  </div>
</div>
<div v-if="isFinalJeopardy">
  <div class="question" v-if="!isGameFinished">
  Final Jeopardy Topic is: {{finalJeopadyTopic["name"]}}
  </div>
  <div class="question" v-if="!isGameFinished">
    <div v-html="finalJoepadyquestionParsing"></div>
    <label v-for="(option, index) in finalJeopadyQuestion[0]['choices']" :key="index" class="answer-choice">
      <input type="radio" :value="option" v-model="selectedJeopardyAnswer" />
      {{ option }}
    </label>
  </div>
  <div class="submit-section"  v-if="isFinalJeopardy && !isGameFinished">
    <button class="submit-button" @click="jeopardyQuestionAnswered()">Submit</button>
  </div>
  
    <div class="next-player" v-if="!isGameFinished">
    <span class="next-action">Player {{finalPlayers[finalJeopadyPlayerIndex]}} to select</span>
  </div>

  <div v-if="isGameFinished">
    <div v-for="(player, index) in finalPlayers">
    <div>
    <p>{{player }}'s Answer: {{ finalJeopadyAnswers[index]}} - <strong>{{ isCorrect[index] == 1 ? "CORRECT" : "INCORRECT" }}</strong></p>
    <p>Wagered Amount: ${{ finalWagerAmounts[index] }}</p>
    <p>Updated Balance: ${{ player_scores[player-1]+isCorrect[index]*finalWagerAmounts[index] }}</p>
  </div>
</div>
<div class="game-result" v-if="isGameFinished">
    <span class="next-action">{{gameResult}}</span>
  </div>
  </div>
</div>

</template>

<style scoped>
body {
      font-family: Arial, sans-serif;
      background-color: white;
      color: black;
      display: flex;
      height: 100vh;
    }

.scoreboard {
      display: flex;
      justify-content: space-between;
      margin: 5px;
    }

    .scoreboard div {
      font-size: 20px;
      font-weight: bold;
      color: #00008B;
      padding: 10px;
      backgroundAZAAz-color: white;
      border: 2px solid #00008B;
      border-radius: 5px;
    }


    table {
      border-collapse: collapse;
      margin: 5px;
      background-color: #00008B; 
      color: white;
    }

    th, td {
      border: 2px solid #000000;
      padding: 20px;
      text-align: center;
      font-size: 24px;
    }

    th {
      background-color: #00008B; 
      color: white; 
    }

    td {
      background-color: #00008B;
      font-weight: bold;
    }

    
    td:hover {
      background-color: #1E90FF; 
    }

    .game-status {
  padding: 10px;
  margin-top: 20px;
  background-color: #f1f5f9; 
  border-radius: 8px;
  font-family: Arial, sans-serif;
  border: 2px solid #1e3a8a; 
}

.player-action {
  font-size: 1.2rem;
  margin-bottom: 8px;
  font-weight: bold;
  color: #1e3a8a; 
}

.player-name {
  color: #1e40af; 
}

.category, .amount {
  color: #1e40af; 
}

.question {
  font-size: 1.1rem;
  margin-bottom: 10px;
  color: #374151; 
}

.answer-choice {
  font-weight: bold;
  margin-left: 5px;
  color: #1e3a8a; 
}

.submit-section {
  margin-top: 10px;
}

.submit-button {
  background-color: #1e3a8a; 
  color: white;
  font-size: 1rem;
  padding: 8px 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.result {
  font-size: 1.2rem;
  font-weight: bold;
  color: #1e3a8a; 
}

.next-player {
  margin-top: 10px;
  font-size: 1.1rem;
  font-style: italic;
  color: #374151; 
}

.next-action {
  color: #1e40af; 
}
</style>

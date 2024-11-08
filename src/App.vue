<script>
import Jeopardy from './components/labfetch_act2_sverma89.vue'
export default {
  data() {
    return {
      numberOfPlayers: 0,
      numberOfCategories: 0,
      submitted: false,
    };
  },
  components: {
    Jeopardy
    },
  methods: {
    handleSubmit() {
      if (this.numberOfPlayers && this.numberOfCategories) {
        this.submitted = true;
      } else {
        alert("Please select both the number of players and categories.");
      }
    },
  },
};
</script>

<template>
      <div>
      <div v-if="!submitted">
      <label for="players">Number of players: </label>
      <select v-model="numberOfPlayers" id="players">
        <option disabled value="">Select</option>
        <!-- Hardcoded options for players -->
        <option value="2">2</option>
        <option value="3">3</option>
        <option value="4">4</option>
      </select>
    </div>

    <div v-if="!submitted">
      <label for="categories">Number of categories: </label>
      <select v-model="numberOfCategories" id="categories">
        <option disabled value="">Select</option>
        <!-- Hardcoded options for categories -->
        <option value="3">3</option>
        <option value="4">4</option>
        <option value="5">5</option>
        <option value="6">6</option>
      </select>
    </div>

    <button v-if="!submitted" @click="handleSubmit">Submit</button>

      <div v-if="submitted"><Jeopardy :numberOfPlayers="Number(this.numberOfPlayers)" :numberOfCategories="Number(this.numberOfCategories)"/></div>
      </div>
</template>

<style scoped>
div {
  margin-bottom: 1em;
}
label {
  margin-right: 0.5em;
  color: #1e3a8a;
  font-weight: bold;
}
select {
  padding: 0.3em;
  font-size: 1em;
  border: 2px solid #f59e0b; 
  border-radius: 4px;
  background-color: #eff6ff; 
  color: #1e3a8a;
}
select:focus {
  outline: none;
  border-color: #d97706; 
  background-color: #dbf4ff;
}

button {
  padding: 0.5em 1em;
  margin-top: 1em;
  color: #fff;
  background-color: #1e3a8a; 
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
  transition: background-color 0.3s ease;
}
button:hover {
  background-color: #3b82f6; 
}
button:active {
  background-color: #1e40af; 
}

p {
  color: #d97706;
  font-weight: bold;
}

</style>

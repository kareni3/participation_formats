<template>
  <div id="app">
    <div class="background">
      PARTICIPATION FORMATS
    </div>
    <Main class="main"
    :participationFormatsList="participationFormatsList"
    :questionsList="questionsList"
    :clarifyFormatsQuestions="clarifyFormatsQuestions"
    />
  </div>
</template>

<script>
import Main from './components/main.vue'
import participationFormatsList from './assets/participationFormatsList.json'
import questionsList from './assets/questionsList.json'
import clarifyFormatsQuestions from './assets/clarifyFormatsQuestions.json'

export default {
  name: 'App',
  components: {
    Main
  },
  data() {
    return {
      participationFormatsList: participationFormatsList.participationFormatsList,
      questionsList: questionsList.questionsList,
      clarifyFormatsQuestions: clarifyFormatsQuestions.clarifyFormatsQuestions,
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
}
body {
  margin: 0!important;
  background-color: #00000005;
}
.main {
    background-color: white;
    width: 50%;
    top: 250px;
    min-height: 50%;
    left: 50%;
    transform: translate(-50%, 0);
    position: absolute;
    box-shadow: -12px 10px 30px 5px #00000030;
    border-bottom: 1vh solid #4169e190;
}
.background {
    color: #4169e1;
    font-weight: 900;
    text-align: center;
    font-size: calc(12vh);
}
.background * {
    font-size: 1rem;
    color: red;
}
</style>

<template>
  <div class="yesno">
    <b-form-fieldset
      label="Ask a yes/no question:"
      :feedback="answer"
      :label-cols="3"
      :state="state"
    >

      <b-form-input v-model="question" :state="state"></b-form-input>
    </b-form-fieldset>
    <div class="row justify-content-center" v-show="isQuestion">
      <img width='40%' height='40%' :src="imgSrc">
    </div>
  </div>
</template>

<script>
import _ from 'lodash'
import axios from 'axios'

export default {
  name: 'yesno',
  data () {
    return {
      question: '',
      answer: 'I cannot give you an answer until you ask a question!',
      imgSrc: '',
      isQuestion: ''
    }
  },
  computed: {
    state () {
      if (this.isQuestion === '') {
        return
      }
      return this.isQuestion ? 'success' : 'danger'
    }
  },
  watch: {
    // whenever question changes, this function will run
    question: function (newQuestion) {
      this.answer = 'Waiting for you to stop typing...'
      this.getAnswer()
    }
  },
  methods: {
    // _.debounce is a function provided by lodash to limit how
    // often a particularly expensive operation can be run.
    // In this case, we want to limit how often we access
    // yesno.wtf/api, waiting until the user has completely
    // finished typing before making the ajax request. To learn
    // more about the _.debounce function (and its cousin
    // _.throttle), visit: https://lodash.com/docs#debounce
    getAnswer: _.debounce(
      function () {
        if (this.question.indexOf('?') === -1) {
          this.answer = 'Questions usually contain a question mark. ;-)'
          this.isQuestion = false
          return
        }
        this.answer = 'Thinking...'
        var vm = this
        axios.get('https://yesno.wtf/api')
          .then(function (response) {
            vm.answer = _.capitalize(response.data.answer)
            vm.imgSrc = response.data.image
          })
          .catch(function (error) {
            vm.answer = 'Error! Could not reach the API. ' + error
          })
        this.isQuestion = true
      },
      // This is the number of milliseconds we wait for the
      // user to stop typing.
      500
    )
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>


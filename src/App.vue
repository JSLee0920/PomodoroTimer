<template>
  <div id="app">
    <PomodoroTimer />
  </div>
</template>

<script>
import PomodoroTimer from './components/Pomodoro.vue'

export default {
  name: 'App',
  components: {
    PomodoroTimer
  }
}
</script>
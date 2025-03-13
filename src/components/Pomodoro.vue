<!-- Pomodoro.vue -->
<template>
    <div class="pomodoro-container">
      <h1>Pomodoro Timer</h1>
      
      <div class="timer-display">
        <div class="timer-circle" :style="circleStyle">
          <div class="time">{{ formatTime(currentTime) }}</div>
          <div class="mode">{{ currentMode }}</div>
        </div>
      </div>
      
      <div class="controls">
        <button @click="startTimer" :disabled="isRunning">Start</button>
        <button @click="pauseTimer" :disabled="!isRunning">Pause</button>
        <button @click="resetTimer">Reset</button>
      </div>
      
      <div class="settings">
        <h2>Settings</h2>
        <div class="setting-group">
          <label>Work Duration (minutes)</label>
          <select v-model="workDuration" @change="resetTimer">
            <option :value="25">25 (Standard)</option>
            <option :value="15">15 (Short)</option>
            <option :value="45">45 (Long)</option>
            <option :value="50">50 (Extended)</option>
          </select>
        </div>
        
        <div class="setting-group">
          <label>Short Break (minutes)</label>
          <select v-model="shortBreakDuration" @change="resetTimer">
            <option :value="5">5 (Standard)</option>
            <option :value="3">3 (Brief)</option>
            <option :value="10">10 (Extended)</option>
          </select>
        </div>
        
        <div class="setting-group">
          <label>Long Break (minutes)</label>
          <select v-model="longBreakDuration" @change="resetTimer">
            <option :value="15">15 (Standard)</option>
            <option :value="10">10 (Short)</option>
            <option :value="20">20 (Extended)</option>
            <option :value="30">30 (Extra Long)</option>
          </select>
        </div>
        
        <div class="setting-group">
          <label>Pomodoros before long break</label>
          <select v-model="pomodorosBeforeLongBreak" @change="resetTimer">
            <option :value="4">4 (Standard)</option>
            <option :value="2">2</option>
            <option :value="3">3</option>
            <option :value="5">5</option>
          </select>
        </div>
      </div>
      
      <div class="session-info">
        <h2>Session Info</h2>
        <p>Completed Pomodoros: {{ completedPomodoros }}</p>
        <p>Current Session: {{ sessionCount }} of {{ pomodorosBeforeLongBreak }}</p>
      </div>
      
      <audio ref="alarmSound" src="https://assets.mixkit.co/active_storage/sfx/2869/2869-preview.mp3"></audio>
    </div>
  </template>
  
  <script>
  export default {
    name: 'PomodoroTimer',
    data() {
      return {
        // Timer durations in minutes
        workDuration: 25,
        shortBreakDuration: 5,
        longBreakDuration: 15,
        pomodorosBeforeLongBreak: 4,
        
        // Timer state
        currentTime: 25 * 60, // Initial time in seconds
        initialTime: 25 * 60,
        isRunning: false,
        timerInterval: null,
        currentMode: 'Work',
        
        // Session tracking
        completedPomodoros: 0,
        sessionCount: 1
      };
    },
    computed: {
      circleStyle() {
        // Calculate progress percentage for the circle background
        const progress = (this.currentTime / this.initialTime) * 100;
        const color = this.currentMode === 'Work' ? '#ff6347' : '#4caf50';
        
        return {
          background: `conic-gradient(${color} ${progress}%, #f5f5f5 ${progress}%)`
        };
      }
    },
    methods: {
      startTimer() {
        if (!this.isRunning) {
          this.isRunning = true;
          this.timerInterval = setInterval(() => {
            if (this.currentTime > 0) {
              this.currentTime--;
            } else {
              this.handleTimerComplete();
            }
          }, 1000);
        }
      },
      
      pauseTimer() {
        clearInterval(this.timerInterval);
        this.isRunning = false;
      },
      
      resetTimer() {
        clearInterval(this.timerInterval);
        this.isRunning = false;
        
        if (this.currentMode === 'Work') {
          this.currentTime = this.workDuration * 60;
        } else if (this.currentMode === 'Short Break') {
          this.currentTime = this.shortBreakDuration * 60;
        } else {
          this.currentTime = this.longBreakDuration * 60;
        }
        
        this.initialTime = this.currentTime;
      },
      
      handleTimerComplete() {
        this.playAlarmSound();
        
        // Switch modes based on current status
        if (this.currentMode === 'Work') {
          this.completedPomodoros++;
          
          // Determine if we need a short break or long break
          if (this.sessionCount % this.pomodorosBeforeLongBreak === 0) {
            this.currentMode = 'Long Break';
            this.currentTime = this.longBreakDuration * 60;
          } else {
            this.currentMode = 'Short Break';
            this.currentTime = this.shortBreakDuration * 60;
          }
          
          // Notification for work session completed
          this.showNotification(`Work session completed! Time for a ${this.currentMode.toLowerCase()}.`);
        } else {
          // After any break, go back to work mode
          this.currentMode = 'Work';
          this.currentTime = this.workDuration * 60;
          
          // If we just finished a long break, reset session counter
          if (this.sessionCount >= this.pomodorosBeforeLongBreak) {
            this.sessionCount = 1;
          } else {
            this.sessionCount++;
          }
          
          // Notification for break completed
          this.showNotification('Break completed! Time to get back to work.');
        }
        
        this.initialTime = this.currentTime;
        this.pauseTimer(); // Pause at the end of a session
      },
      
      formatTime(timeInSeconds) {
        const minutes = Math.floor(timeInSeconds / 60);
        const seconds = timeInSeconds % 60;
        return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
      },
      
      playAlarmSound() {
        if (this.$refs.alarmSound) {
          this.$refs.alarmSound.currentTime = 0;
          this.$refs.alarmSound.play();
        }
      },
      
      showNotification(message) {
        // Check if browser supports notifications
        if ('Notification' in window) {
          if (Notification.permission === 'granted') {
            new Notification('Pomodoro Timer', { body: message });
          } else if (Notification.permission !== 'denied') {
            Notification.requestPermission().then(permission => {
              if (permission === 'granted') {
                new Notification('Pomodoro Timer', { body: message });
              }
            });
          }
        }
      }
    },
    mounted() {
      // Request notification permission when component mounts
      if ('Notification' in window && Notification.permission !== 'denied') {
        Notification.requestPermission();
      }
    },
    beforeUnmount() {
      // Clean up interval when component is destroyed
      clearInterval(this.timerInterval);
    }
  };
  </script>
  
  <style scoped>
  .pomodoro-container {
    max-width: 500px;
    margin: 0 auto;
    padding: 20px;
    font-family: Arial, sans-serif;
  }
  
  h1 {
    text-align: center;
    color: #333;
  }
  
  .timer-display {
    display: flex;
    justify-content: center;
    margin: 30px 0;
  }
  
  .timer-circle {
    width: 200px;
    height: 200px;
    border-radius: 50%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
    transition: background 1s linear;
  }
  
  .time {
    font-size: 2.5rem;
    font-weight: bold;
    color: #333;
  }
  
  .mode {
    font-size: 1rem;
    margin-top: 5px;
    color: #555;
  }
  
  .controls {
    display: flex;
    justify-content: center;
    gap: 10px;
    margin-bottom: 30px;
  }
  
  button {
    padding: 10px 20px;
    font-size: 1rem;
    background-color: #4a90e2;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.3s;
  }
  
  button:hover {
    background-color: #357abD;
  }
  
  button:disabled {
    background-color: #cccccc;
    cursor: not-allowed;
  }
  
  .settings, .session-info {
    background-color: #f9f9f9;
    padding: 15px;
    border-radius: 8px;
    margin-bottom: 20px;
  }
  
  h2 {
    margin-top: 0;
    color: #333;
    font-size: 1.2rem;
  }
  
  .setting-group {
    margin-bottom: 10px;
    display: flex;
    flex-direction: column;
  }
  
  label {
    margin-bottom: 5px;
    color: #555;
  }
  
  select {
    padding: 8px;
    border-radius: 4px;
    border: 1px solid #ddd;
    font-size: 0.9rem;
  }
  
  @media (max-width: 500px) {
    .timer-circle {
      width: 150px;
      height: 150px;
    }
    
    .time {
      font-size: 2rem;
    }
  }
  </style>
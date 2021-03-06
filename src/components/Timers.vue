<template>
<div class="container">
  <h1 class="title">Timetrackr</h1>
  <div class="columns" v-for="(t,id) in timer" :key="id">
    <div class="column">
      <input v-model='t.title' size="80" :disabled="t.state === 'stopped' ? false:true">
    </div>
    <div class="column time">
      {{ toHHMMSS(t.runTime) }}
    </div>
    <div class="column">
      <template v-if="deleteMode">
        <template v-if="t.state === 'running'">
          <b-tooltip label="Cannot delete a running timer" position="is-bottom">
            <button class="button is-danger" disabled><b-icon icon="delete"></b-icon></button>
          </b-tooltip>
        </template>
        <template v-else>
            <button class="button is-danger" v-on:click="deleteTimer(id)"><b-icon icon="delete"></b-icon></button>
        </template>
      </template>
      <template v-else>
        <button class="button is-success" v-on:click="start(id)" :disabled="t.state === 'running'"><b-icon icon="play"></b-icon></button>
        <button class="button is-warning" v-on:click="pause(id)" :disabled="t.state !== 'running'"><b-icon icon="pause"></b-icon></button>
        <button class="button is-danger" v-on:click="reset(id)" :disabled="t.state !== 'paused'"><b-icon icon="restart"></b-icon></button>
      </template>
    </div>
  </div>
  <template v-if="deleteMode">
    <button class="button is-primary" v-on:click="disableDeleteMode"><b-icon icon="check"></b-icon><span>Finish</span></button>
  </template>
  <template v-else>
    <button class="button is-primary" v-on:click="addTimer"><b-icon icon="plus"></b-icon></button>
    <button v-if="Object.keys(timer).length !== 0" class="button is-danger" v-on:click="enableDeleteMode"><b-icon icon="delete"></b-icon></button>
  </template>
</div>
</template>

<script>
import { throttle } from 'lodash';

function newTimer() {
  return {
          interval: null,
          title: '',
          runTime: 0,
          state: 'stopped'
        };
}

export default {
  name: 'Timers',
  data () {
    return {
      deleteMode: false,
      timer: {
        0: newTimer(),
      }
    }
  },
  methods: {
    pad: function (time) {
      return (time < 10 ? '0' : '') + time
    },
    toHHMMSS: function (runtime) {
      var hours = Math.floor(runtime / 3600)
      var minutes = Math.floor((runtime - (hours * 3600)) / 60)
      var seconds = runtime - (hours * 3600) - (minutes * 60)

      return this.pad(hours) + ':' + this.pad(minutes) + ':' + this.pad(seconds)
    },
    start: function (id) {
      this.timer[id].interval = setInterval(() => this.stopwatch(id), 1000)
      this.timer[id].state = 'running'
    },
    pause: function (id) {
      clearInterval(this.timer[id].interval)
      this.timer[id].interval = null
      this.timer[id].state = 'paused'
    },
    reset: function (id) {
      this.timer[id].interval = null
      this.timer[id].runTime = 0
      this.timer[id].state = 'stopped'
      this.timer[id].title = ''
    },
    stopwatch: function (id) {
      this.timer[id].runTime += 1
    },
    addTimer: function () {
      const l = Object.keys(this.timer).length;
      this.$set(this.timer, l, newTimer());
    },
    deleteTimer: function (id) {
      this.$delete(this.timer, id);
    },
    enableDeleteMode: function () {
      this.deleteMode = true;
    },
    disableDeleteMode: function () {
      this.deleteMode = false;
    },
    saveState: throttle(function () {
      localStorage.setItem('timetrackr.timer-state', JSON.stringify(this.timer));
    }, 5000, { leading: true, trailing: true }),
    restoreState: function () {
      const previouslySavedState = localStorage.getItem('timetrackr.timer-state');
      if(previouslySavedState !== null) {
        try {
          const restoredState = JSON.parse(previouslySavedState);
          this.timer = restoredState;
          Object.entries(this.timer).forEach(([timerId,timer]) => {
            if(timer.state === 'running') {
              this.start(timerId);
            }
          });
        } catch(e) {
          console.log('Failed to restore previously saved state:', e, previouslySavedState);
          localStorage.setItem('timetrackr.timer-state', JSON.stringify({ 0: newTimer() }));
        }
      }
    },
  },
  created() {
    this.restoreState();
  },
  updated() {    
    this.saveState();
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.columns {
  align-items: center;
}

.time {
  text-align: center;
  font-weight: bold;
  font-size: 120%;
  font-family: monospace;
}

.button {
  margin-inline-end: 5px;
}
</style>

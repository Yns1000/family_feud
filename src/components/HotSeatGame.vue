<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue';
import { Timer, Check, X, SkipForward, RotateCcw, Trophy, Flame, Repeat, Users, ArrowLeft, Pause, Play } from 'lucide-vue-next';
import hotseatData from '../assets/hotseat.json';

const props = defineProps(['teams']);
const emit = defineEmits(['quit']);

const step = ref('selection');
const activeTeamKey = ref(null);
const roundScore = ref(0);
const timeLeft = ref(60);
const currentIndex = ref(0);
const words = ref([...hotseatData].sort(() => 0.5 - Math.random()));
const isPaused = ref(false);
let timerInterval = null;

const currentCard = computed(() => words.value[currentIndex.value]);

const selectTeam = (teamKey) => {
  activeTeamKey.value = teamKey;
  step.value = 'intro';
};

const startGame = () => {
  step.value = 'playing';
  isPaused.value = false;
  timerInterval = setInterval(() => {
    if (!isPaused.value) {
      if (timeLeft.value > 0) {
        timeLeft.value--;
      } else {
        endGame();
      }
    }
  }, 1000);
};

const togglePause = () => {
  isPaused.value = !isPaused.value;
};

const endGame = () => {
  clearInterval(timerInterval);
  step.value = 'end';
};

const nextCard = (pointsToAdd) => {
  if (isPaused.value) return;
  if (pointsToAdd > 0) {
    roundScore.value += pointsToAdd;
    if (activeTeamKey.value) {
      props.teams[activeTeamKey.value].score += pointsToAdd;
    }
  }
  if (currentIndex.value < words.value.length - 1) {
    currentIndex.value++;
  } else {
    endGame();
  }
};

const playAgain = () => {
  roundScore.value = 0;
  timeLeft.value = 60;
  currentIndex.value = 0;
  activeTeamKey.value = null;
  step.value = 'selection';
  isPaused.value = false;
  words.value = [...hotseatData].sort(() => 0.5 - Math.random());
};

onUnmounted(() => clearInterval(timerInterval));
</script>

<template>
  <div class="hotseat-screen" :class="{ 'is-critical': timeLeft <= 10 && step === 'playing' && !isPaused }">
    <div class="header-bar">
      <div class="team-score blue" :class="{ 'active': activeTeamKey === 'team1' }">
        <span class="t-name">{{ teams.team1.name }}</span>
        <span class="t-val">{{ teams.team1.score }}</span>
      </div>
      <div class="center-hud">
        <div class="timer-wrapper" v-if="step === 'playing'">
          <div class="timer-box" :class="{ 'paused-anim': isPaused }">
            <Timer :size="32" />
            <span>{{ timeLeft }}s</span>
            <div v-if="isPaused" class="paused-badge">PAUSED</div>
          </div>
          <button @click="togglePause" class="pause-btn" :title="isPaused ? 'Resume' : 'Pause'">
            <Play v-if="isPaused" fill="currentColor" :size="24" />
            <Pause v-else fill="currentColor" :size="24" />
          </button>
        </div>
        <button @click="emit('quit')" class="quit-btn">EXIT <X :size="20"/></button>
      </div>
      <div class="team-score red" :class="{ 'active': activeTeamKey === 'team2' }">
        <span class="t-name">{{ teams.team2.name }}</span>
        <span class="t-val">{{ teams.team2.score }}</span>
      </div>
    </div>

    <div v-if="step === 'selection'" class="overlay">
      <div class="selection-card">
        <h1>WHO IS IN THE HOT SEAT?</h1>
        <div class="team-buttons">
          <button @click="selectTeam('team1')" class="select-btn blue">
            <Users /> {{ teams.team1.name }}
          </button>
          <div class="vs">OR</div>
          <button @click="selectTeam('team2')" class="select-btn red">
            {{ teams.team2.name }} <Users />
          </button>
        </div>
      </div>
    </div>

    <div v-if="step === 'intro'" class="overlay">
      <div class="intro-card">
        <div class="playing-badge">
          Playing: <strong>{{ teams[activeTeamKey].name }}</strong>
        </div>
        <div class="intro-content">
          <Flame :size="100" color="#e74c3c" class="flame-icon" />
          <h1>READY?</h1>
          <p>Describe the word without saying the 6 forbidden ones!</p>
        </div>
        <div class="intro-actions">
          <button @click="step = 'selection'" class="back-btn">
            <ArrowLeft :size="24" /> BACK
          </button>
          <button @click="startGame" class="start-btn">
            START TIMER ⏱️
          </button>
        </div>
      </div>
    </div>

    <div v-if="step === 'playing'" class="game-card">
      <h1 class="main-word" :class="{ 'blur-word': isPaused }">{{ currentCard.word }}</h1>
      <div class="forbidden-list" :class="{ 'blur-word': isPaused }">
        <div class="forbidden-label">FORBIDDEN WORDS</div>
        <div class="words-grid">
          <div v-for="(word, index) in currentCard.forbidden" :key="index" class="forbidden-word">
            {{ word }}
          </div>
        </div>
      </div>
      <div class="controls">
        <button @click="nextCard(0)" class="ctrl-btn taboo" :disabled="isPaused"><X :size="20" /> TABOO</button>
        <button @click="nextCard(0)" class="ctrl-btn seen" :disabled="isPaused"><Repeat :size="20" /> SEEN</button>
        <button @click="nextCard(0)" class="ctrl-btn skip" :disabled="isPaused"><SkipForward :size="20" /> SKIP</button>
        <button @click="nextCard(5)" class="ctrl-btn correct" :disabled="isPaused"><Check :size="24" /> CORRECT (+5)</button>
      </div>
    </div>

    <div v-if="step === 'end'" class="overlay">
      <div class="end-card">
        <Trophy :size="80" color="#f1c40f" class="trophy-anim" />
        <h2>TIME'S UP!</h2>
        <div class="round-result">
          SCORE FOR THIS ROUND: {{ roundScore }}
        </div>
        <div class="total-now">
          TOTAL TEAM SCORE: {{ teams[activeTeamKey].score }}
        </div>
        <button @click="playAgain" class="restart-btn">
          <RotateCcw :size="24" /> NEXT PLAYER
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.hotseat-screen {
  width: 100vw; height: 100vh;
  display: flex; flex-direction: column; align-items: center;
  background: radial-gradient(circle at center, #2c3e50 0%, #000000 100%);
  color: white; overflow: hidden; position: relative;
  transition: background 0.5s;
}
.hotseat-screen.is-critical {
  background: radial-gradient(circle at center, #c0392b 0%, #500000 100%);
  animation: pulseBg 1s infinite;
}
.header-bar {
  width: 100%; padding: 15px 40px;
  display: flex; justify-content: space-between; align-items: flex-start;
  z-index: 10; height: 100px;
}
.team-score {
  background: rgba(0,0,0,0.5); padding: 10px 20px; border-radius: 15px;
  border: 2px solid rgba(255,255,255,0.1);
  display: flex; flex-direction: column; align-items: center; min-width: 140px;
  transition: all 0.3s; opacity: 0.5;
}
.team-score.active { opacity: 1; transform: scale(1.1); box-shadow: 0 0 20px rgba(255,255,255,0.2); }
.team-score.blue.active { border-color: #3498db; }
.team-score.red.active { border-color: #e74c3c; }
.t-name { font-size: 0.9rem; font-weight: bold; color: #ccc; }
.t-val { font-family: 'Anton'; font-size: 2.5rem; line-height: 1; }
.center-hud { display: flex; flex-direction: column; align-items: center; gap: 10px; }
.timer-wrapper { display: flex; align-items: center; gap: 15px; }
.timer-box {
  font-family: 'Anton'; font-size: 4rem; color: #f1c40f;
  text-shadow: 0 0 10px black; display: flex; align-items: center; gap: 10px;
  position: relative;
}
.paused-anim { opacity: 0.6; animation: blink 1s infinite; }
.paused-badge {
  position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%) rotate(-10deg);
  background: #e74c3c; color: white; font-size: 1.2rem; padding: 2px 10px;
  border: 2px solid white; border-radius: 5px; pointer-events: none;
}
.pause-btn {
  background: rgba(255,255,255,0.2); border: 2px solid rgba(255,255,255,0.3);
  color: white; width: 50px; height: 50px; border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  cursor: pointer; transition: all 0.2s;
}
.pause-btn:hover { background: white; color: #2c3e50; transform: scale(1.1); }
.quit-btn { background: rgba(255,255,255,0.1); border: none; color: white; cursor: pointer; padding: 5px 15px; border-radius: 20px; display: flex; align-items: center; gap: 5px; }
.overlay {
  position: absolute; inset: 0;
  display: flex; align-items: center; justify-content: center;
  background: rgba(0,0,0,0.85); backdrop-filter: blur(8px); z-index: 20;
}
.selection-card, .end-card {
  text-align: center; animation: zoomIn 0.4s;
  background: rgba(255,255,255,0.05); padding: 40px; border-radius: 30px;
  border: 1px solid rgba(255,255,255,0.1); max-width: 90%;
}
.intro-card {
  text-align: center; animation: zoomIn 0.4s;
  background: rgba(20, 20, 20, 0.95); padding: 50px; border-radius: 40px;
  border: 1px solid rgba(255, 255, 255, 0.1); box-shadow: 0 20px 50px rgba(0,0,0,0.5);
  display: flex; flex-direction: column; align-items: center; gap: 30px;
  max-width: 600px; width: 90%;
}
.playing-badge {
  background: #f1c40f; color: black; padding: 8px 20px; border-radius: 50px;
  font-weight: bold; font-size: 1.2rem; font-family: Arial, sans-serif;
  box-shadow: 0 5px 15px rgba(241, 196, 15, 0.3);
}
.intro-content h1 { font-family: 'Anton'; font-size: 5rem; margin: 10px 0; color: white; letter-spacing: 5px; line-height: 1; }
.intro-content p { color: #bdc3c7; font-size: 1.2rem; margin: 0; }
.flame-icon { filter: drop-shadow(0 0 20px rgba(231, 76, 60, 0.6)); animation: bounce 2s infinite; }
.intro-actions { display: flex; align-items: center; justify-content: center; gap: 20px; width: 100%; margin-top: 10px; }
.back-btn {
  background: transparent; border: 2px solid rgba(255, 255, 255, 0.2);
  color: rgba(255, 255, 255, 0.6); padding: 15px 30px; border-radius: 50px;
  cursor: pointer; font-family: 'Anton'; font-size: 1.2rem;
  display: flex; align-items: center; gap: 10px; transition: all 0.2s; height: 60px;
}
.back-btn:hover { border-color: white; color: white; background: rgba(255,255,255,0.05); }
.team-buttons { display: flex; align-items: center; gap: 30px; margin-top: 30px; justify-content: center; flex-wrap: wrap; }
.select-btn {
  font-family: 'Anton'; font-size: 1.5rem; padding: 20px 40px; border-radius: 20px;
  border: none; cursor: pointer; display: flex; flex-direction: column; align-items: center; gap: 10px;
  transition: transform 0.2s; min-width: 180px;
}
.select-btn:hover { transform: scale(1.05); }
.select-btn.blue { background: #3498db; color: white; }
.select-btn.red { background: #e74c3c; color: white; }
.vs { font-family: 'Anton'; font-size: 2rem; color: #7f8c8d; }
.game-card {
  flex: 1; display: flex; flex-direction: column; align-items: center; justify-content: center;
  width: 100%; max-width: 900px; animation: slideUp 0.3s;
  margin-top: -20px; padding-bottom: 20px;
}
.main-word {
  font-family: 'Anton'; font-size: 5rem; margin: 0 0 10px 0;
  text-transform: uppercase; color: white; text-shadow: 0 10px 20px rgba(0,0,0,0.5);
  line-height: 1.1; text-align: center; transition: filter 0.3s;
}
.blur-word { filter: blur(15px); }
.forbidden-list {
  background: rgba(231, 76, 60, 0.15); border: 4px solid #e74c3c;
  padding: 15px 30px; border-radius: 20px; text-align: center; margin-bottom: 20px; width: 100%;
  transition: filter 0.3s;
}
.forbidden-label { font-family: 'Anton'; color: #e74c3c; font-size: 1.2rem; margin-bottom: 10px; letter-spacing: 2px; }
.words-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 5px 30px; }
.forbidden-word { font-family: Arial, sans-serif; font-weight: bold; font-size: 1.8rem; color: #e74c3c; text-transform: uppercase; }
.controls { display: flex; gap: 10px; flex-wrap: wrap; justify-content: center; }
.ctrl-btn {
  font-family: 'Anton'; font-size: 1rem; padding: 12px 20px; border-radius: 50px;
  border: none; cursor: pointer; display: flex; align-items: center; gap: 8px;
  transition: transform 0.1s; white-space: nowrap;
}
.ctrl-btn:active { transform: scale(0.95); }
.ctrl-btn:disabled { opacity: 0.3; cursor: not-allowed; }
.taboo { background: #c0392b; color: white; }
.seen { background: #34495e; color: white; }
.skip { background: #95a5a6; color: white; }
.correct { background: #2ecc71; color: white; font-size: 1.2rem; padding: 12px 30px; }
.round-result { font-family: 'Anton'; font-size: 3rem; color: #2ecc71; margin: 20px 0 10px 0; }
.total-now { font-size: 1.2rem; color: #ccc; margin-bottom: 30px; }
.start-btn, .restart-btn {
  background: #f1c40f; color: black; font-family: 'Anton'; font-size: 2rem;
  padding: 0 40px; height: 60px; border-radius: 50px; border: none; cursor: pointer;
  display: flex; align-items: center; gap: 10px; margin: 0 auto;
  box-shadow: 0 10px 30px rgba(241, 196, 15, 0.3);
}
.start-btn:hover { transform: translateY(-3px); box-shadow: 0 15px 40px rgba(241, 196, 15, 0.5); }
@keyframes zoomIn { from { transform: scale(0.8); opacity: 0; } to { transform: scale(1); opacity: 1; } }
@keyframes slideUp { from { transform: translateY(50px); opacity: 0; } to { transform: translateY(0); opacity: 1; } }
@keyframes bounce { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-10px); } }
@keyframes blink { 0%, 100% { opacity: 0.5; } 50% { opacity: 1; } }
@keyframes pulseBg { 0%, 100% { opacity: 1; } 50% { opacity: 0.8; } }
</style>
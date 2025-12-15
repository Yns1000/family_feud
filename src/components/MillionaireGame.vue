<script setup>
import { ref, computed } from 'vue';
import { Trophy, X, Phone, Users, Divide, LogOut, Wallet } from 'lucide-vue-next';
import millionaireData from '../assets/millionaire.json';

const props = defineProps(['teams']);
const emit = defineEmits(['quit']);

const moneyLadder = millionaireData.team1.map(q => q.value).reverse();

const step = ref('selection');
const activeTeamKey = ref(null);
const currentLevel = ref(0);
const selectedOption = ref(null);
const gameState = ref('waiting');
const lifelines = ref({ fifty: true, phone: true, public: true });
const hiddenOptions = ref([]);
const showAudience = ref(false);
const audienceStats = ref([]);
const showPhoneToast = ref(false);
const initialTeamScore = ref(0);

const activeQuestions = ref([]);

const currentQuestion = computed(() => activeQuestions.value[currentLevel.value]);

const pendingPoints = computed(() => {
  if (currentLevel.value === 0) return 0;
  const secureAmount = activeQuestions.value[currentLevel.value - 1].value;
  return Math.floor(secureAmount / 100);
});

const startRun = (teamKey) => {
  activeTeamKey.value = teamKey;
  initialTeamScore.value = props.teams[teamKey].score;

  activeQuestions.value = millionaireData[teamKey];

  currentLevel.value = 0;
  lifelines.value = { fifty: true, phone: true, public: true };
  resetQuestionState();
  step.value = 'game';
};

const resetQuestionState = () => {
  selectedOption.value = null;
  gameState.value = 'waiting';
  hiddenOptions.value = [];
  showAudience.value = false;
  showPhoneToast.value = false;
};

const selectAnswer = (index) => {
  if (gameState.value !== 'waiting') return;
  if (hiddenOptions.value.includes(index)) return;
  selectedOption.value = index;
  gameState.value = 'selected';
};

const walkAway = () => {
  if (gameState.value !== 'waiting' && gameState.value !== 'selected') return;
  const pointsToAdd = pendingPoints.value;
  props.teams[activeTeamKey.value].score += pointsToAdd;
  step.value = 'walkaway';
};

const validateAnswer = () => {
  if (selectedOption.value === null) return;
  gameState.value = 'revealed';

  const isCorrect = selectedOption.value === currentQuestion.value.correct;

  if (isCorrect) {
    setTimeout(() => {
      if (currentLevel.value < activeQuestions.value.length - 1) {
        currentLevel.value++;
        resetQuestionState();
      } else {
        finishGame(true);
      }
    }, 2000);
  } else {
    setTimeout(() => {
      finishGame(false);
    }, 2000);
  }
};

const finishGame = (wonMillion) => {
  let finalGain = 0;

  if (wonMillion) {
    finalGain = 1000000;
    step.value = 'win';
  } else {
    if (currentLevel.value > 9) finalGain = 32000;
    else if (currentLevel.value > 4) finalGain = 1000;
    else finalGain = 0;
    step.value = 'lose';
  }

  const finalPoints = Math.floor(finalGain / 100);
  props.teams[activeTeamKey.value].score = initialTeamScore.value + finalPoints;
};

const useFifty = () => {
  if (!lifelines.value.fifty) return;
  lifelines.value.fifty = false;
  const correct = currentQuestion.value.correct;
  const wrongs = [0, 1, 2, 3].filter(i => i !== correct);
  wrongs.splice(Math.floor(Math.random() * wrongs.length), 1);
  hiddenOptions.value = wrongs;
};

const usePublic = () => {
  if (!lifelines.value.public) return;
  lifelines.value.public = false;
  const correctIdx = currentQuestion.value.correct;
  let stats = [0, 0, 0, 0];
  let remaining = 100;
  stats[correctIdx] = Math.floor(Math.random() * 30) + 40;
  remaining -= stats[correctIdx];
  for(let i=0; i<4; i++) {
    if(i !== correctIdx) {
      if (i === 3 || (i === 2 && correctIdx === 3)) stats[i] = remaining;
      else {
        const val = Math.floor(Math.random() * remaining);
        stats[i] = val;
        remaining -= val;
      }
    }
  }
  stats = stats.map(s => s < 0 ? 0 : s);
  audienceStats.value = stats;
  showAudience.value = true;
};

const usePhone = () => {
  if (!lifelines.value.phone) return;
  lifelines.value.phone = false;
  showPhoneToast.value = true;
  setTimeout(() => showPhoneToast.value = false, 10000);
};
</script>

<template>
  <div class="millionaire-screen">

    <div class="header-bar">
      <div class="logo-zone">MILLIONAIRE</div>

      <div class="teams-scores">
        <div class="team-block blue" :class="{ 'playing': activeTeamKey === 'team1' }">
          <span class="name">{{ teams.team1.name }}:</span>
          <span class="real-score">{{ teams.team1.score }}</span>
          <span v-if="step === 'game' && activeTeamKey === 'team1'" class="potential-badge">
            (+{{ pendingPoints }})
          </span>
        </div>

        <span class="vs">|</span>

        <div class="team-block red" :class="{ 'playing': activeTeamKey === 'team2' }">
          <span class="name">{{ teams.team2.name }}:</span>
          <span class="real-score">{{ teams.team2.score }}</span>
          <span v-if="step === 'game' && activeTeamKey === 'team2'" class="potential-badge">
            (+{{ pendingPoints }})
          </span>
        </div>
      </div>

      <button @click="emit('quit')" class="quit-btn"><LogOut :size="18"/> EXIT</button>
    </div>

    <div v-if="step === 'selection'" class="intro-overlay">
      <div class="intro-card">
        <h1>WHO WANTS TO BE A MILLIONAIRE ?</h1>
        <div class="selection-btns">
          <button @click="startRun('team1')" class="sel-btn blue">{{ teams.team1.name }}</button>
          <div class="vs-big">VS</div>
          <button @click="startRun('team2')" class="sel-btn red">{{ teams.team2.name }}</button>
        </div>
      </div>
    </div>

    <div v-if="step === 'game'" class="game-layout">

      <div class="main-area">

        <transition name="slide-fade">
          <div v-if="showPhoneToast" class="phone-toast">
            <Phone :size="32" class="phone-icon-anim" />
            <div class="toast-content">
              <h3>CALLING A TEACHER...</h3>
              <p>You have 30 seconds to ask for help!</p>
            </div>
            <button @click="showPhoneToast = false" class="close-toast"><X :size="16"/></button>
          </div>
        </transition>

        <div class="lifelines-bar">
          <div class="jokers-group">
            <button @click="useFifty" class="joker-btn" :disabled="!lifelines.fifty" title="50:50"><Divide /></button>
            <button @click="usePhone" class="joker-btn" :disabled="!lifelines.phone" title="Phone a Friend"><Phone /></button>
            <button @click="usePublic" class="joker-btn" :disabled="!lifelines.public" title="Ask Audience"><Users /></button>
          </div>

          <button @click="walkAway" class="walk-away-btn" title="Stop & Keep Money">
            <Wallet :size="20" /> TAKE <span class="money-val">{{ pendingPoints }} PTS</span>
          </button>
        </div>

        <div v-if="showAudience" class="audience-graph" @click="showAudience = false">
          <div class="graph-title">AUDIENCE POLL</div>
          <div class="bars-container">
            <div v-for="(stat, i) in audienceStats" :key="i" class="bar-col">
              <div class="bar-val">{{ stat }}%</div>
              <div class="bar-fill" :style="{ height: stat * 2 + 'px' }"></div>
              <div class="bar-label">{{ ['A','B','C','D'][i] }}</div>
            </div>
          </div>
          <div class="graph-hint">(Click to close)</div>
        </div>

        <div class="question-container">
          <div class="question-box">
            <span class="q-num">{{ currentLevel + 1 }}.</span> {{ currentQuestion.question }}
          </div>
        </div>

        <div class="answers-grid">
          <button
              v-for="(opt, index) in currentQuestion.options"
              :key="index"
              class="answer-btn"
              :class="{
              'selected': selectedOption === index,
              'correct': gameState === 'revealed' && index === currentQuestion.correct,
              'wrong': gameState === 'revealed' && selectedOption === index && index !== currentQuestion.correct,
              'hidden': hiddenOptions.includes(index)
            }"
              @click="selectAnswer(index)"
          >
            <span class="opt-letter">{{ ['A','B','C','D'][index] }}:</span>
            <span class="opt-text">{{ opt.substring(3) }}</span>
          </button>
        </div>

        <div v-if="gameState === 'selected'" class="validation-zone">
          <div class="final-text">IS THAT YOUR FINAL ANSWER?</div>
          <button @click="validateAnswer" class="validate-btn">YES, FINAL ANSWER</button>
        </div>

      </div>

      <div class="ladder-area">
        <div
            v-for="(amount, index) in moneyLadder"
            :key="index"
            class="ladder-step"
            :class="{
            'current-step': (14 - index) === currentLevel,
            'passed-step': (14 - index) < currentLevel,
            'safety-step': [1000, 32000, 1000000].includes(amount)
          }"
        >
          <span class="step-num">{{ 15 - index }}</span>
          <div class="step-info">
            <span class="step-val">{{ amount.toLocaleString() }}</span>
            <span class="step-pts" v-if="amount > 0">(+{{ Math.floor(amount/100) }} pts)</span>
          </div>
        </div>
      </div>

    </div>

    <div v-if="step === 'walkaway'" class="end-overlay walkaway">
      <Wallet :size="100" color="#2ecc71" />
      <h1>WISE CHOICE!</h1>
      <p>You secured your earnings.</p>
      <div class="score-summary">+ {{ pendingPoints }} POINTS</div>
      <button @click="step = 'selection'" class="restart-btn">NEW GAME</button>
    </div>

    <div v-if="step === 'win'" class="end-overlay win">
      <Trophy :size="100" color="#f1c40f" />
      <h1>MILLIONAIRE!</h1>
      <p>AMAZING PERFORMANCE</p>
      <div class="score-summary">+ 10,000 POINTS</div>
      <button @click="step = 'selection'" class="restart-btn">NEW GAME</button>
    </div>

    <div v-if="step === 'lose'" class="end-overlay lose">
      <X :size="100" color="#e74c3c" />
      <h1>GAME OVER</h1>
      <p>You fell back to the safety net.</p>
      <button @click="step = 'selection'" class="restart-btn">TRY AGAIN</button>
    </div>

  </div>
</template>

<style scoped>
.millionaire-screen {
  width: 100vw; height: 100vh;
  background: radial-gradient(circle at center, #020024 0%, #090979 35%, #000000 100%);
  display: flex; flex-direction: column;
  color: white; font-family: 'Arial', sans-serif; overflow: hidden;
}

.header-bar {
  display: flex; justify-content: space-between; align-items: center;
  padding: 10px 30px; background: rgba(0,0,0,0.5); border-bottom: 1px solid #444;
  height: 60px;
}
.logo-zone { font-family: 'Anton'; font-size: 1.5rem; color: #f1c40f; letter-spacing: 2px; }

.teams-scores { font-family: 'Anton'; font-size: 1.2rem; display: flex; align-items: center; }
.team-block { display: flex; align-items: center; gap: 8px; padding: 5px 15px; border-radius: 20px; transition: all 0.3s; opacity: 0.6; }
.team-block.playing { opacity: 1; background: rgba(255,255,255,0.1); border: 1px solid rgba(255,255,255,0.3); }
.blue { color: #3498db; } .red { color: #e74c3c; } .vs { margin: 0 10px; color: #555; }

.potential-badge {
  background: #f39c12; color: black; font-size: 0.9rem; padding: 2px 8px; border-radius: 10px;
  animation: pulse 1.5s infinite; font-family: Arial, sans-serif; font-weight: bold;
}

.quit-btn { background: none; border: 1px solid #555; color: #aaa; padding: 5px 15px; border-radius: 20px; cursor: pointer; display: flex; align-items: center; gap: 5px; }
.quit-btn:hover { border-color: white; color: white; }

.intro-overlay, .end-overlay {
  position: absolute; inset: 0; background: rgba(0,0,0,0.9); z-index: 50;
  display: flex; flex-direction: column; align-items: center; justify-content: center;
  animation: fadeIn 0.5s;
}
.intro-card { text-align: center; }
.intro-card h1 { font-family: 'Anton'; font-size: 3.5rem; margin-bottom: 40px; color: #f1c40f; }
.selection-btns { display: flex; gap: 30px; align-items: center; justify-content: center; }
.sel-btn { font-family: 'Anton'; font-size: 1.5rem; padding: 20px 40px; border-radius: 50px; border: none; cursor: pointer; transition: transform 0.2s; color: white; }
.sel-btn:hover { transform: scale(1.1); }
.sel-btn.blue { background: #3498db; } .sel-btn.red { background: #e74c3c; }
.vs-big { font-family: 'Anton'; font-size: 2rem; color: #555; }

.game-layout { display: flex; flex: 1; height: calc(100vh - 60px); }
.main-area { flex: 1; display: flex; flex-direction: column; align-items: center; justify-content: center; position: relative; }
.ladder-area { width: 280px; background: rgba(0,0,0,0.3); padding: 20px; display: flex; flex-direction: column; justify-content: center; border-left: 2px solid #f1c40f; }

.ladder-step {
  display: flex; justify-content: space-between; padding: 2px 10px;
  color: #f39c12; font-family: 'Anton'; font-size: 1.1rem; margin: 1px 0;
  border-radius: 5px;
}
.ladder-step.safety-step { color: white; }
.ladder-step.passed-step { color: #555; }
.ladder-step.current-step { background: #f39c12; color: black; box-shadow: 0 0 10px #f39c12; transform: scale(1.05); }
.step-info { display: flex; gap: 10px; align-items: center; }
.step-pts { font-size: 0.8rem; font-family: Arial; opacity: 0.7; color: #ccc; font-weight: normal; }
.current-step .step-pts { color: black; opacity: 1; font-weight: bold; }

.lifelines-bar {
  position: absolute; top: 20px; width: 90%;
  display: flex; justify-content: space-between; align-items: center; z-index: 20;
}
.jokers-group { display: flex; gap: 10px; }

.joker-btn {
  background: black; border: 2px solid #f1c40f; color: #f1c40f; border-radius: 50%;
  width: 50px; height: 50px; display: flex; align-items: center; justify-content: center;
  cursor: pointer; transition: all 0.2s;
}
.joker-btn:hover:not(:disabled) { background: #f1c40f; color: black; }
.joker-btn:disabled { border-color: #555; color: #555; cursor: not-allowed; }
.joker-btn:disabled::after { content: 'X'; position: absolute; font-size: 3rem; color: red; opacity: 0.8; }

.walk-away-btn {
  background: rgba(231, 76, 60, 0.2); border: 2px solid #e74c3c; color: #e74c3c;
  padding: 8px 20px; border-radius: 30px; font-family: 'Anton'; cursor: pointer;
  display: flex; align-items: center; gap: 10px; transition: all 0.2s;
}
.walk-away-btn:hover { background: #e74c3c; color: white; }
.money-val { font-size: 1.2rem; }

.phone-toast {
  position: absolute; top: 80px; background: #e74c3c; color: white;
  padding: 15px 25px; border-radius: 10px; display: flex; align-items: center; gap: 15px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.5); z-index: 100; border: 2px solid white;
}
.phone-icon-anim { animation: ring 0.5s infinite; }
.toast-content h3 { margin: 0; font-family: 'Anton'; letter-spacing: 1px; }
.toast-content p { margin: 5px 0 0; font-size: 0.9rem; }
.close-toast { background: none; border: none; color: white; cursor: pointer; opacity: 0.7; }
.close-toast:hover { opacity: 1; }

.audience-graph {
  position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
  background: rgba(0,0,0,0.95); padding: 30px; border: 2px solid #f1c40f; border-radius: 20px;
  display: flex; flex-direction: column; align-items: center; gap: 20px; z-index: 100;
  cursor: pointer; box-shadow: 0 0 50px rgba(0,0,0,0.8);
}
.graph-title { font-family: 'Anton'; font-size: 1.5rem; color: #f1c40f; }
.bars-container { display: flex; gap: 20px; align-items: flex-end; height: 200px; }
.bar-col { display: flex; flex-direction: column; align-items: center; gap: 5px; height: 100%; justify-content: flex-end; }
.bar-fill { width: 30px; background: #f1c40f; transition: height 1s ease-out; }
.bar-val { font-size: 0.8rem; font-weight: bold; } .bar-label { font-weight: bold; color: #f1c40f; }
.graph-hint { font-size: 0.8rem; color: #777; margin-top: 10px; }

.question-container { width: 100%; display: flex; justify-content: center; margin-bottom: 30px; }
.question-box {
  background: linear-gradient(to bottom, #000, #003366);
  border: 2px solid #aaa; border-radius: 50px; width: 80%; padding: 30px;
  text-align: center; font-size: 1.8rem; box-shadow: 0 0 20px rgba(255,255,255,0.1);
  position: relative; z-index: 10;
}
.q-num { color: #f1c40f; margin-right: 10px; }

.answers-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; width: 80%; }
.answer-btn {
  background: linear-gradient(to bottom, #000, #003366);
  border: 2px solid #aaa; border-radius: 50px; padding: 20px;
  text-align: left; color: white; cursor: pointer; font-size: 1.2rem;
  display: flex; gap: 10px; transition: all 0.2s; position: relative;
}
.answer-btn:hover:not(.selected) { background: #004080; border-color: #f1c40f; }
.answer-btn.selected { background: #f39c12; color: black; border-color: white; }
.answer-btn.correct { background: #2ecc71; color: black; border-color: white; animation: flashGreen 0.5s infinite; }
.answer-btn.wrong { background: #c0392b; color: white; border-color: white; }
.answer-btn.hidden { opacity: 0; pointer-events: none; }

.opt-letter { color: #f1c40f; font-weight: bold; }
.selected .opt-letter, .correct .opt-letter { color: black; }

.validation-zone { margin-top: 30px; text-align: center; animation: slideUp 0.3s; }
.final-text { font-family: 'Anton'; font-size: 1.5rem; margin-bottom: 10px; letter-spacing: 2px; }
.validate-btn {
  background: #f1c40f; border: none; padding: 10px 40px; font-size: 1.2rem;
  font-family: 'Anton'; cursor: pointer; border-radius: 5px; color: black;
  box-shadow: 0 0 20px rgba(241, 196, 15, 0.5);
}
.validate-btn:hover { transform: scale(1.05); }

.end-overlay h1 { font-family: 'Anton'; font-size: 5rem; margin: 10px 0; }
.score-summary { font-family: 'Anton'; font-size: 3rem; color: #2ecc71; margin-bottom: 20px; }
.restart-btn { background: white; border: none; padding: 15px 40px; font-family: 'Anton'; font-size: 1.5rem; cursor: pointer; border-radius: 50px; color: black; margin-top: 30px; }

@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
@keyframes flashGreen { 0%, 100% { background: #2ecc71; } 50% { background: #27ae60; } }
@keyframes slideUp { from { transform: translateY(20px); opacity: 0; } to { transform: translateY(0); opacity: 1; } }
@keyframes pulse { 0% { transform: scale(1); } 50% { transform: scale(1.1); } 100% { transform: scale(1); } }
@keyframes ring { 0% { transform: rotate(0); } 25% { transform: rotate(15deg); } 75% { transform: rotate(-15deg); } 100% { transform: rotate(0); } }
.slide-fade-enter-active { transition: all 0.3s ease-out; }
.slide-fade-leave-active { transition: all 0.3s cubic-bezier(1, 0.5, 0.8, 1); }
.slide-fade-enter-from, .slide-fade-leave-to { transform: translateY(-20px); opacity: 0; }
</style>
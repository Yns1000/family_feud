<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue';
import { Mic, Coins, X, TriangleAlert, Eye, EyeOff, ArrowRight, Flag, Gavel } from 'lucide-vue-next';
import questionsData from '../assets/questions.json';

const props = defineProps(['teams']);
const emit = defineEmits(['quit']);

const rounds = ref(questionsData.map(round => {
  return {
    ...round,
    answers: round.answers.map(answer => ({ ...answer, revealed: false }))
  };
}));

const currentRoundIndex = ref(0);
const roundScore = ref(0);
const showStrike = ref(false);
const showCheatSheet = ref(false);
const pointsAwarded = ref(false);
const strikesCount = ref(0);
const activeTeam = ref('team1');

const currentRound = computed(() => rounds.value[currentRoundIndex.value]);
const isLastRound = computed(() => currentRoundIndex.value === rounds.value.length - 1);

const revealedCount = computed(() => {
  return currentRound.value.answers.filter(a => a.revealed).length;
});

const setActiveTeam = (teamKey) => {
  activeTeam.value = teamKey;
  strikesCount.value = 0;
};

const revealAnswer = (answer) => {
  if (!answer.revealed) {
    answer.revealed = true;
    roundScore.value += answer.points;
  }
};

const revealRemainingAnswers = () => {
  currentRound.value.answers.forEach(answer => {
    answer.revealed = true;
  });
};

const triggerStrike = () => {
  showStrike.value = true;
  if (strikesCount.value < 3) {
    strikesCount.value++;
  }
  setTimeout(() => showStrike.value = false, 1200);
};

const awardPoints = (teamKey) => {
  if (roundScore.value === 0 && !pointsAwarded.value) return;
  props.teams[teamKey].score += roundScore.value;
  roundScore.value = 0;
  pointsAwarded.value = true;
};

const nextRound = () => {
  if (!isLastRound.value) {
    currentRoundIndex.value++;
    roundScore.value = 0;
    pointsAwarded.value = false;
    strikesCount.value = 0;
    activeTeam.value = 'team1';
  } else {
    emit('quit');
  }
};

const handleKeydown = (e) => {
  if (e.code === 'Space') {
    e.preventDefault();
    triggerStrike();
  }
};

onMounted(() => window.addEventListener('keydown', handleKeydown));
onUnmounted(() => window.removeEventListener('keydown', handleKeydown));
</script>

<template>
  <div class="game-screen">

    <header class="glass-header">
      <div class="team-badge blue" :class="{ 'is-active': activeTeam === 'team1' }" @click="setActiveTeam('team1')">
        <div class="speaking-indicator" v-if="activeTeam === 'team1'">
          <Mic :size="14" stroke-width="3" /> SPEAKING
        </div>
        <div class="team-name">{{ teams.team1.name }}</div>
        <div class="team-total">{{ teams.team1.score }}</div>
        <button v-if="!pointsAwarded && roundScore > 0 && revealedCount >= 3" class="bank-btn" @click.stop="awardPoints('team1')">
          <Coins :size="18" /> BANK HERE
        </button>
      </div>

      <div class="center-panel">
        <button @click="emit('quit')" class="exit-btn">EXIT</button>

        <div class="bank-display">
          <span class="bank-label">BANK</span>
          <span class="bank-amount">{{ roundScore }}</span>
        </div>

        <div class="strikes-container">
          <div class="strike-bulb" :class="{ 'lit': strikesCount >= 1 }"><X :size="28" stroke-width="4" /></div>
          <div class="strike-bulb" :class="{ 'lit': strikesCount >= 2 }"><X :size="28" stroke-width="4" /></div>
          <div class="strike-bulb" :class="{ 'lit': strikesCount >= 3 }"><X :size="28" stroke-width="4" /></div>
        </div>
      </div>

      <div class="team-badge red" :class="{ 'is-active': activeTeam === 'team2' }" @click="setActiveTeam('team2')">
        <div class="speaking-indicator" v-if="activeTeam === 'team2'">
          <Mic :size="14" stroke-width="3" /> SPEAKING
        </div>
        <div class="team-name">{{ teams.team2.name }}</div>
        <div class="team-total">{{ teams.team2.score }}</div>
        <button v-if="!pointsAwarded && roundScore > 0 && revealedCount >= 3" class="bank-btn" @click.stop="awardPoints('team2')">
          <Coins :size="18" /> BANK HERE
        </button>
      </div>
    </header>

    <main class="board-container">

      <div class="notification-area">
        <Transition name="pop">
          <div v-if="revealedCount < 3" class="warning-banner">
            <TriangleAlert :size="28" stroke-width="2.5" />
            <span>FIND AT LEAST 3 ANSWERS TO BANK!</span>
            <TriangleAlert :size="28" stroke-width="2.5" />
          </div>
        </Transition>
      </div>

      <div class="question-banner">
        <span class="round-badge">ROUND {{ currentRoundIndex + 1 }} / {{ rounds.length }}</span>
        <h1>{{ currentRound.question }}</h1>
      </div>

      <div class="answers-grid">
        <div v-for="(answer, index) in currentRound.answers" :key="index" class="card-wrapper" @click="revealAnswer(answer)">
          <div class="card" :class="{ 'flipped': answer.revealed }">
            <div class="face front"><span class="num">{{ index + 1 }}</span></div>
            <div class="face back">
              <span class="text">{{ answer.text }}</span>
              <span class="points">{{ answer.points }}</span>
            </div>
          </div>
        </div>
      </div>
    </main>

    <footer class="game-footer">
      <div class="controls-left">
        <div class="cheat-sheet">
          <button @click="showCheatSheet = !showCheatSheet" class="cheat-toggle">
            <Eye v-if="showCheatSheet" />
            <EyeOff v-else />
          </button>
          <div v-if="showCheatSheet" class="cheat-content">
            <span v-for="(a, i) in currentRound.answers" :key="i"><b>{{i+1}}</b>:{{a.text}} </span>
          </div>
        </div>
      </div>
      <div class="controls-center">
        <button @click="triggerStrike" class="action-btn strike-btn">
          <Gavel :size="20" /> STRIKE
        </button>
        <button @click="revealRemainingAnswers" class="action-btn reveal-btn">
          <Eye :size="20" /> REVEAL
        </button>
        <button @click="nextRound" class="action-btn next-btn" :class="{ 'pulse-animation': pointsAwarded }">
          <span v-if="isLastRound">FINISH</span>
          <span v-else>NEXT</span>
          <Flag v-if="isLastRound" :size="20" />
          <ArrowRight v-else :size="20" />
        </button>
      </div>
      <div class="controls-right"></div>
    </footer>

    <Transition name="strike-anim">
      <div v-if="showStrike" class="strike-overlay"><div class="giant-x">X</div></div>
    </Transition>

  </div>
</template>

<style scoped>
.game-screen {
  width: 100vw; height: 100vh; display: flex; flex-direction: column;
  background-image: linear-gradient(rgba(0,0,0,0.1) 1px, transparent 1px), linear-gradient(90deg, rgba(0,0,0,0.1) 1px, transparent 1px);
  background-size: 50px 50px; overflow: hidden;
}

.glass-header {
  width: 100%; display: flex; justify-content: space-between; align-items: flex-start;
  padding: 15px 40px; background: rgba(255, 255, 255, 0.1); backdrop-filter: blur(10px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
  min-height: 180px;
  box-sizing: border-box;
  z-index: 50;
  position: relative;
  box-shadow: 0 5px 20px rgba(0,0,0,0.2);
}

.center-panel { display: flex; flex-direction: column; align-items: center; flex: 1; position: relative; gap: 10px; }

.exit-btn {
  background: rgba(255,255,255,0.1); border: 1px solid rgba(255,255,255,0.2); color: #aaa;
  font-family: 'Anton'; font-size: 0.8rem; padding: 5px 15px; border-radius: 20px;
  cursor: pointer; transition: all 0.2s;
}
.exit-btn:hover { background: rgba(231, 76, 60, 0.8); color: white; border-color: #e74c3c; }

.team-badge {
  display: flex; flex-direction: column; align-items: center; justify-content: center;
  width: 220px; padding: 10px; background: rgba(0, 0, 0, 0.4);
  border-radius: 15px; border: 3px solid transparent; cursor: pointer; transition: all 0.3s;
  position: relative; opacity: 0.5; transform: scale(0.95); user-select: none;
}
.team-badge.is-active { opacity: 1; transform: scale(1.05); background: rgba(0, 0, 0, 0.7); box-shadow: 0 10px 30px rgba(0,0,0,0.5); z-index: 10; }
.team-badge.blue.is-active { border-color: #3498db; box-shadow: 0 0 20px rgba(52, 152, 219, 0.6); }
.team-badge.red.is-active { border-color: #e74c3c; box-shadow: 0 0 20px rgba(231, 76, 60, 0.6); }

.team-name { font-size: 1.2rem; font-weight: bold; letter-spacing: 1px; color: #fff; pointer-events: none; }
.team-total { font-size: 3rem; font-weight: 800; color: #fff; line-height: 1; text-shadow: 2px 2px 0 #000; pointer-events: none; }
.speaking-indicator {
  position: absolute; top: -12px; background: #f1c40f; color: black; font-weight: bold;
  padding: 2px 10px; border-radius: 10px; font-size: 0.8rem; animation: float 2s infinite;
  display: flex; align-items: center; gap: 5px;
}
.bank-btn {
  margin-top: 5px; background: #27ae60; color: white; border: none;
  padding: 5px 15px; border-radius: 20px; font-weight: bold; cursor: pointer;
  font-family: 'Anton', sans-serif; animation: bounce 1s infinite;
  display: flex; align-items: center; gap: 8px;
}

.bank-display {
  text-align: center; background: linear-gradient(135deg, #2c3e50, #000000);
  padding: 5px 40px; border-radius: 12px; border: 2px solid rgba(255,255,255,0.2);
  box-shadow: 0 5px 15px rgba(0,0,0,0.3);
}
.bank-label { display: block; font-size: 0.8rem; color: #bdc3c7; letter-spacing: 2px; }
.bank-amount { font-size: 3.5rem; font-weight: bold; color: white; }

.strikes-container { display: flex; gap: 20px; margin-top: 5px; }
.strike-bulb {
  font-family: Arial, sans-serif; font-weight: 900; font-size: 2rem;
  color: rgba(255, 255, 255, 0.2); background: rgba(0,0,0,0.3);
  border: 2px solid rgba(255, 255, 255, 0.2); width: 45px; height: 45px;
  display: flex; align-items: center; justify-content: center;
  border-radius: 50%; transition: all 0.2s;
}
.strike-bulb.lit {
  color: #fff; background: #e74c3c; border-color: #c0392b;
  box-shadow: 0 0 20px #e74c3c, 0 0 40px #e74c3c; transform: scale(1.1);
}

/* CONTAINER PRINCIPAL MODIFIÉ : On pousse le contenu vers le bas */
.board-container {
  flex: 1; display: flex; flex-direction: column; align-items: center; justify-content: center;
  width: 100%; max-width: 1400px; margin: 0 auto;
  padding-top: 20px; /* Ajoute de l'espace pour ne pas coller au Header */
  z-index: 1;
}

.notification-area {
  display: flex; align-items: center; justify-content: center; width: 100%;
  margin-bottom: 20px;
  height: 60px;
}

.warning-banner {
  background: #f1c40f; color: black; font-family: 'Anton', sans-serif; font-size: 1.5rem;
  padding: 10px 30px; border: 4px solid black; border-radius: 50px;
  text-transform: uppercase; letter-spacing: 1px;
  box-shadow: 5px 5px 0px black;
  display: flex; align-items: center; gap: 15px;
  z-index: 10;
}

.question-banner {
  background: rgba(0, 0, 0, 0.6); padding: 15px 40px; border-radius: 20px; border: 1px solid rgba(255,255,255,0.1);
  text-align: center; margin-bottom: 20px; backdrop-filter: blur(5px); position: relative; width: 80%;
}
.round-badge { background: #f1c40f; color: #000; padding: 5px 15px; border-radius: 20px; font-weight: bold; font-size: 0.9rem; position: absolute; top: -15px; left: 50%; transform: translateX(-50%); }
.question-banner h1 { margin: 0; font-size: 2.2rem; color: white; text-transform: uppercase; font-family: 'Anton', sans-serif; letter-spacing: 1px; }

.answers-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; width: 80%; }
.card-wrapper:last-child:nth-child(odd) { grid-column: span 2; width: 50%; margin: 0 auto; }
.card-wrapper { height: 80px; perspective: 1000px; cursor: pointer; }
.card { width: 100%; height: 100%; position: relative; transition: transform 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275); transform-style: preserve-3d; }
.card.flipped { transform: rotateX(180deg); }
.face { position: absolute; width: 100%; height: 100%; backface-visibility: hidden; border-radius: 10px; display: flex; align-items: center; justify-content: center; box-shadow: 0 5px 10px rgba(0,0,0,0.3); border: 2px solid rgba(255,255,255,0.1); }
.front { background: linear-gradient(135deg, #005f9e 0%, #003b64 100%); }
.front .num { font-size: 2.5rem; font-family: 'Anton'; color: rgba(255,255,255,0.2); border: 3px solid rgba(255,255,255,0.2); width: 50px; height: 50px; display: flex; align-items: center; justify-content: center; border-radius: 50%; }
.back { background: linear-gradient(135deg, #f1c40f 0%, #f39c12 100%); transform: rotateX(180deg); justify-content: space-between; padding: 0 30px; color: #2c3e50; }
.text { font-size: 1.5rem; font-weight: 800; text-transform: uppercase; }
.points { font-size: 2rem; background: #000; color: #fff; padding: 5px 15px; border-radius: 5px; font-family: 'Anton'; }

.game-footer { padding: 15px 40px; display: flex; justify-content: space-between; align-items: center; background: rgba(0,0,0,0.3); backdrop-filter: blur(5px); width: 100%; box-sizing: border-box; }
.controls-center { display: flex; gap: 20px; }
.action-btn {
  border: none; padding: 10px 30px; border-radius: 50px; font-family: 'Anton';
  font-size: 1.2rem; cursor: pointer; transition: all 0.2s; letter-spacing: 1px;
  display: flex; align-items: center; gap: 10px;
}
.strike-btn { background: rgba(231, 76, 60, 0.2); border: 2px solid #e74c3c; color: #e74c3c; } .strike-btn:hover { background: #e74c3c; color: white; }
.next-btn { background: rgba(255, 255, 255, 0.1); border: 2px solid rgba(255,255,255,0.3); color: rgba(255,255,255,0.5); } .next-btn:hover { background: white; color: #2c3e50; }
.reveal-btn { background: rgba(155, 89, 182, 0.2); border: 2px solid #9b59b6; color: #9b59b6; } .reveal-btn:hover { background: #9b59b6; color: white; }
.pulse-animation { background: #2ecc71; color: white; border-color: #2ecc71; box-shadow: 0 0 0 0 rgba(46, 204, 113, 0.7); animation: pulse-green 2s infinite; }

.cheat-sheet { display: flex; align-items: center; gap: 10px; }
.cheat-toggle { background: none; border: none; cursor: pointer; opacity: 0.5; color: white; display: flex; align-items: center; }
.cheat-content { font-size: 0.9rem; color: #aaa; background: rgba(0,0,0,0.8); padding: 5px 15px; border-radius: 10px; display: flex; gap: 10px; }
.strike-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.7); display: flex; justify-content: center; align-items: center; z-index: 100; backdrop-filter: blur(5px); }
.giant-x { font-size: 30rem; font-weight: 900; color: #ff0000; text-shadow: 0 0 50px red, 0 0 100px red; animation: shake 0.5s cubic-bezier(.36,.07,.19,.97) both; }

@keyframes float { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-3px); } }
@keyframes bounce { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-5px); } }
@keyframes pulse-green { 0% { transform: scale(0.95); box-shadow: 0 0 0 0 rgba(46, 204, 113, 0.7); } 70% { transform: scale(1); box-shadow: 0 0 0 20px rgba(46, 204, 113, 0); } 100% { transform: scale(0.95); box-shadow: 0 0 0 0 rgba(46, 204, 113, 0); } }
@keyframes shake { 10%, 90% { transform: translate3d(-1px, 0, 0); } 20%, 80% { transform: translate3d(2px, 0, 0); } 30%, 50%, 70% { transform: translate3d(-4px, 0, 0); } 40%, 60% { transform: translate3d(4px, 0, 0); } }
</style>
<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue';
import { Mic, Coins, X, TriangleAlert, Eye, EyeOff, ArrowRight, Flag, Gavel, LogOut } from 'lucide-vue-next';
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

    <button @click="emit('quit')" class="top-exit-btn">
      <LogOut :size="20" /> EXIT
    </button>

    <header class="game-header">

      <div class="team-panel blue" :class="{ 'active': activeTeam === 'team1' }" @click="setActiveTeam('team1')">
        <div class="mic-badge" v-if="activeTeam === 'team1'">
          <Mic :size="14" /> SPEAKING
        </div>
        <div class="t-name">{{ teams.team1.name }}</div>
        <div class="t-score">{{ teams.team1.score }}</div>
        <button v-if="!pointsAwarded && roundScore > 0 && revealedCount >= 3" class="bank-btn" @click.stop="awardPoints('team1')">
          <Coins :size="16" /> BANK
        </button>
      </div>

      <div class="center-display">
        <div class="bank-box">
          <span class="bank-label">BANK</span>
          <span class="bank-val">{{ roundScore }}</span>
        </div>

        <div class="strikes-row">
          <div class="x-mark" :class="{ 'lit': strikesCount >= 1 }">X</div>
          <div class="x-mark" :class="{ 'lit': strikesCount >= 2 }">X</div>
          <div class="x-mark" :class="{ 'lit': strikesCount >= 3 }">X</div>
        </div>
      </div>

      <div class="team-panel red" :class="{ 'active': activeTeam === 'team2' }" @click="setActiveTeam('team2')">
        <div class="mic-badge" v-if="activeTeam === 'team2'">
          <Mic :size="14" /> SPEAKING
        </div>
        <div class="t-name">{{ teams.team2.name }}</div>
        <div class="t-score">{{ teams.team2.score }}</div>
        <button v-if="!pointsAwarded && roundScore > 0 && revealedCount >= 3" class="bank-btn" @click.stop="awardPoints('team2')">
          <Coins :size="16" /> BANK
        </button>
      </div>

    </header>

    <main class="board-area">

      <div class="alert-container">
        <Transition name="pop">
          <div v-if="revealedCount < 3" class="alert-banner">
            <TriangleAlert :size="24" />
            <span>FIND 3 ANSWERS TO BANK</span>
            <TriangleAlert :size="24" />
          </div>
        </Transition>
      </div>

      <div class="question-box">
        <div class="round-tag">ROUND {{ currentRoundIndex + 1 }} / {{ rounds.length }}</div>
        <h1>{{ currentRound.question }}</h1>
      </div>

      <div class="answers-grid">
        <div v-for="(answer, index) in currentRound.answers" :key="index" class="flip-card" @click="revealAnswer(answer)">
          <div class="inner" :class="{ 'flipped': answer.revealed }">
            <div class="front">
              <span class="number">{{ index + 1 }}</span>
            </div>
            <div class="back">
              <span class="ans-text">{{ answer.text }}</span>
              <span class="ans-points">{{ answer.points }}</span>
            </div>
          </div>
        </div>
      </div>

    </main>

    <footer class="game-footer">

      <div class="left-tools">
        <button @click="showCheatSheet = !showCheatSheet" class="icon-btn" title="Toggle Answers">
          <Eye v-if="showCheatSheet" />
          <EyeOff v-else />
        </button>
        <div v-if="showCheatSheet" class="cheat-list">
          <span v-for="(a, i) in currentRound.answers" :key="i">{{ i+1 }}:{{ a.text }} </span>
        </div>
      </div>

      <div class="center-actions">
        <button @click="triggerStrike" class="action-btn strike">
          <Gavel :size="20" /> STRIKE
        </button>
        <button @click="revealRemainingAnswers" class="action-btn reveal">
          <Eye :size="20" /> REVEAL ALL
        </button>
        <button @click="nextRound" class="action-btn next" :class="{ 'pulse': pointsAwarded }">
          <span v-if="isLastRound">FINISH GAME</span>
          <span v-else>NEXT ROUND</span>
          <ArrowRight v-if="!isLastRound" :size="20" />
          <Flag v-else :size="20" />
        </button>
      </div>

      <div class="right-spacer"></div>
    </footer>

    <Transition name="strike-anim">
      <div v-if="showStrike" class="strike-overlay">
        <div class="giant-x">X</div>
      </div>
    </Transition>

  </div>
</template>

<style scoped>
.game-screen {
  width: 100vw; height: 100vh;
  display: flex; flex-direction: column;
  background: radial-gradient(circle at center, #004e92 0%, #000428 100%);
  color: white; font-family: 'Arial', sans-serif; overflow: hidden;
  position: relative;
}

/* --- BOUTON EXIT --- */
.top-exit-btn {
  position: absolute; top: 20px; right: 20px;
  background: rgba(255,255,255,0.1); border: 1px solid rgba(255,255,255,0.2);
  color: #aaa; padding: 8px 15px; border-radius: 20px;
  cursor: pointer; display: flex; align-items: center; gap: 5px;
  font-family: 'Anton'; font-size: 0.9rem; z-index: 100;
  transition: all 0.2s;
}
.top-exit-btn:hover { background: #c0392b; color: white; border-color: #c0392b; }

/* --- HEADER --- */
.game-header {
  display: flex; justify-content: space-between; align-items: flex-start;
  padding: 20px 60px; height: 160px;
}

.team-panel {
  width: 250px; padding: 15px; background: rgba(0,0,0,0.3);
  border-radius: 15px; border: 2px solid transparent;
  display: flex; flex-direction: column; align-items: center;
  position: relative; transition: all 0.3s; opacity: 0.6; cursor: pointer;
}
.team-panel.active { opacity: 1; transform: scale(1.05); background: rgba(0,0,0,0.6); box-shadow: 0 10px 30px rgba(0,0,0,0.5); }
.team-panel.blue.active { border-color: #3498db; }
.team-panel.red.active { border-color: #e74c3c; }

.mic-badge {
  position: absolute; top: -10px; background: #f1c40f; color: black;
  font-size: 0.7rem; font-weight: bold; padding: 2px 10px; border-radius: 10px;
  display: flex; align-items: center; gap: 5px;
}
.t-name { font-weight: bold; margin-bottom: 5px; letter-spacing: 1px; }
.t-score { font-family: 'Anton'; font-size: 3rem; line-height: 1; }
.bank-btn {
  margin-top: 10px; background: #27ae60; border: none; color: white;
  padding: 5px 20px; border-radius: 20px; font-weight: bold; cursor: pointer;
  display: flex; align-items: center; gap: 5px; animation: bounce 1s infinite;
}

.center-display {
  display: flex; flex-direction: column; align-items: center; gap: 10px;
}
.bank-box {
  background: linear-gradient(135deg, #34495e, #2c3e50);
  padding: 10px 50px; border-radius: 10px; border: 2px solid #7f8c8d;
  text-align: center; box-shadow: 0 5px 15px rgba(0,0,0,0.4);
}
.bank-label { display: block; font-size: 0.8rem; color: #bdc3c7; letter-spacing: 2px; }
.bank-val { font-family: 'Anton'; font-size: 3.5rem; line-height: 1; }

.strikes-row { display: flex; gap: 15px; }
.x-mark {
  font-family: 'Anton'; font-size: 2rem; color: #444;
  background: rgba(0,0,0,0.3); width: 40px; height: 40px;
  display: flex; align-items: center; justify-content: center;
  border-radius: 5px; border: 2px solid #444; transition: all 0.2s;
}
.x-mark.lit {
  color: #fff; background: #c0392b; border-color: #e74c3c;
  box-shadow: 0 0 15px #e74c3c; transform: scale(1.1);
}

/* --- BOARD AREA --- */
.board-area {
  flex: 1; display: flex; flex-direction: column; align-items: center; justify-content: center;
  gap: 20px; width: 100%; max-width: 1200px; margin: 0 auto;
}

.alert-container { height: 50px; display: flex; align-items: center; justify-content: center; }
.alert-banner {
  background: #f1c40f; color: black; padding: 8px 30px; border-radius: 30px;
  font-weight: bold; font-family: 'Anton'; letter-spacing: 1px; display: flex; align-items: center; gap: 10px;
  box-shadow: 0 5px 10px rgba(0,0,0,0.3); border: 2px solid black;
}

.question-box {
  background: rgba(0,0,0,0.6); padding: 20px 40px; border-radius: 20px;
  border: 1px solid rgba(255,255,255,0.1); position: relative; text-align: center; width: 80%;
}
.round-tag {
  position: absolute; top: -12px; left: 50%; transform: translateX(-50%);
  background: #3498db; color: white; font-weight: bold; font-size: 0.8rem;
  padding: 4px 15px; border-radius: 10px;
}
.question-box h1 { margin: 0; font-family: 'Anton'; font-size: 2.2rem; text-transform: uppercase; letter-spacing: 1px; }

.answers-grid {
  display: grid; grid-template-columns: 1fr 1fr; gap: 15px; width: 90%;
}
.flip-card { height: 70px; perspective: 1000px; cursor: pointer; }
.flip-card:last-child:nth-child(odd) { grid-column: span 2; width: 60%; margin: 0 auto; }

.inner {
  position: relative; width: 100%; height: 100%; transition: transform 0.6s;
  transform-style: preserve-3d;
}
.inner.flipped { transform: rotateX(180deg); }

.front, .back {
  position: absolute; width: 100%; height: 100%; backface-visibility: hidden;
  border-radius: 10px; display: flex; align-items: center; justify-content: center;
  border: 2px solid rgba(255,255,255,0.2); box-shadow: 0 4px 0 rgba(0,0,0,0.3);
}
.front { background: linear-gradient(135deg, #003b64, #001f3f); }
.front .number {
  background: #001f3f; color: white; width: 40px; height: 40px;
  border-radius: 50%; display: flex; align-items: center; justify-content: center;
  font-family: 'Anton'; font-size: 1.5rem; border: 2px solid #3498db;
}
.back {
  background: linear-gradient(135deg, #f1c40f, #f39c12); color: black;
  transform: rotateX(180deg); justify-content: space-between; padding: 0 30px;
}
.ans-text { font-family: 'Anton'; font-size: 1.6rem; text-transform: uppercase; }
.ans-points { font-family: 'Anton'; font-size: 1.6rem; background: black; color: white; padding: 2px 12px; border-radius: 5px; }

/* --- FOOTER --- */
.game-footer {
  height: 80px; background: rgba(0,0,0,0.4); backdrop-filter: blur(10px);
  display: flex; align-items: center; justify-content: space-between; padding: 0 40px;
}
.left-tools, .right-spacer { width: 200px; }
.center-actions { display: flex; gap: 20px; }

.action-btn {
  border: none; padding: 10px 25px; border-radius: 30px; font-family: 'Anton';
  font-size: 1.1rem; cursor: pointer; display: flex; align-items: center; gap: 8px;
  transition: transform 0.2s;
}
.action-btn:hover { transform: translateY(-3px); }
.strike { background: #c0392b; color: white; }
.reveal { background: #8e44ad; color: white; }
.next { background: white; color: #2c3e50; }
.pulse { animation: pulseGreen 1.5s infinite; background: #2ecc71; color: white; }

.icon-btn { background: none; border: 1px solid #555; color: #aaa; padding: 8px; border-radius: 50%; cursor: pointer; }
.icon-btn:hover { color: white; border-color: white; }
.cheat-list { position: absolute; bottom: 80px; left: 40px; background: rgba(0,0,0,0.9); padding: 10px; border-radius: 10px; color: #aaa; font-size: 0.8rem; }

/* --- OVERLAY --- */
.strike-overlay {
  position: fixed; inset: 0; background: rgba(0,0,0,0.8);
  display: flex; justify-content: center; align-items: center; z-index: 9999;
}
.giant-x { font-size: 25rem; font-family: 'Anton'; color: #c0392b; animation: shake 0.5s; text-shadow: 0 0 50px red; }

@keyframes bounce { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-5px); } }
@keyframes pulseGreen { 0% { box-shadow: 0 0 0 0 rgba(46, 204, 113, 0.7); } 70% { box-shadow: 0 0 0 15px rgba(46, 204, 113, 0); } }
@keyframes shake { 10%, 90% { transform: translate3d(-2px, 0, 0); } 20%, 80% { transform: translate3d(4px, 0, 0); } 30%, 50%, 70% { transform: translate3d(-8px, 0, 0); } 40%, 60% { transform: translate3d(8px, 0, 0); } }
</style>
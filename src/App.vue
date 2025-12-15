<script setup>
import { ref, reactive, watch, onMounted } from 'vue';
import HomeScreen from './components/HomeScreen.vue';
import RulesScreen from "@/components/RulesScreen.vue";
import SetupScreen from './components/SetupScreen.vue';
import GameScreen from './components/GameScreen.vue';
import EndScreen from './components/EndScreen.vue';
import HotSeatGame from './components/HotSeatGame.vue';

const gameState = ref('home');
const targetGame = ref('feud');

const savedData = sessionStorage.getItem('ff_teams');
const initialTeams = savedData ? JSON.parse(savedData) : {
  team1: { name: '', score: 0 },
  team2: { name: '', score: 0 }
};

const teams = reactive(initialTeams);

watch(teams, (newVal) => {
  sessionStorage.setItem('ff_teams', JSON.stringify(newVal));
}, { deep: true });

const prepareFeud = () => {
  targetGame.value = 'feud';
  if (teams.team1.name && teams.team2.name) {
    gameState.value = 'rules';
  } else {
    gameState.value = 'setup';
  }
};

const prepareHotSeat = () => {
  targetGame.value = 'hotseat';
  if (teams.team1.name && teams.team2.name) {
    gameState.value = 'hotseat';
  } else {
    gameState.value = 'setup';
  }
};

const handleSetupBack = () => {
  gameState.value = 'home';
};

const launchGame = () => {
  if (!teams.team1.name) teams.team1.name = "Team 1";
  if (!teams.team2.name) teams.team2.name = "Team 2";

  if (targetGame.value === 'feud') {
    gameState.value = 'game';
  } else {
    gameState.value = 'hotseat';
  }
};

const backToMenu = () => {
  gameState.value = 'home';
};

const hardReset = () => {
  if(confirm("Are you sure? This will delete team names and scores.")) {
    teams.team1.name = '';
    teams.team1.score = 0;
    teams.team2.name = '';
    teams.team2.score = 0;

    sessionStorage.removeItem('ff_teams');

    gameState.value = 'home';
  }
};
</script>

<template>
  <div class="app-container">

    <HomeScreen
        v-if="gameState === 'home'"
        :teams="teams"
        @go-to-setup="prepareFeud"
        @go-to-hotseat="prepareHotSeat"
        @reset-data="hardReset"
    />

    <RulesScreen
        v-else-if="gameState === 'rules'"
        @back="gameState = 'home'"
        @next="gameState = 'game'"
    />

    <SetupScreen
        v-else-if="gameState === 'setup'"
        :teams="teams"
        @back="handleSetupBack"
        @start-game="launchGame"
    />

    <GameScreen
        v-else-if="gameState === 'game'"
        :teams="teams"
        @quit="gameState = 'end'"
    />

    <HotSeatGame
        v-else-if="gameState === 'hotseat'"
        :teams="teams"
        @quit="backToMenu"
    />

    <EndScreen
        v-else-if="gameState === 'end'"
        :teams="teams"
        @restart="backToMenu"
    />

  </div>
</template>

<style>
body { margin: 0; overflow: hidden; }
.app-container {
  width: 100vw; height: 100vh;
  background: radial-gradient(circle at center, #005f9e 0%, #000046 100%);
  display: flex; justify-content: center; align-items: center;
}
</style>
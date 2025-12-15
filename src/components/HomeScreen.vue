<script setup>
import { Play, Gamepad2, CodeXml, X, Flame, Trash2 } from 'lucide-vue-next';
import logoCentrale from '../assets/ig2i_white.png';
import logoThyltech from '../assets/thyltech_logo_name_white.png';

defineProps(['teams']);
const emit = defineEmits(['go-to-setup', 'go-to-hotseat', 'reset-data']);
</script>

<template>
  <div class="home-screen">

    <div v-if="teams.team1.name && teams.team2.name" class="session-info">
      <div class="current-teams">
        SESSION:
        <span class="blue">
          {{ teams.team1.name }} <span class="score">({{ teams.team1.score }})</span>
        </span>
        <span class="vs-sep">VS</span>
        <span class="red">
          {{ teams.team2.name }} <span class="score">({{ teams.team2.score }})</span>
        </span>
      </div>
      <button @click="emit('reset-data')" class="reset-btn" title="Reset Teams & Scores">
        <Trash2 :size="20" /> RESET
      </button>
    </div>

    <div class="hero-content">

      <div class="icon-floating">
        <Gamepad2 :size="80" color="#f1c40f" stroke-width="1.5" />
      </div>

      <h1 class="game-title">
        <span class="line-1">ENGLISH</span>
        <span class="line-2">GAME STATION</span>
      </h1>

      <div class="edition-badge">
        SELECT A GAME MODE
      </div>

      <div class="game-selection">

        <button @click="emit('go-to-setup')" class="game-btn feud-btn">
          <Play :size="32" fill="currentColor" />
          <div class="btn-text">
            <span class="main">FAMILY FEUD</span>
            <span class="sub">2 TEAMS VS BOARD</span>
          </div>
        </button>

        <button @click="emit('go-to-hotseat')" class="game-btn hotseat-btn">
          <Flame :size="32" fill="currentColor" />
          <div class="btn-text">
            <span class="main">HOT SEAT</span>
            <span class="sub">TABOO CHALLENGE</span>
          </div>
        </button>

      </div>

    </div>

    <footer class="team-footer">
      <div class="logos-container">
        <img :src="logoCentrale" alt="Centrale Lille IG2I" class="footer-logo" />
        <X :size="24" color="#bdc3c7" class="logo-separator" />
        <img :src="logoThyltech" alt="Hyltech" class="footer-logo" />
      </div>

      <div class="team-label">
        <CodeXml :size="20" />
        <span>DEVELOPED BY</span>
      </div>
      <div class="team-names">
        <span class="member">Younes</span>
        <span class="dot">•</span>
        <span class="member">Hakim</span>
        <span class="dot">•</span>
        <span class="member">Tom</span>
      </div>
    </footer>

  </div>
</template>

<style scoped>
.home-screen {
  width: 100vw; height: 100vh;
  display: flex; flex-direction: column; align-items: center; justify-content: center;
  background-image: linear-gradient(rgba(0,0,0,0.1) 1px, transparent 1px), linear-gradient(90deg, rgba(0,0,0,0.1) 1px, transparent 1px);
  background-size: 50px 50px;
  position: relative; overflow: hidden;
}

.session-info {
  position: absolute; top: 20px; right: 20px;
  display: flex; align-items: center; gap: 20px;
  background: rgba(0, 0, 0, 0.6); padding: 10px 25px; border-radius: 30px;
  border: 1px solid rgba(255,255,255,0.2); animation: slideDown 0.5s;
  box-shadow: 0 10px 20px rgba(0,0,0,0.3);
}

.current-teams {
  color: white; font-family: 'Anton'; letter-spacing: 1px; font-size: 1.1rem;
  display: flex; align-items: center; gap: 10px;
}

.blue { color: #3498db; }
.red { color: #e74c3c; }
.score { color: white; opacity: 0.8; font-family: Arial, sans-serif; font-weight: bold; }
.vs-sep { color: #aaa; font-size: 0.8rem; margin: 0 5px; }

.reset-btn {
  background: #e74c3c; color: white; border: none; padding: 8px 15px;
  border-radius: 20px; cursor: pointer; display: flex; align-items: center; gap: 5px;
  font-family: 'Anton'; font-size: 0.9rem; transition: background 0.2s;
}
.reset-btn:hover { background: #c0392b; }

.hero-content {
  text-align: center;
  display: flex; flex-direction: column; align-items: center;
  animation: zoomIn 0.8s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  z-index: 10;
}

.icon-floating {
  margin-bottom: 10px;
  animation: float 3s infinite ease-in-out;
  filter: drop-shadow(0 0 15px rgba(241, 196, 15, 0.5));
}

.game-title {
  display: flex; flex-direction: column;
  font-family: 'Anton', sans-serif;
  line-height: 0.85; margin: 0;
  text-transform: uppercase;
  filter: drop-shadow(0 10px 20px rgba(0,0,0,0.5));
}

.line-1 { font-size: 4rem; color: white; letter-spacing: 5px; }
.line-2 {
  font-size: 7rem; color: #f1c40f; letter-spacing: 5px;
  text-shadow: 4px 4px 0 #000;
}

.edition-badge {
  background: white; color: black;
  font-family: Arial, sans-serif; font-weight: bold; font-size: 1rem; letter-spacing: 3px;
  padding: 5px 20px; border-radius: 50px;
  margin-top: 20px; margin-bottom: 40px;
}

.game-selection {
  display: flex; gap: 30px;
}

.game-btn {
  border: none; padding: 20px 40px; border-radius: 20px;
  cursor: pointer; display: flex; align-items: center; gap: 20px;
  transition: all 0.2s; position: relative; overflow: hidden;
  width: 350px;
}

.btn-text { display: flex; flex-direction: column; text-align: left; }
.btn-text .main { font-family: 'Anton'; font-size: 2rem; line-height: 1; }
.btn-text .sub { font-family: Arial; font-size: 0.8rem; opacity: 0.8; letter-spacing: 1px; margin-top: 5px; }

.feud-btn { background: #2ecc71; color: white; box-shadow: 0 10px 0 #27ae60; }
.feud-btn:hover { transform: translateY(-5px); box-shadow: 0 15px 0 #27ae60; background: #2ecc71; }
.feud-btn:active { transform: translateY(5px); box-shadow: 0 0 0; }

.hotseat-btn { background: #e74c3c; color: white; box-shadow: 0 10px 0 #c0392b; }
.hotseat-btn:hover { transform: translateY(-5px); box-shadow: 0 15px 0 #c0392b; background: #e74c3c; }
.hotseat-btn:active { transform: translateY(5px); box-shadow: 0 0 0; }


.team-footer {
  position: absolute; bottom: 30px;
  background: rgba(0, 0, 0, 0.5); backdrop-filter: blur(5px);
  padding: 15px 40px; border-radius: 20px;
  border: 1px solid rgba(255,255,255,0.1);
  display: flex; flex-direction: column; align-items: center; gap: 5px;
  animation: slideUp 1s ease-out 0.5s backwards;
}

.logos-container { display: flex; align-items: center; justify-content: center; margin-bottom: 10px; }
.footer-logo { max-height: 30px; width: auto; object-fit: contain; }
.logo-separator { margin: 0 15px; }

.team-label { display: flex; align-items: center; gap: 8px; color: #bdc3c7; font-size: 0.8rem; letter-spacing: 2px; }
.team-names { display: flex; align-items: center; gap: 15px; color: white; font-family: 'Anton', sans-serif; font-size: 1.2rem; letter-spacing: 1px; }
.dot { color: #f1c40f; font-size: 1.5rem; line-height: 0; }
.member { text-shadow: 0 2px 5px rgba(0,0,0,0.5); }

@keyframes float { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-10px); } }
@keyframes zoomIn { from { opacity: 0; transform: scale(0.8); } to { opacity: 1; transform: scale(1); } }
@keyframes slideUp { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: translateY(0); } }
@keyframes slideDown { from { opacity: 0; transform: translateY(-20px); } to { opacity: 1; transform: translateY(0); } }
</style>
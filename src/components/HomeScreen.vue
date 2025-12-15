<script setup>
import { Play, Gamepad2, CodeXml, X, Flame, Trash2, Flag } from 'lucide-vue-next';
import logoCentrale from '../assets/ig2i_white.png';
import logoThyltech from '../assets/thyltech_logo_name_white.png';

defineProps(['teams']);
const emit = defineEmits(['go-to-setup', 'go-to-hotseat', 'reset-data', 'go-to-motus', 'go-to-millionaire', 'finish-session']);
</script>

<template>
  <div class="home-screen">

    <div v-if="teams.team1.name && teams.team2.name" class="session-info">
      <div class="current-teams">
        SESSION :
        <span class="blue">
          {{ teams.team1.name }} <span class="score">({{ teams.team1.score }})</span>
        </span>
        <span class="vs-sep">VS</span>
        <span class="red">
          {{ teams.team2.name }} <span class="score">({{ teams.team2.score }})</span>
        </span>
      </div>
      <button @click="emit('finish-session')" class="finish-btn" title="End Session & Show Results">
        <Flag :size="18" /> FINISH
      </button>
      <button @click="emit('reset-data')" class="reset-btn" title="Reset Teams & Scores">
        <Trash2 :size="20" /> RESET
      </button>
    </div>

    <div class="hero-content">

      <div class="logo-area">
        <div class="icon-floating">
          <Gamepad2 :size="80" color="#f1c40f" stroke-width="1.5" />
        </div>

        <h1 class="game-title">
          <span class="line-1">ENGLISH</span>
          <span class="line-2">GAME STATION</span>
        </h1>

        <div class="edition-badge">SELECT A GAME MODE</div>
      </div>

      <div class="game-grid">

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

        <button @click="emit('go-to-motus')" class="game-btn motus-btn">
          <div class="icon-box-motus">M</div>
          <div class="btn-text">
            <span class="main">MOTUS</span>
            <span class="sub">WORD GUESSING</span>
          </div>
        </button>

        <button @click="emit('go-to-millionaire')" class="game-btn millionaire-btn">
          <div class="icon-box-mil">$</div>
          <div class="btn-text">
            <span class="main">MILLIONAIRE</span>
            <span class="sub">QUIZ LADDER</span>
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
      </div>
    </footer>

  </div>
</template>

<style scoped>
/* STRUCTURE GLOBALE */
.home-screen {
  width: 100vw;
  height: 100vh;
  /* Flex Column permet de gérer l'espace vertical proprement */
  display: flex;
  flex-direction: column;
  justify-content: space-between; /* Espace entre le haut, le contenu et le footer */
  align-items: center;
  background-image: linear-gradient(rgba(0,0,0,0.1) 1px, transparent 1px), linear-gradient(90deg, rgba(0,0,0,0.1) 1px, transparent 1px);
  background-size: 50px 50px;
  position: relative;
  overflow-y: auto; /* Permet le scroll si l'écran est petit */
  overflow-x: hidden;
}

.finish-btn {
  background: #f1c40f; color: black; border: none; padding: 6px 15px; margin-right: 10px;
  border-radius: 20px; cursor: pointer; display: flex; align-items: center; gap: 5px;
  font-family: 'Anton'; font-size: 0.8rem; transition: transform 0.2s;
}
.finish-btn:hover { transform: scale(1.05); box-shadow: 0 0 10px rgba(241, 196, 15, 0.5); }

/* SESSION INFO */
.session-info {
  position: absolute; top: 20px; right: 20px;
  display: flex; align-items: center; gap: 20px;
  background: rgba(0, 0, 0, 0.6); padding: 8px 20px; border-radius: 30px;
  border: 1px solid rgba(255,255,255,0.2); z-index: 20;
}
.current-teams { color: white; font-family: 'Anton'; letter-spacing: 1px; font-size: 1rem; display: flex; gap: 10px; }
.blue { color: #3498db; } .red { color: #e74c3c; } .vs-sep { color: #aaa; margin: 0 5px; } .score { color: white; opacity: 0.8; }
.reset-btn { background: #e74c3c; color: white; border: none; padding: 5px 12px; border-radius: 20px; cursor: pointer; display: flex; align-items: center; gap: 5px; font-family: 'Anton'; font-size: 0.8rem; }
.reset-btn:hover { background: #c0392b; }

/* CONTENU CENTRAL */
.hero-content {
  flex: 1; /* Prend toute la place disponible */
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 20px 0; /* Un peu d'air en haut et en bas */
  transform: scale(0.95); /* Petite astuce pour tout faire rentrer sur les petits écrans */
}

.logo-area { text-align: center; margin-bottom: 30px; animation: zoomIn 0.8s; }
.icon-floating { margin-bottom: 5px; animation: float 3s infinite ease-in-out; filter: drop-shadow(0 0 15px rgba(241, 196, 15, 0.5)); }
.game-title { display: flex; flex-direction: column; font-family: 'Anton', sans-serif; line-height: 0.85; margin: 0; text-transform: uppercase; filter: drop-shadow(0 10px 20px rgba(0,0,0,0.5)); }
.line-1 { font-size: 3.5rem; color: white; letter-spacing: 5px; }
.line-2 { font-size: 6rem; color: #f1c40f; letter-spacing: 5px; text-shadow: 4px 4px 0 #000; }
.edition-badge { background: white; color: black; font-family: Arial, sans-serif; font-weight: bold; font-size: 0.9rem; letter-spacing: 3px; padding: 5px 20px; border-radius: 50px; margin-top: 15px; display: inline-block; }

/* GRILLE DES BOUTONS (LE 2x2 STRICT) */
.game-grid {
  display: grid;
  grid-template-columns: 1fr 1fr; /* 2 colonnes égales */
  gap: 25px;
  animation: slideUp 0.8s;
}

.game-btn {
  border: none; padding: 15px 30px; border-radius: 20px;
  cursor: pointer; display: flex; align-items: center; gap: 15px;
  transition: all 0.2s; position: relative; overflow: hidden;
  width: 320px; /* Largeur fixe pour uniformité */
  height: 90px;
}

.btn-text { display: flex; flex-direction: column; text-align: left; }
.btn-text .main { font-family: 'Anton'; font-size: 1.8rem; line-height: 1; }
.btn-text .sub { font-family: Arial; font-size: 0.75rem; opacity: 0.8; letter-spacing: 1px; margin-top: 4px; }

/* COULEURS DES JEUX */
.feud-btn { background: #27ae60; color: white; box-shadow: 0 8px 0 #219150; }
.feud-btn:hover { transform: translateY(-4px); box-shadow: 0 12px 0 #219150; filter: brightness(1.1); }
.feud-btn:active { transform: translateY(4px); box-shadow: 0 0 0; }

.hotseat-btn { background: #c0392b; color: white; box-shadow: 0 8px 0 #96281b; }
.hotseat-btn:hover { transform: translateY(-4px); box-shadow: 0 12px 0 #96281b; filter: brightness(1.1); }
.hotseat-btn:active { transform: translateY(4px); box-shadow: 0 0 0; }

.motus-btn { background: #2980b9; color: white; box-shadow: 0 8px 0 #2c3e50; }
.motus-btn:hover { transform: translateY(-4px); box-shadow: 0 12px 0 #2c3e50; filter: brightness(1.1); }
.motus-btn:active { transform: translateY(4px); box-shadow: 0 0 0; }

/* STYLE MILLIONAIRE (CORRIGÉ) */
.millionaire-btn {
  /* Dégradé Bleu Nuit -> Violet style TV */
  background: linear-gradient(135deg, #090979 0%, #2c3e50 100%);
  color: white;
  box-shadow: 0 8px 0 #000040;
  border: 1px solid rgba(255,255,255,0.2);
}
.millionaire-btn:hover { transform: translateY(-4px); box-shadow: 0 12px 0 #000040; filter: brightness(1.2); }
.millionaire-btn:active { transform: translateY(4px); box-shadow: 0 0 0; }

/* ICONES PERSONNALISÉES */
.icon-box-motus { width: 35px; height: 35px; background: #e74c3c; color: white; font-family: 'Anton'; font-size: 1.2rem; display: flex; align-items: center; justify-content: center; border-radius: 5px; border: 2px solid white; }
.icon-box-mil { width: 35px; height: 35px; background: #f1c40f; color: black; font-family: 'Anton'; font-size: 1.5rem; display: flex; align-items: center; justify-content: center; border-radius: 50%; border: 2px solid white; }

/* FOOTER */
.team-footer {
  background: rgba(0, 0, 0, 0.5); backdrop-filter: blur(5px);
  padding: 20px 40px; border-radius: 20px 20px 0 0;
  border-top: 1px solid rgba(255,255,255,0.1);
  display: flex; flex-direction: column; align-items: center; gap: 5px;
  width: 100%; max-width: 600px;
  margin-bottom: 0; /* Collé au bas */
}

.logos-container { display: flex; align-items: center; justify-content: center; margin-bottom: 5px; }
.footer-logo { max-height: 25px; width: auto; object-fit: contain; }
.logo-separator { margin: 0 15px; }
.team-label { display: flex; align-items: center; gap: 8px; color: #bdc3c7; font-size: 0.7rem; letter-spacing: 2px; }
.team-names { display: flex; align-items: center; gap: 15px; color: white; font-family: 'Anton', sans-serif; font-size: 1rem; letter-spacing: 1px; }
.dot { color: #f1c40f; font-size: 1.2rem; line-height: 0; }

@keyframes float { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-10px); } }
@keyframes zoomIn { from { opacity: 0; transform: scale(0.8); } to { opacity: 1; transform: scale(1); } }
@keyframes slideUp { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: translateY(0); } }
@keyframes slideDown { from { opacity: 0; transform: translateY(-20px); } to { opacity: 1; transform: translateY(0); } }
</style>
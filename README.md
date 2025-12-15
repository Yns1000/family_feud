# 🎮 Family Feud - English Class Edition

An interactive web game based on the famous TV show "Family Feud", built for our English class project.
Designed to be played in a classroom setting with two teams facing off to guess the top answers to various survey questions.

---

## ✨ Features

* **Interactive Board:** Real-time card flipping animations and score updates.
* **Game Logic:** Complete round management, points banking system, and strike handling (X).
* **Dynamic Data:** Questions are loaded from a JSON file, making it easy to add new levels.
* **Audio/Visual Effects:** "Cartoon" style alerts, neon effects, and confetti celebration for the winner.
* **Customizable Teams:** Set team names before starting the game.
* **Responsive Design:** Optimized for full-screen projection.

## 🛠️ Tech Stack

* **Framework:** [Vue.js 3](https://vuejs.org/) (Composition API)
* **Build Tool:** [Vite](https://vitejs.dev/)
* **Icons:** [Lucide Vue Next](https://lucide.dev/)
* **Styling:** CSS3 (Glassmorphism, Animations, Flexbox/Grid)

---

## 🚀 How to Run

### Prerequisites
Make sure you have [Node.js](https://nodejs.org/) installed on your machine.

### Installation

1.  **Clone the repository** (or extract the project folder):
    ```bash
    git clone [https://github.com/Yns1000/family_feud.git](https://github.com/Yns1000/family_feud.git)
    cd family_feud
    ```

2.  **Install dependencies:**
    ```bash
    npm install
    ```

3.  **Start the development server:**
    ```bash
    npm run dev
    ```

4.  **Open your browser:**
    Click the link shown in the terminal (usually `http://localhost:5173`).

---

## 🕹️ Controls (Host Guide)

As the game host, you control the flow using the mouse and keyboard:

| Action | Control |
| :--- | :--- |
| **Reveal Answer** | **Click** on the numbered card. |
| **Strike (Wrong Answer)** | Press **SPACEBAR** or click the "❌ STRIKE" button. |
| **Give Control** | **Click** on a Team's badge (illuminates the active team). |
| **Bank Points** | Click the green **💰 BANK HERE** button (appears after 3 answers found). |
| **Reveal All** | Click the **👀 REVEAL** button to show remaining answers without points. |
| **Next Round** | Click **NEXT ROUND** (bottom center). |

---

## 👥 Authors & Credits

**Developed by:**
* Younes

**Schools / Partners:**
* Centrale Lille IG2I
* Thyltech

<template>
  <div id="app" @click="startGame">
    <div v-if="isGamePaused" class="game-menu">
      <div class="game-menu__new" v-if="!rocket.health">
        <h2>Score {{ score }}</h2>
        <button class="game-menu__start" @click="newGame()">Try Again</button>
      </div>
      <button v-else class="game-menu__start" @click="startGame">
        I'm ready to save the world
      </button>
    </div>
    <img
      class="rocket"
      src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/t-1/rocket.svg"
      ref="rocket"
      alt="Rocket ship"
    />
    <img
      v-for="star in stars"
      :key="star.id"
      :ref="`stars-${star.id}`"
      class="star"
      :style="{ opacity: star.isDead ? 0 : 1 }"
      :src="require(`./assets/${star.img}.svg`)"
    />

    <img
      v-for="bullet in bullets"
      :key="bullet.id"
      :ref="`bullets-${bullet.id}`"
      class="bullet"
      src="./assets/bullet.svg"
    />

    
    <div class="score">Score: {{ score }}</div>
    
    <div class="health">
      <span v-for="index in rocket.health" :key="index">️ ♥</span>
    </div>

    <div class="copy-right">
      Icons made by
      <a href="https://www.flaticon.com/authors/smashicons" title="Smashicons"
        >Smashicons</a
      >
      from
      <a href="https://www.flaticon.com/" title="Flaticon">www.flaticon.com</a>
    </div>

    <input
      ref="control-panel"
      class="control-panel"
      @keyup="keyUp"
      @keydown="keyDown"
    />
  </div>
</template>

<script>
function uuidv4() {
  return "xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx".replace(/[xy]/g, function(c) {
    var r = (Math.random() * 16) | 0,
      v = c == "x" ? r : (r & 0x3) | 0x8;
    return v.toString(16);
  });
}

function getRandomInt(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

function isIntersecting(rect1, rect2) {
  rect1 = rect1.getBoundingClientRect();
  rect2 = rect2.getBoundingClientRect();
  return (
    rect1.x < rect2.x + rect2.width &&
    rect1.x + rect1.width > rect2.x &&
    rect1.y < rect2.y + rect2.height &&
    rect1.y + rect1.height > rect2.y
  );
}
export default {
  name: "App",
  data() {
    return {
      gameLoop: null,
      score: 0,
      level: {
        number: 5,
        starSpeed: 5
      },
      isGamePaused: true,
      rocket: {
        health: 5,
        direction: "",
        x: 50,
        y: 90
      },
      bullets: [],
      stars: []
    };
  },
  mounted() {
    this.newGame();
    this.$nextTick(() => {
      this.gameLoop = setInterval(this.startGameLoop, 1000 / 60);
    });
  },
  beforeDestroy() {
    clearInterval(this.gameLoop);
  },
  methods: {
    // GAME LIFE CYCLE
    startGameLoop() {
      if (this.isGamePaused) return true;

      this.physics();
      this.gameLogic();
      this.paintScene();
      this.clean();
    },

    physics() {
      this.moveStars();
      this.moveRocket();
      this.moveBullets();
    },
    gameLogic() {
      this.updateRocketHealth();
      this.checkIfStarIsDead();

      if (this.stars.length == 0) {
        this.newLevel();
      }
    },
    paintScene() {
      this.paint(this.bullets, "bullets");
      this.paint(this.stars, "stars");
      this.paint(this.rocket, "rocket");
    },
    clean() {
      this.stars = this.stars.filter(star => !star.isDead);
      this.bullets = this.bullets.filter(bullet => !bullet.isDead);
    },
    // END LIFE CYCLE

    // STAR
    createStars({ number, speed }) {
      this.stars = Array.from({ length: number }, () => {
        const size = getRandomInt(30, 100);
        return {
          id: uuidv4(),
          isDead: false,
          x: getRandomInt(1, 100),
          y: -10,
          width: size,
          height: size,
          speed: getRandomInt(3, speed),
          img: `${getRandomInt(0, 7)}`
        };
      });
    },
    checkIfStarIsDead() {
      this.bullets.forEach(bullet => {
        const bulletRef = this.$refs[`bullets-${bullet.id}`][0];
        this.stars.forEach(star => {
          const starRef = this.$refs[`stars-${star.id}`][0];
          const isHit = isIntersecting(starRef, bulletRef);
          if (isHit) {
            star.isDead = true;
            bullet.isDead = true;
            this.score++;
          }
        });
      });
    },
    moveStars() {
      const resetStar = star => {
        star.x = getRandomInt(0, 90);
        star.isDead = false;
        star.y = 0;
      };

      this.stars.forEach(star => {
        if (star.y < 100) {
          star.y = star.y + (1 * star.speed) / 10;
        } else {
          resetStar(star);
        }
      });
    },

    // END STAR

    // ROCKET
    updateRocketHealth() {
      const isRocketHit = () => {
        let rocketRef = this.$refs.rocket;
        return this.stars.some(star => {
          const starRef = this.$refs[`stars-${star.id}`][0];
          if (isIntersecting(starRef, rocketRef)) {
            star.isDead = true;
            return true;
          }
        });
      };

      if (isRocketHit()) {
        this.rocket.health--;
        if (this.rocket.health <= 0) {
          this.isGamePaused = true;
        }
      }
    },
    moveRocket() {
      let x = this.direction == "left" ? -1 : this.direction == "right" ? 1 : 0;
      this.rocket.x = this.rocket.x + x;

      if (this.rocket.x < 0) {
        this.rocket.x = 0;
      }

      if (this.rocket.x >= 95) {
        this.rocket.x = 95;
      }
    },
    // END ROCKET

    // GAME
    newGame() {
      this.score = 0;
      this.level = {
        number: 1,
        speed: 1
      };
      this.rocket = {
        health: 5,
        direction: "",
        x: 93,
        y: 90
      };
      this.bullets = [];
      this.stars = [];
    },
    startGame() {
      this.$refs["control-panel"].focus();
      this.isGamePaused = false;
    },

    newLevel() {
      this.level.number++;
      this.level.speed = this.level.speed + 0.4;
      this.createStars(this.level);
    },

    // END GAME

    // BULLETS
    moveBullets() {
      this.bullets.forEach(bullet => {
        bullet.y--;
        if (bullet.y < 0) {
          bullet.isDead = true;
        }
      });
    },

    fireBullet() {
      const bulletLimit = 5;
      if (this.bullets.length > bulletLimit) {
        return;
      }

      this.bullets.push({
        id: uuidv4(),
        x: this.rocket.x + 3,
        y: this.rocket.y,
        isDead: false
      });
    },

    // END BULLET

    paint(elements, refName) {
      if (Array.isArray(elements)) {
        elements.forEach(element => {
          const ref = this.$refs[`${refName}-${element.id}`][0];
          if (ref) {
            ref.style.left = `${element.x}%`;
            ref.style.top = `${element.y}%`;
            ref.style.width = element.width ? `${element.width}px` : "";
            ref.style.height = element.height ? `${element.height}px` : "";
          }
        });
      } else {
        const ref = this.$refs[refName];
        if (ref) {
          ref.style.left = `${elements.x}%`;
          ref.style.top = `${elements.y}%`;
        }
      }
    },

    keyDown(event) {
      switch (event.keyCode) {
        case 65:
        case 37:
          this.direction = "left";
          break;
        case 68:
        case 39:
          this.direction = "right";
          break;
        case 32:
        case 70:
          this.fireBullet();
      }
    },
    keyUp(event) {
      switch (event.keyCode) {
        case 65:
        case 37:
        case 68:
        case 39:
          this.direction = "";
          break;
        case 27:
          this.isGamePaused = !this.isGamePaused;
          break;
      }
    }
  }
};
</script>

<style>
@import url("https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap");
html,
body {
  border: 0;
  padding: 0;
  margin: 0;
  overflow: hidden;
  font-size: 12px;
  font-family: "Press Start 2P", cursive;
}
#app {
  width: 100vw;
  height: 100vh;
  padding: 0;
  position: relative;
  background: #222;
  background-image: radial-gradient(
      2px 2px at 20px 30px,
      #eee,
      rgba(0, 0, 0, 0)
    ),
    radial-gradient(2px 2px at 40px 70px, #fff, rgba(0, 0, 0, 0)),
    radial-gradient(2px 2px at 50px 160px, #ddd, rgba(0, 0, 0, 0)),
    radial-gradient(2px 2px at 90px 40px, #fff, rgba(0, 0, 0, 0)),
    radial-gradient(2px 2px at 130px 80px, #fff, rgba(0, 0, 0, 0)),
    radial-gradient(2px 2px at 160px 120px, #ddd, rgba(0, 0, 0, 0));
  background-repeat: repeat;
  background-size: 200px 200px;
  animation: scrollBackground 90s linear infinite;
}

.rocket {
  position: absolute;
  width: 100px;
  height: 100px;
  top: 80%;
  left: calc(50% - 50px);
}
.star {
  position: absolute;
  width: 0;
  height: 0;
  animation: spin 10s linear infinite;
}

.copy-right {
  position: fixed;
  bottom: 0;
  right: 0px;
  color: white;
  font-size: 10px;
  font-family: arial;
}
.copy-right a {
  color: white;
}

.score {
  position: absolute;
  text-align: left;
  left: 20px;
  top: 20px;
  font-size: 11px;
  color: #ff9800;
}

.bullet {
  position: absolute;
  width: 20px;
  height: 20px;
}

.health {
  position: absolute;
  display: flex;
  flex-direction: row-reverse;
  right: 10px;
  top: 20px;
  font-size: 14px;
  color: #fd3e69;
}

.game-menu {
  position: fixed;
  background: rgba(0, 0, 0, 0.5);
  z-index: 9;
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.game-menu__start {
  border: 4px dashed #ff9800;
  font-size: 24px;
  background: none;
  font-family: "Press Start 2P", cursive;
  color: #ff9800;
  padding: 20px;
  margin-top: 20px;
  cursor: pointer;
}
.control-panel {
  opacity: 0;
}


@keyframes scrollBackground {
  from {
    background-position: 0 0;
  }
  to {
    background-position: 0 10000px;
  }
}
@keyframes spin {
  100% {
    transform: rotate(360deg);
  }
}

</style>

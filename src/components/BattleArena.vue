<template>
    <div class="arena">
    <div id="battle-log">Battlelog: {{ battleLog }}</div>
    <div class="player-containers">
        <div class="player-container">
            <div class="health">
                <div>{{ player.health }}/{{ player.maxHealth }}</div>
                <div class="health-bar">
                    <div class="health-bar-fill" :style="{ width: player.health + '%' }"> </div>
                </div>
            </div>
            <div>{{ player.name }}</div>

            <div class="character">
                <img src="../assets/walking.gif" alt="">
            </div>
            
            <div class="attacks">
                <button @click="playerAttack(player.attack1)">Kick</button>
                <button @click="playerAttack(player.attack2)">Punch</button>
                <button @click="healPlayer()">Heal</button>
            </div>
        </div>
        <div class="player-container">
            <div class="health">
                <div>{{ enemy.health }}/{{ enemy.maxHealth }}</div>
                <div class="health-bar">
                    <div class="health-bar-fill" :style="{ width: enemy.health + '%' }"> </div>
                </div>
            </div>
            {{ enemy.name }}

            <div class="enemy-character">
                <img src="../assets/bunnyjump.gif" alt="">
            </div>
        </div>
    </div>
    </div>

</template>

<script>
import { ref } from 'vue';

export default {
  data() {
    return {
      battleLog: '',
      player: {
        name: 'You',
        health: ref(100),
        maxHealth: ref(100),
        attack1: { name: 'Kick', damage: Math.floor(Math.random() * 20) + 20 },
        attack2: { name: 'Punch', damage: Math.floor(Math.random() * 45) + 0 },
      },
      enemy: {
        name: 'Easter Bunny Soldier',
        maxHealth: ref(100),
        health: ref(100),
        attack1: { name: 'Egg Throw', damage: Math.floor(Math.random() * 18) + 12 },
        attack2: { name: 'Chocolat Sword Slash', damage: Math.floor(Math.random() * 28) + 3 },
      },
      isPlayerTurn: true,
    };
  },

  methods: {
    playerAttack(attack) {
      if (this.isPlayerTurn) {
        this.attackEnemy(attack);
      }
    },

    attackEnemy(attack) {
      const damage = attack.damage;
      this.battleLog = `${this.player.name} used "${attack.name}" and dealt ${attack.damage} damage`;
      this.enemy.health -= damage;
      this.isPlayerTurn = false;
      if (this.enemy.health <= 0) {
        this.endGame('player');
      } else {
        setTimeout(() => {
          this.enemyAttack();
        }, 3000);
      }
    },

    enemyAttack() {
      this.battleLog = `${this.enemy.name} is thinking...`;
      setTimeout(() => {
        const attacks = [this.enemy.attack1, this.enemy.attack2];
        const attack = attacks[Math.floor(Math.random() * attacks.length)];
        const damage = attack.damage;
        this.player.health  -= damage;
        this.isPlayerTurn = true;
        if (this.player.health  <= 0) {
          this.endGame('enemy');
        } else {
          this.battleLog = `${this.enemy.name} used "${attack.name}" and dealt ${damage} damage. `;
        }
      }, 1000);
    },

    healPlayer() {
      if (this.isPlayerTurn) {
        const healingAmount = Math.floor(Math.random() * 20) + 10;
        this.player.health  += healingAmount;
        if (this.player.health > this.player.maxHealth ) {
          this.player.health  = this.player.maxHealth ;
        }
        this.battleLog = `You got healed for ${healingAmount}`;
        this.isPlayerTurn = false;
        setTimeout(() => {
          this.enemyAttack();
        }, 3000);
      }
    },

    endGame(winner) {
        setTimeout(() => {
            alert(winner + ' wins!');
        }, 500);
    },
  },
};
</script>
<style scoped>
.arena{ 
    position: fixed;
    top: 0px;
    left: 0px;
    width: 100vw;
    height: 100vh;
    background-color: rgba(0, 0, 0);
    color: white;
    z-index: 998;
    display: flex;
    flex-direction: column;
    text-align: center;
    justify-content: center;
}
.container{
    display: flex;
    justify-content: space-between;
}
.player-container{
    width: 50%;
    height: 100%;
    position: relative;
    border: none;
}
.player-containers{
    display: flex;
    height: 80vh;
}
#battle-log{ 
    border: none;
    width: 100vw;
    height: 50px;
    position: absolute;
    bottom: 0px;
    left: 0px;
    padding: 10px;
    text-align: left;
}
.attacks{
    width: 40%;
    position: absolute;
    left: 0px;
    bottom: 0px;
    display: grid;
    grid-template-columns: 1fr 1fr 1fr ;
    gap: 10px;
}
.attacks button {
    padding: 20px 10px;
    cursor: pointer;
}
.attacks button:nth-child(1){
    background: orange;
}
.attacks button:nth-child(2){
    background: red;
}
.attacks button:nth-child(3){
    background: green;
}
.health-bar {
  width: 200px;
  height: 20px;
  background-color: #333;
  margin-bottom: 10px;
  border: 2px solid white;
}
.health-bar-fill {
  height: 100%;
  background-color: red;
  transition: width 0.3s ease-in-out;
}
.health{
    width: 200px;
    margin: 10px;
}
.character{
    width: 300px;
    height: 300px;
    position: absolute;
    left: 27%;
    top: 30%;
}
.character img {
    width: 100%;
    height: 100%;
}
.enemy-character{
    width: 300px;
    height: 300px;
    position: absolute;
    left: 25%;
    top: 20%;
}
.character img {
    width: 100%;
    height: 100%;
}
</style>
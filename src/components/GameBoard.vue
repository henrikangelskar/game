<template>
    <div>
      <div class="health-bar">
        <div class="health-bar-fill" :style="{ width: currentHealth + '%' }"> </div>
      </div>
      <div class="health-text">Health: {{ currentHealth }} / {{ maxHealth }}</div>
      <div class="egg-text">Eggs: {{ eggCount }}/24</div>
      <div class="elapsed-time">Time: {{ elapsedTimeSeconds }}</div>
  
      <div
        v-for="(row, rowIndex) in map"
        :key="rowIndex"
        class="row"
      >
        <div
          v-for="(tile, tileIndex) in row"
          :key="tileIndex"
          class="tile"
          :class="{ active: playerPosition.row === rowIndex && playerPosition.tile === tileIndex }"
          :style="getTileStyle(rowIndex, tileIndex)"
        >
          <div class="egg" v-if="eggTiles[rowIndex][tileIndex]"></div>
          <div class="chocolate" v-if="chocolateTiles[rowIndex][tileIndex]"></div>

        </div>
    </div>

    <div class="arena" v-if="battleActive">
        <div>You encountered an enemy!</div>
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
                    <button @click="playerAttack(player.attack1)">Kick (Power)</button>
                    <button @click="playerAttack(player.attack2)">Punch (Reliable)</button>
                    <button @click="healPlayer()">Heal</button>
                </div>
            </div>
            <div class="player-container">
                <div class="health">
                    <div>{{ enemy.health }}/{{ enemy.maxHealth }}</div>
                    <div class="health-bar" :style="{ width: enemy.maxHealth + 'px' }">
                        <div class="health-bar-fill" :style="{ width: enemy.health + 'px' }"> </div>
                    </div>
                </div>
                {{ enemy.name }}

                <div class="enemy-character">
                    <img src="../assets/ogre.gif" alt="enemy">
                </div>
            </div>
        </div>
    </div>

    <div v-show="gameMessage" class="game-message-container">
        <div class="game-message">{{ gameMessage }}</div>
    </div>
    
    </div>
 </template>
  
<script>
import CharacterImage from '../assets/walking.gif';
import SpikeImage from '../assets/spikes.png'
import { ref } from 'vue';


export default {
    data() {
        return {
            map: Array(15).fill().map(() => Array(15).fill()),
            playerPosition: { row: 7, tile: 7 },
            maxHealth: 100,
            currentHealth: 100,
            eggTiles: Array(15).fill().map(() => Array(15).fill(false)),
            chocolateTiles: Array(15).fill().map(() => Array(15).fill(false)),
            spikeTiles: [
            { row: 3, tile: 4 },
            { row: 9, tile: 5 },
            { row: 12, tile: 10 },
            { row: 7, tile: 13 },
            { row: 8, tile: 1 },
            ],
            gameMessage: '',
            eggCount: 0,
            battleActive: false,
            battleLog: '',
            player: ref({
                name: 'You',
                health: ref(100),
                maxHealth: ref(100),
                attack1: ref({ name: 'Kick', damage: Math.floor(Math.random() * 10) + 10  }),
                attack2: ref({ name: 'Punch', damage: Math.floor(Math.random() * 30) + 4 }),
            }),
            isPlayerTurn: true,
            startTime: null, // set the initial start time to null
            elapsedTime: 0, // set the initial elapsed time to 0 seconds
            elapsedTimeSeconds: ref(0),
            tilesMoved: 0,

            enemyNames: [
                'Soldier',
                'Goblin',
                'Dragon',
                'Ogre',
                'Bandit',
                'Skeleton',
                'Zombie',
                'Giant Spider',
            ],
            attackNames: [
                'Sword Slash',
                'Sword Stab',
                'Knife Stab',
                'Bite',
                'Punch',
                'Kick',
                'Fireball',
                'Ice Shard',
                'Lightning Bolt',
                'Acid Spit',
                'Headbutt',
                'Tail Whip',
                'Poison Sting',
                'Bone Crush',
                'Shadow Strike'
            ],
            enemy: null
           
            
        };
    },
    mounted() {
        window.addEventListener('keydown', this.movePlayer);
        this.generateEggs();
        this.generateChocolateBars();
        this.startGame();
    },
    methods: {
        startGame() {
            this.startTime = Date.now();
            setInterval(() => {
                this.elapsedTimeSeconds = Math.floor((Date.now() - this.startTime) / 1000);
            }, 1000);
        },

        movePlayer(event) {
            if (this.battleActive) {
                return;
            }
            switch (event.keyCode) {
                case 37: // Left Arrow
                this.move(0, -1);
                break;
                case 38: // Up Arrow
                this.move(-1, 0);
                break;
                case 39: // Right Arrow
                this.move(0, 1);
                break;
                case 40: // Down Arrow
                this.move(1, 0);
                break;
                default:
                break;
            }
        },
        move(row, tile) {
            const newRow = this.playerPosition.row + row;
            const newTile = this.playerPosition.tile + tile;

            if (newRow >= 0 && newRow < 15 && newTile >= 0 && newTile < 15) {
                this.tilesMoved++;

                this.checkEggCount();
                this.playerPosition.row = newRow;
                this.playerPosition.tile = newTile;

                if (this.eggTiles[newRow][newTile]) {
                    this.eggTiles[newRow][newTile] = false;
                    this.eggCount += 1;
                }
                if (this.chocolateTiles[newRow][newTile]) {
                    this.chocolateTiles[newRow][newTile] = false;
                    if (this.currentHealth < 100) {
                        this.currentHealth += 10;
                    }
                }
                for (let i = 0; i < this.spikeTiles.length; i++) {
                    const spikeTile = this.spikeTiles[i];
                    if (spikeTile.row === newRow && spikeTile.tile === newTile) {
                        this.currentHealth -= 50;
                        break;
                    }
                }
            }
        },
        spawnEnemy() {
            const enemyName = this.randomEnemyName();
            const attack1 = this.randomAttackName();
            const attack2 = this.randomAttackName();

            if (this.eggCount === 5) {
                this.enemy = {
                    name: enemyName,
                    health: 50,
                    maxHealth:50,
                    attack1: { name: attack1, damage: 5 }, 
                    attack2: { name: attack2, damage: 5 }  
                };
            } else if (this.eggCount === 10) {
                this.enemy = {
                    name: enemyName,
                    health: 75,
                    maxHealth: 75,
                    attack1: { name: attack1, damage:  10 }, // 10 to 20
                    attack2: { name: attack2, damage: 10 }  // 10 to 20
                };
            } else if (this.eggCount === 15) {
                this.enemy = {
                    name: enemyName,
                    health: 100,
                    maxHealth: 100,
                    attack1: { name: attack1, damage: 15 }, // 15 to 30
                    attack2: { name: attack2, damage: 15 }  // 15 to 30
                };
            } else if (this.eggCount === 19) {
                this.enemy = {
                    name: enemyName,
                    health: 125,
                    maxHealth: 125,
                    attack1: { name: attack1, damage: 20 }, // 20 to 40
                    attack2: { name: attack2, damage: 20 }  // 20 to 40
                };
            }
        },
        randomEnemyName() {
            const randomIndex = Math.floor(Math.random() * this.enemyNames.length);
            return this.enemyNames[randomIndex];
        },
        randomAttackName() {
            const randomIndex = Math.floor(Math.random() * this.attackNames.length);
            return this.attackNames[randomIndex];
        },
        checkEggCount() {
            const endTime = Date.now();

            if (this.eggCount === 24) {
                alert(`Congratulations! You collected all the eggs in ${ this.elapsedTime = Math.round((endTime - this.startTime) / 1000)} seconds!`);
            } else if (this.eggCount === 5 || this.eggCount === 10 || this.eggCount === 15 || this.eggCount === 19) {
                
                this.gameMessage = "You encountered an enemy!";
                setTimeout(() => {
                    this.battleLog = ''
                    this.battleActive = true;
                    this.spawnEnemy();
                    this.gameMessage = false;
                }, 900);
                
                return;
            }
        },
        checkEnemyEncounter() {
        if (this.tilesMoved % 5 === 0) {
            this.battleActive = true;
            console.log("You have encountered an enemy!");
            this.spawnEnemy();
            return;
        }
        },
        getTileStyle(rowIndex, tileIndex) {
            if (this.playerPosition.row === rowIndex && this.playerPosition.tile === tileIndex) {
            return { backgroundImage: `url(${CharacterImage})` };
            }
            for (let i = 0; i < this.spikeTiles.length; i++) {
            const spikeTile = this.spikeTiles[i];
                if (spikeTile.row === rowIndex && spikeTile.tile === tileIndex) {
                return { backgroundImage: `url(${SpikeImage})` };
                }
            }
            return {};
        },
        generateEggs() {
            for (let i = 0; i < 20; i++) {
            let row, tile;
            do {
                row = Math.floor(Math.random() * 15);
                tile = Math.floor(Math.random() * 15);
            } while (this.eggTiles[row][tile] || (row === 0 && tile === 0));
            this.eggTiles[row][tile] = true;
            }
        },
        generateChocolateBars() {
            for (let i = 0; i < 5; i++) {
                let row, tile;
                do {
                row = Math.floor(Math.random() * 15);
                tile = Math.floor(Math.random() * 15);
                } while (this.eggTiles[row][tile] || this.chocolateTiles[row][tile] || (row === 0 && tile === 0));
                this.chocolateTiles[row][tile] = true;
            }
        },
        onBattleEnded() {
            this.battleActive = false;
        },
        playerAttack(attack) {
      if (this.isPlayerTurn) {
        this.attackEnemy(attack);
      }
    },

    attackEnemy(attack) {
        const damage = (attack.damage + Math.floor(Math.random() * 15) + 3);
        this.battleLog = `${this.player.name} used "${attack.name}" and dealt ${attack.damage} damage`;
        this.enemy.health -= damage;

        // Ensure enemy health doesn't go below 0
        if (this.enemy.health < 0) {
            this.enemy.health = 0;
        }

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
        const damage = attack.damage + Math.floor(Math.random() * 20) ;
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
        const healingAmount = Math.floor(Math.random() * 25) + 30;
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
        this.gameMessage = `${winner === 'player' ? 'You' : 'Enemy'} defeated ${winner === 'player' ? 'Enemy' : 'You'}`;

        if (winner !== 'player') {
            // Deduct 30 from player's maxHealth
            this.player.maxHealth -= 30;

            // Ensure player's maxHealth doesn't go below 0
            if (this.player.maxHealth < 0) {
                this.player.maxHealth = 0;
            }

            // Update player's current health to match the reduced maxHealth if needed
            if (this.player.health > this.player.maxHealth) {
                this.player.health = this.player.maxHealth;
            }
        }

        setTimeout(() => {
            this.eggCount ++;
            this.battleActive = false;
            this.resetGame();
        }, 3000);
    },
    resetGame() {
        this.gameMessage = '';
        this.player.health = this.player.maxHealth;
        this.enemy.health = this.enemy.maxHealth;
        this.isPlayerTurn = true;
    },

    },
};
</script>
  <style>
  .health-bar {
  width: 200px;
  height: 20px;
  background-color: #333;
  margin-bottom: 10px;
}

.health-bar-fill {
  height: 100%;
  background-color: red;
  transition: width 0.3s ease-in-out;
}
  .row {
    display: flex;
  }
  .tile {
    width: 50px;
    height: 50px;
    border: .5px solid rgb(38, 93, 29);
    background: rgb(59, 167, 59);
    background-repeat: no-repeat;
    background-size: contain;
  }
  .active {
    background-color: rgb(70, 194, 70);
  }
  .egg {
  background-image: url('../assets/egg.png');
  background-repeat: no-repeat;
  background-size: contain;
  width: 100%;
  height: 100%;
  z-index: 1;
}
.chocolate {
    background-image: url('../assets/candy.png');
  background-repeat: no-repeat;
  background-size: contain;
  width: 100%;
  height: 100%;
  z-index: 1;
}
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
    background: url("../assets/woods.png");
    object-fit: cover;
    background-size:     cover;                      /* <------ */
    background-repeat:   no-repeat;
    background-position: center center; 
}
.container{
    display: flex;
    justify-content: space-between;
}
.player-container{
    width: 50%;
    height: 100%;
    position: relative;
}
.player-containers{
    display: flex;
    height: 80vh;
}
#battle-log{ 
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
.game-message-container {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: rgba(0, 0, 0, 0.75);
    z-index: 999;
}

.game-message {
    font-size: 3rem;
    color: #ffffff;
    text-align: center;
    padding: 2rem;
    background-color: rgba(0, 0, 0, 0.5);
    border-radius: 1rem;
}
  </style>
  
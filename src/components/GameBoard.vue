<template>
    <div>
      <div class="about">
          <div class="player">
              <div class="health-bar">
                <div class="health-bar-fill" :style="{ width: currentHealth + '%' }"> </div>
              </div>
              <div class="health-text">Health: {{ currentHealth }} / {{ maxHealth }}</div>
              <div class="egg-text">Snickers bars: {{ eggCount }}/24</div>
              <div class="elapsed-time">Time: {{ elapsedTimeSeconds }}</div>
          </div>
          <div class="items">
            <div class="shield" >
                <img src="@/assets/shield.gif" alt="shield" v-if="player.hasShield">
            </div>
            <div class="sword" >
                <img src="@/assets/sword.gif" alt="sword" v-if="player.hasSword">
            </div>
          </div>
      </div>
  
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
        <div class="sword-item" v-if="swordTile && swordTile.row === rowIndex && swordTile.tile === tileIndex"></div>
        <div class="shield-item" v-if="shieldTile && shieldTile.row === rowIndex && shieldTile.tile === tileIndex"></div>      
        </div>
    </div>

    <div class="arena" v-if="battleActive">
        <div>You encountered an enemy!</div>
        <div v-if="enemy.name === 'ivor' || enemy.name === 'igor'">(BOSS)</div>
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

                <transition name="fade" v-on:after-enter="resetPlayerEffect">
                    <div class="character" :class="{ 'red-transition': damageEffect.player, 'green-transition': damageEffect.playerHeal }">
                        <img src="../assets/walking.gif" alt="">
                    </div>
                </transition>
                
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

                <div class="enemy-character" :class="{ 'red-transition': damageEffect.enemy }">
                    <transition name="fade" v-on:after-enter="resetEnemyEffect" class="my-transition">
                        <img v-if="enemy.name === 'soldier'" src="../assets/soldier.gif" alt="Soldier">
                    </transition>
                    <transition name="fade" v-on:after-enter="resetEnemyEffect"> 
                        <img v-if="enemy.name === 'ogre'" src="../assets/ogre.gif" alt="Ogre">
                    </transition>
                    <transition name="fade" v-on:after-enter="resetEnemyEffect"> 
                        <img v-if="enemy.name === 'bandit'" src="../assets/bandit.gif" alt="Bandit">
                    </transition>
                    <transition name="fade" v-on:after-enter="resetEnemyEffect"> 
                        <img v-if="enemy.name === 'skeleton'" src="../assets/skeleton.gif" alt="Skeleton">
                    </transition>
                    <transition name="fade" v-on:after-enter="resetEnemyEffect"> 
                        <img v-if="enemy.name === 'zombie'" src="../assets/zombie.gif" alt="Zombie">
                    </transition>
                    <transition name="fade" v-on:after-enter="resetEnemyEffect" class="my-transition"> 
                        <img v-if="enemy.name === 'igor'" src="../assets/igor.gif" alt="Igor">
                    </transition>
                    <transition name="fade" v-on:after-enter="resetEnemyEffect" class="my-transition"> 
                        <img v-if="enemy.name === 'ivor'" src="../assets/ivor.gif" alt="Ivor">
                    </transition>
                </div>
            </div>
        </div>
    </div>

    <div v-show="gameMessage" class="game-message-container">
        <div v-if="gameMessage">
            {{ gameMessage }}
            <button v-if="eggCount === 24 || dead" @click="resetGameData">Restart Game</button>
        </div>
    </div>
    
    <div style="width: 500px; padding: 10px;">
        Info: Goal is to collect all the snickers bars as fast as possible!
    </div>
    </div>
</template>
  
<script>
import CharacterImage from '../assets/walking.gif'
import SpikeImage from '../assets/spikes.png'
import TreeImage from '../assets/tree.png'
import { ref } from 'vue'


export default {
    data() {
        return {
            map: Array(15).fill().map(() => Array(15).fill()),
            playerPosition: { row: 7, tile: 7 },
            maxHealth: 100,
            treeTile: null,
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
            dead: false,
            swordTile: null,
            shieldTile: null,
            gameMessage: '',
            eggCount: 0,
            igorSpawned: false,
            ivorSpawned: false,
            battleActive: false,
            battleLog: '',
            player: ref({
                name: 'you',
                health: ref(100),
                maxHealth: ref(100),
                    attack1: ref({ name: 'Kick', damage: 0 }),
                    attack2: ref({ name: 'Punch', damage: 0 }),
                hasSword: false,
                hasShield: false,
            }),
            isPlayerTurn: true,
            startTime: null, // set the initial start time to null
            elapsedTime: 0, // set the initial elapsed time to 0 seconds
            elapsedTimeSeconds: ref(0),
            tilesMoved: 0,
            enemyImage: '',
            allowMovement: true,
            damageEffect: {
                player: false,
                enemy: false,
                playerHeal: false,
            },
            enemyNames: [
                'soldier',
                'ogre',
                'bandit',
                'skeleton',
                'zombie',
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
        this.generateSwordAndShield();
        this.generateTrees();
    },
    methods: {
        startGame() {
            this.startTime = Date.now();
            setInterval(() => { 
                this.elapsedTimeSeconds = Math.floor((Date.now() - this.startTime) / 1000);
            }, 1000);
        },

        movePlayer(event) {
            if (this.battleActive || !this.allowMovement) {
                return;
            }
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

            // check if new row and tile is within bounds of the map array
            if (newRow < 0 || newRow >= this.map.length || newTile < 0 || newTile >= this.map[newRow].length) {
                return;
            }

            if (this.map[newRow][newTile]) {
                return;
            }

            if (this.swordTile && this.swordTile.row === newRow && this.swordTile.tile === newTile) {
                this.player.hasSword = true;
                this.swordTile = null;
                this.gameMessage = "You encountered IGOR!";
                setTimeout(() => {
                    this.battleLog = '';
                    this.battleActive = true;
                    this.spawnIgor();
                    this.gameMessage = false;
                }, 1000);
            }

            if (this.shieldTile && this.shieldTile.row === newRow && this.shieldTile.tile === newTile) {
                this.player.hasShield = true;
                this.shieldTile = null;
                this.gameMessage = "You encountered IVOR!";
                setTimeout(() => {
                    this.battleLog = '';
                    this.battleActive = true;
                    this.spawnIvor();
                    this.gameMessage = false;
                }, 1000);
            }

            if (newRow >= 0 && newRow < 15 && newTile >= 0 && newTile < 15) {
                this.tilesMoved++;
                this.checkPlayerHealth();

                if (this.eggTiles[newRow][newTile]) {
                    this.eggTiles[newRow][newTile] = false;
                    this.eggCount += 1;
                    this.checkEggCount();
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

                if (!this.battleActive) {
                    this.playerPosition.row = newRow;
                    this.playerPosition.tile = newTile;
                }
            }
        },
        spawnEnemy() {
            if (this.igorSpawned) {
                return; // If Igor is already spawned, do nothing
            } else {
                const enemyName = this.randomEnemyName();
                const attack1 = this.randomAttackName();
                const attack2 = this.randomAttackName();

                if (this.eggCount === 5) {
                    this.enemy = {
                        name: enemyName,
                        health: 50,
                        maxHealth: 50,
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
                this.gameMessage = `Congratulations! You collected all the snickers in ${this.elapsedTime = Math.round((endTime - this.startTime) / 1000)} seconds!`;
                this.allowMovement = false;
            } else if (this.eggCount === 5 || this.eggCount === 10 || this.eggCount === 15 || this.eggCount === 19) {
                this.allowMovement = false;
                this.gameMessage = "You encountered an enemy!";
                setTimeout(() => {
                    this.battleLog = '';
                    this.battleActive = true;
                    this.spawnEnemy();
                    this.gameMessage = false;
                }, 900);
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
            if (this.map[rowIndex][tileIndex]) {
                return { backgroundImage: `url(${TreeImage})` };
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
        generateTrees() {
            for (let i = 0; i < 10; i++) {
                let row, tile;
                do {
                row = Math.floor(Math.random() * 15);
                tile = Math.floor(Math.random() * 15);
                } while (this.map[row][tile] || this.spikeTiles.find(spikeTile => spikeTile.row === row && spikeTile.tile === tile) || this.eggTiles[row][tile] || this.chocolateTiles[row][tile] || (row === 0 && tile === 0) || (this.playerPosition.row === row && this.playerPosition.tile === tile));
                this.map[row][tile] = true;
                this.treeTile = { row, tile };
            }
        },
        onBattleEnded() {
            this.battleActive = false;
            if (this.enemy.name === 'igor') {
                this.igorSpawned = false;
            }
            if (this.enemy.name === 'ivor') {
                this.ivorSpawned = false;
            }
        },
        playerAttack(attack) {
            if (this.isPlayerTurn) {
                this.attackEnemy(attack);
            }
        },
        spawnIgor() {
            this.enemy = {
                name: "igor",
                health: 150,
                maxHealth: 150,
                attack1: { name: "Igor's Smash", damage: 25 },
                attack2: { name: "Igor's Swipe", damage: 20 },
                image: "igor.gif",
            };
            this.igorSpawned = true;
        },
        spawnIvor() {
            this.enemy = {
                name: "ivor",
                health: 200,
                maxHealth: 200,
                attack1: { name: "Ivor's Shield Bash", damage: 15 },
                attack2: { name: "Ivor's Stab", damage: 12 },
                image: "ivor.gif",
            };
            this.ivorSpawned = true;
        },
        generateSwordAndShield() {
            let row, tile;

            // Generate sword position
            do {
                row = Math.floor(Math.random() * 15);
                tile = Math.floor(Math.random() * 15);
            } while (this.eggTiles[row][tile] || this.chocolateTiles[row][tile] || this.map[row][tile] || (row === 0 && tile === 0));
            this.swordTile = { row, tile };

            // Generate shield position
            do {
                row = Math.floor(Math.random() * 15);
                tile = Math.floor(Math.random() * 15);
            } while (this.eggTiles[row][tile] || this.chocolateTiles[row][tile] || this.map[row][tile] || (row === 0 && tile === 0) || (this.swordTile.row === row && this.swordTile.tile === tile));
            this.shieldTile = { row, tile };
        },
        attackEnemy(attack) {
            let damage;
            if (attack.name === 'Kick') {
                damage = Math.floor(Math.random() * 40);
            } else if (attack.name === 'Punch') {
                damage = Math.floor(Math.random() * 15) + 15;
            }

            if (this.player.hasSword) {
                damage = Math.floor(damage * 1.05); // Increase damage by 5% if the player has a sword
            }

            this.battleLog = `${this.player.name} used "${attack.name}" and dealt ${damage} damage`;
            this.enemy.health -= damage;

            this.damageEffect.enemy = true;
            setTimeout(() => {
                this.damageEffect.enemy = false;
            }, 500);

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
                this.damageEffect.player = true;
                setTimeout(() => { 
                    this.damageEffect.player = false;
                }, 1000);
                let attacks = [this.enemy.attack1, this.enemy.attack2];
                let attack = attacks[Math.floor(Math.random() * attacks.length)];
                let damage = attack.damage + Math.floor(Math.random() * 20) ;
                if (this.player.hasShield) {
                    damage = Math.floor(damage * 0.95); // Decrease damage by 5% if the player has a shield
                }
                this.player.health  -= damage;
                this.isPlayerTurn = true;
                if (this.player.health  <= 0) {
                    this.endGame('enemy');
                    this.player.health = 0;
                } else {
                    this.battleLog = `${this.enemy.name} used "${attack.name}" and dealt ${damage} damage. `;
                }
            }, 2000);
        },

        healPlayer() {
        if (this.isPlayerTurn) {
            const healingAmount = Math.floor(Math.random() * 80) + 20;
            this.damageEffect.playerHeal = true;
            setTimeout(() => {
                this.damageEffect.playerHeal = false;
            }, 1000);
            this.player.health  += healingAmount;
            if (this.player.health > this.player.maxHealth ) {
            this.player.health  = this.player.maxHealth ;
            }
            this.battleLog = `You got healed for ${healingAmount} health`;
            this.isPlayerTurn = false;
            setTimeout(() => {
            this.enemyAttack();
            }, 3000);
        }
        },

        endGame(winner) {
            this.gameMessage = `${winner === 'player' ? 'You' : this.enemy.name } defeated ${winner === 'player' ? this.enemy.name : 'You'}`;

            if (winner !== 'player') {
                // Deduct 30 from player's maxHealth
                this.currentHealth -= 30;

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
                this.eggCount++;
                this.battleActive = false;
                this.resetGame();
                this.allowMovement = true; // Re-enable player movement
            }, 3000);
        },
        resetGame() {
            this.gameMessage = '';
            this.player.health = this.player.maxHealth;
            this.enemy.health = this.enemy.maxHealth;
            this.isPlayerTurn = true;
        },
        resetGameData() {
            this.tilesMoved = 0;
            this.eggCount = 0;
            this.currentHealth = 100; // Set the player health to 100
            this.elapsedTimeSeconds = 0;
            this.elapsedTime = 0;
            this.startTime = Date.now();
            this.generateEggs();
            this.generateChocolateBars();
            this.allowMovement = true;
            this.gameMessage = '';
            this.resetEggs();
        },
        resetPlayerEffect() {
            this.damageEffect.player = false;
            this.damageEffect.playerHeal = false;
        },
        resetEnemyEffect() {
            this.damageEffect.enemy = false;
        },
        resetEggs() {
            for (let row = 0; row < this.eggTiles.length; row++) {
                for (let tile = 0; tile < this.eggTiles[row].length; tile++) {
                    if (this.eggTiles[row][tile]) {
                        this.eggTiles[row][tile] = false;
                    }
                }
            }
            this.generateEggs();
        },
        die() {
            this.allowMovement = false;
            this.gameMessage = "You died! Better luck next time.";
            this.battleActive = false;
            this.dead = true; // Set the dead variable to true when the player dies
        },
        checkPlayerHealth() {
            if (this.currentHealth <= 0) {
                this.currentHealth = 0;
                this.die();
            }
        },
    },
    computed: {
        enemyImagePath() {
            return this.getEnemyImage(this.enemy.type);
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
.enemy-character{
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
}
.enemy-character img{
    max-width: 250px;
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
.shield, .sword{
    width: 100px;
    height: 100px;
    border: 2px solid white;
}
.shield img, .sword img {
    width: 100%;
    height: 100%;
    object-fit: cover;
}
.egg {
  background-image: url('../assets/snickers.png');
  background-repeat: no-repeat;
  background-size: contain;
  width: 100%;
  height: 100%;
  z-index: 1;
}
.chocolate {
  background-image: url('../assets/coffee.png');
  background-repeat: no-repeat;
  background-size: contain;
  width: 100%;
  height: 100%;
  z-index: 1;
}
.shield-item {
  background-image: url('../assets/shield.gif');
  background-repeat: no-repeat;
  background-size: contain;
  width: 100%;
  height: 100%;
  z-index: 1;
}
.sword-item {
  background-image: url('../assets/sword.gif');
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
    background-size:     cover;             
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
.about{
    display: flex;
}
.player{
    width: 60%;
}
.items{
    width: 40%;
    display: flex;
    justify-content: center;
    align-items: center;
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
.character > * {
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
.fade-enter-active, .fade-leave-active {
  transition: opacity 1s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
.red-transition {
    background-color: rgba(255, 0, 0, 0.2);
    animation: redTransition 1s ease-out forwards;
    border-radius: 60%;
}

.green-transition {
  background-color: rgba(0, 255, 0, 0.3);
  border-radius: 50%;
}
</style>  
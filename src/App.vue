<template>
  <div id="main-app">
     <h1 class="text-center">Monster Slayer Game</h1>
     <div class="opponents">
        <div class="opponent">
            <h1 class="text-center">YOU</h1>
            <div class="healthbar">
                <div class="healthbar text-center"
                        style="background-color: green; margin: 0; color: white;"
                        :style="[{width: playerHealth + '%'}]">
                    {{ playerHealth }}
                </div>
            </div>
        </div>
        <div class="opponent">
            <h1 class="text-center">MONSTER</h1>
            <div class="healthbar">
                <div class="healthbar text-center"
                        style="background-color: green; margin: 0; color: white;"
                        :style="{width: monsterHealth + '%'}">
                    {{ monsterHealth }}
                </div>
            </div>
        </div>
    </div>
    <div class="controls" v-if="!gameIsRunning">
        <div>
            <button id="start-game" @click="startGame">START NEW GAME</button>
        </div>
    </div>
    <div class="controls" v-else>
        <div>
            <button id="attack" @click="attack">ATTACK</button>
            <button id="special-attack" @click="specialAttack">SPECIAL ATTACK</button>
            <button id="heal" @click="heal">HEAL</button>
            <button id="give-up" @click="giveUp">GIVE UP</button>
        </div>
    </div>
    <transition name="fade">
        <div class="log" v-if="turns.length > 0">
            <div>
                <ul>
                    <transition-group name="fade">
                        <li v-for="turn in turns"
                            :class="{'player-turn': turn.isPlayer, 'monster-turn': !turn.isPlayer}"
                            :key="turn.id">
                            {{ turn.text }}
                        </li>
                    </transition-group>
                </ul>
            </div>
        </div>
    </transition>
  </div>
</template>

<script>
export default {
  data() {
    return {
        playerHealth: 100,
        monsterHealth: 100,
        gameIsRunning: false,
        turns: [],
        currentTurn: 0
    }
  },
  methods: {
        startGame() {
            this.gameIsRunning = true;
            this.playerHealth = 100;
            this.monsterHealth = 100;
            this.turns = [];
        },
        attack() {
            var damage = this.calculateDamage(3, 10);
            this.monsterHealth -= damage;
            this.turns.unshift({
                isPlayer: true,
                text: 'Player hits Monster for ' + damage,
                id: this.currentTurn + 1
            });
            this.currentTurn++;
            if (this.checkWin()) {
                return;
            }

            this.monsterAttacks();
        },
        specialAttack() {
            var damage = this.calculateDamage(10, 20);
            this.monsterHealth -= damage;
            this.turns.unshift({
                isPlayer: true,
                text: 'Player hits Monster hard for ' + damage,
                id: this.currentTurn + 1
            });
            this.currentTurn++;
            if (this.checkWin()) {
                return;
            }
            this.monsterAttacks();
        },
        heal: function () {
            if (this.playerHealth <= 90) {
                this.playerHealth += 10;
            } else {
                this.playerHealth = 100;
            }
            this.turns.unshift({
                isPlayer: true,
                text: 'Player heals for 10',
                id: this.currentTurn + 1
            });
            this.currentTurn++;
            this.monsterAttacks();
        },
        giveUp: function () {
            this.gameIsRunning = false;
        },
        monsterAttacks() {
            var damage = this.calculateDamage(5, 12);
            this.playerHealth -= damage;
            this.checkWin();
            this.turns.unshift({
                isPlayer: false,
                text: 'Monster hits Player for ' + damage,
                id: this.currentTurn + 1
            });
            this.currentTurn++;
        },
        calculateDamage(min, max) {
            return Math.max(Math.floor(Math.random() * max) + 1, min);
        },
        checkWin() {
            if (this.monsterHealth <= 0) {
                this.monsterHealth = 0;
                this.MessageBox(true);
                return true;
            } else if (this.playerHealth <= 0) {
                this.playerHealth = 0;
                this.MessageBox(false);
                return true;
            }
            return false;
        },
        MessageBox(isWon) {
            var msg = '';
            var type = null;
            if (isWon) {
                msg = 'You won!';
                type = 'success';
            } else {
                msg = 'You lost!';
                type = 'error';
            }

            this.$swal({
                        title: msg,
                        text:'New Game?',
                        type: type,
                        confirmButtonColor: '#33cc33',
                        cancelButtonColor: '#ff0000',
                        confirmButtonText: 'Yes',
                        cancelButtonText: 'No',
                        showCancelButton: true })
                        .then((result) => {
                            if(result.value ) {
                                this.startGame();
                            } else {
                                this.gameIsRunning = false;
                            }                
                        }
            );
        }
    }
}
</script>

<style>
  .fade-enter-active {
      transition: opacity 1.5s;
  }

  .fade-enter {
       opacity: 0;
  }
</style>

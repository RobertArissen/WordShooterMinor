<template>
    <div v-show="gameStarted">
        <div class="p-4 bg-black shadow-inner rounded-b flex items-center px-2 justify-between">
            <question :words="words" :wordIndex="wordIndex" :correctLetters="correctLetters" :activeLetter="activeLetter"/>
            <words :words="words" :wordIndex="wordIndex" :playedOut="playedOut"/>
            <score :score="score"/>
        </div>
    </div>
</template>

<script>
    export default {
        props: ['selected'],

        data(){
            return {
                
                gameStarted: false,
                score: 0,
                playedOut: false,
                playerHealth: 3,

                words: [],
                wordIndex: 0,
                correctLetters: 0,
                activeLetter: 0,
            }
        },

        mounted(){
            this.words = window.words[this.selected].words

            this.gameStartedBus()

            this.hitLetter();

            EventBus.$on('restartGame', data => {    
                if(this.gameStarted){
                    this.restartGame()
                    console.log('3')
                }
            });
        },

        watch: { 
            selected: function(newVal, oldVal) {
                this.words = window.words[newVal].words
            }
        },


        methods: {
            gameStartedBus(){
                EventBus.$on('gameStarted', gameStarted => {
                    this.gameStarted = gameStarted

                    this.emitActiveWord()
                });
            },

            restartGame(){
                this.gameStarted = false
                this.score = 0
                this.playedOut = false
                this.playerHealth = 3

                this.words = window.words[this.selected].words
                this.wordIndex = 0
                this.correctLetters = 0
                this.activeLetter = 0
            },

            emitActiveWord(){
               EventBus.$emit('activeWord', this.words[this.wordIndex].word)
            },

            hitLetter(){
                EventBus.$on('hitLetter', letter => {
                    if(letter.toLowerCase() === this.words[this.wordIndex].word[this.activeLetter].toLowerCase()){
                        this.correctLetters++
                        this.activeLetter++
                        this.score += this.words[this.wordIndex].score/this.words[this.wordIndex].word.length
                        EventBus.$emit('score', this.score)

                        if(this.correctLetters === this.words[this.wordIndex].word.length){
                            if(this.words.length !== this.wordIndex+1){
                                this.wordIndex++
                                this.activeLetter = 0
                                this.correctLetters = 0

                                this.emitActiveWord()
                                EventBus.$emit('wordCorrect', [this.words[this.wordIndex].languageName, this.words[this.wordIndex].wordDutch])
                            }else{
                                this.playedOut = true
                                EventBus.$emit('playedOut', this.playedOut)
                            }
                        }
                    }else{
                        this.playerHealth--;
                        EventBus.$emit('playerHealth', this.playerHealth)
                    }
                });
            }
        }
    }
</script>

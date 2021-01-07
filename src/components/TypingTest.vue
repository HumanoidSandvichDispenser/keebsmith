<template>
    <div id="typing-test">
            <div id="word-list" ref="wordList">
            <Word v-for="(word, index) in wordList" :key="index" :word="word" :ref="setWordRef" :isCurrentWord="index == currentWord"/>
        </div>
        <div id="textarea" style="float: left;">
            <input id="input-box" ref="inputBox" placeholder="Type words here..." onpaste="return false" @keydown="keydown" @input="inputChanged($event.target, $event.target.value)">
        </div>
        <div id="btn-container">
            <div class="btn accent" @click="restart()">⏎ Restart</div>

            <ButtonGroup :radioOptions="{ '2 (Burst)': 2, '10': 10, '25': 25, '50': 50, '100': 100, '150': 150, '200': 200 }"
                default="25" ref="wordCountButton" selectedClass="selected" @changed="settingsChanged($event.target)"/>

            <ButtonGroup :radioOptions="{ 'Common Words': wordSets.commonWords, 'Difficult Words': wordSets.difficultWords }"
                default="Common Words" ref="wordSetButton" selectedClass="selected-alt" @changed="settingsChanged($event.target)"/>

            <ButtonGroup :checkboxOptions="{ 'Accept Errors': 'isAcceptingErrors' }"
                ref="flagsButton" selectedClass="selected-alt2" @changed="settingsChanged($event.target)"/>
        </div>
        <div id="speed-counter" v-if="isCompleted">{{ Math.round(cpm / 5) }} WPM ({{ cpm }} CPM)</div>
    </div>
</template>

<script>
import Word from "./Word"
import ButtonGroup from "./ButtonGroup"
// eslint-disable-next-line no-unused-vars
const words = require("../assets/words.json")

export default {
    data() {
        return {
            wordCountSettings: [ 2, 10, 25, 50, 100, 150 ],

            currentWord: 0,
            errorCount: 0,

            totalCharCount: 0,

            wordList: [],
            wordRefs: [],
            wordSets: words,

            isStarted: false,
            isCompleted: false,
            startTime: Number,
            endTime: Number,
            cpm: 0,

            settings: {
                wordCount: 25,
                wordSet: words.commonWords,
                flags: {
                    isAcceptingErrors: false,
                }
            },

        }
    },

    components: {
        Word,
        ButtonGroup
    },

    methods: {
        /**
         * Binds all Word.vue element refs to an array.
         */
        setWordRef: function(element) {
            if (element) {
                console.log("set word ref, length")
                this.wordRefs.push(element);
            }
        },

        /**
         * Triggers when user inputs to the textbox
         */
        inputChanged: function(inputBox, text) {
            if (!this.isStarted)
            {
                this.isStarted = true;
                this.startTime = new Date().valueOf() / 1000;
            }

            let currentWordText = this.wordRefs[this.currentWord].getWord();
            let errorIndex = this.findError(text, currentWordText);

            this.wordRefs[this.currentWord].charIndex = text.length;
            this.wordRefs[this.currentWord].inputText = text;

            if (text.slice(-1) == " " && (text.slice(0, -1) == currentWordText || this.settings.flags.isAcceptingErrors)) {
                if (this.settings.flags.isAcceptingErrors && text.slice(0, -1) != currentWordText) { // user made an error, but still accept
                    this.wordRefs[this.currentWord].errorIndex = errorIndex;
                    this.errorCount++;
                }

                this.currentWord++;

                if (this.currentWord >= this.wordList.length) {
                    this.restart(true);
                }

                this.wordRefs[this.currentWord].$el.scrollIntoView();

                inputBox.value = "";
            }
            else
            {
                this.wordRefs[this.currentWord].errorIndex = errorIndex;
            }

        },

        /**
         * Returns the index in the string where a typographical error has occurred
         */
        findError: function(text, expected) {
            let errorIndex = -1;

            text.split("").forEach((char, i) => {
                if (errorIndex == -1 && char != expected[i]) {
                    console.log(`Got ${char}, expected ${expected[i]} at ${i} (from the word ${expected})`);
                    errorIndex = i;
                }
            });

            return errorIndex;
        },

        /**
         * Restarts the typing test and sets all data to their initial values
         */
        restart: function(completed=false) {
            if (completed) {
                console.log("completed");
                this.isCompleted = true;
                this.endTime = new Date().valueOf() / 1000;
                this.cpm = this.calculateCPM();
                console.log("CPM: " + this.cpm);
            }

            this.$refs.inputBox.value = "";

            this.isStarted = false;
            this.startTime = 0;

            this.wordList = [];
            this.totalCharCount = 0;

            this.currentWord = 0;

            this.generateWords(this.settings.wordCount);

            this.$refs.wordList.scrollTop = 0;
        },

        /**
         * Generates words to be pushed to wordList
         */
        generateWords: function(count) {
            for (let rep = 0; rep < count; rep++) {
                let word = this.settings.wordSet.words[Math.floor(Math.random() * this.settings.wordSet.words.length)];
                this.totalCharCount += word.length + 1; // count spaces in the total char count
                this.wordList.push(word);
            }

            this.resetWordElements();
        },

        /**
         * Resets all Word.vue elements to their initial state
         */
        resetWordElements: function() {
            console.log(`resetting ${this.wordRefs.length} word elements`)
            this.wordRefs.forEach((wordRef) => wordRef.reset());
            this.wordRefs = [];
        },

        /**
         * Returns the user's typing speed in Chars Per Minute
         */
        calculateCPM: () => Math.round(this.totalCharCount * 60 / (this.endTime - this.startTime)),

        /**
         * Triggers when a user has selected a setting from the buttons
         */
        settingsChanged: function(button) {
            switch (button)
            {
                case this.$refs.wordCountButton:
                    this.settings.wordCount = button.radioValue;
                    break;
                case this.$refs.wordSetButton:
                    this.settings.wordSet = button.radioValue;
                    break;
                case this.$refs.flagsButton:
                    this.settings.flags = button.checkboxValues;
                    break;
            }

            this.restart();
        },
    },

    mounted() {
        this.restart();
    },

    beforeUpdate() {
        this.wordRefs = [];
    },
}
</script>
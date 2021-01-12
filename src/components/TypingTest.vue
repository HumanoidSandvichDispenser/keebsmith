<template>
    <div class="panel">
        <div id="word-list" ref="wordList">
            <div id="word-list-overflow">
                <Word v-for="(word, index) in wordList" :key="index" :word="word" :ref="setWordRef" :isCurrentWord="index == currentWord"
                    :isFavoriteWord="wordSets.favoriteWords.words.includes(word)" @click="toggleArrayElement(wordSets.favoriteWords.words, word)"/>
            </div>
        </div>
        <div id="textarea" style="float: left;">
            <input id="input-box" class="wide" ref="inputBox" placeholder="Type words here..." onpaste="return false" @keyup.enter="onEnter()" @input="inputChanged($event.target, $event.target.value)">
        </div>
        <div id="btn-container">
            <div tooltip="Restart Test" class="btn accent" @click="restart()">⏎ Restart</div>

            <ButtonGroup :radioOptions="{ '2 (Burst)': 2, '10': 10, '25': 25, '50': 50, '100': 100, '150': 150, '200': 200 }"
                default="25" ref="wordCountButton" selectedClass="selected" @changed="settingsChanged($event.target)"/>

            <ButtonGroup :radioOptions="{ 'Common Words': wordSets.commonWords, 'Difficult Words': wordSets.difficultWords, 'Favorite Words': wordSets.favoriteWords }"
                tooltipKey="metadata.description" default="Common Words" ref="wordSetButton" selectedClass="selected-alt" @changed="settingsChanged($event.target)"/>

            <ButtonGroup :checkboxOptions="{ 'Accept Errors': 'isAcceptingErrors' }"
                ref="flagsButton" selectedClass="selected-alt2" @changed="settingsChanged($event.target)"/>

            <input ref="regexFilterBox" list="regex-templates" placeholder="Regex filter...">

            <datalist id="regex-templates">
                <option value="^[asdfjkl]+$">QWERTY Home Row</option>
                <option value="^[aoeuhtns]+$">DVORAK 8-Key Home Row</option>
                <option value="^[aoeuidhtns]+$">DVORAK Full Home Row</option>
                <option value="^[eariotnslc]+$">10 Most Common Letters</option>
                <option value="^[^z]+$">No &quot;Z&quot;</option>
            </datalist>
        </div>
        <div id="speed-counter" :tooltip="cpm + ' CPM'" v-if="isCompleted">{{ Math.round(cpm / 5) }} WPM</div>
    </div>
</template>

<script>
import Word from "./Word";
import ButtonGroup from "./ButtonGroup";
// eslint-disable-next-line no-unused-vars
const words = require("../assets/words.json");

export default {
    name: "TypingTest",

    data() {
        return {
            wordCountSettings: [ 2, 10, 25, 50, 100, 150 ],

            currentWord: 0,
            errorCount: 0,

            totalCharCount: 0,

            wordList: [],
            wordRefs: [],
            wordSets: words,
            favoriteWords: [ "zoo" ],

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
         * Adds element to an array or removes it if the element exists in the array
         */
        toggleArrayElement(arr, word) {
            const index = arr.indexOf(word);

            if (index == -1) {
                arr.push(word);
            } else {
                arr.splice(index, 1);
            }
        },

        /**
         * Triggers when user inputs to the textbox
         */
        inputChanged: function(inputBox, text) {
            if (this.currentWord >= this.wordList.length)
                return;

            if (!this.isStarted) {
                this.isStarted = true;
                this.startTime = new Date().valueOf() / 1000;
            }

            let currentWordText = this.wordRefs[this.currentWord].getWord();
            let errorIndex = this.findError(text, currentWordText);

            this.wordRefs[this.currentWord].inputText = text;
            this.wordRefs[this.currentWord].charIndex = text.length;
            
            if (this.currentWord == this.wordList.length - 1 && text == currentWordText) { // last word and typed correctly
                this.currentWord++;
                this.complete();
            } else if (text.slice(-1) == " " && (text.slice(0, -1) == currentWordText || this.settings.flags.isAcceptingErrors)) {
                if (this.settings.flags.isAcceptingErrors && text.slice(0, -1) != currentWordText) { // user made an error, but still accept
                    this.wordRefs[this.currentWord].errorIndex = errorIndex; // display where the error occurred
                    this.errorCount++;
                }

                this.currentWord++;
                this.wordRefs[this.currentWord].$el.scrollIntoView();
                inputBox.value = "";
            }
            else {
                this.wordRefs[this.currentWord].errorIndex = errorIndex;
            }
        },

        /**
         * Triggers when the enter key is pressed
         */
        onEnter: function() {
            this.restart();
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

        complete: function() {
            this.isCompleted = true;
            this.endTime = new Date().valueOf() / 1000;
            this.cpm = this.calculateCPM();
            this.$refs.inputBox.value = "";
        },

        /**
         * Restarts the typing test and sets all data to their initial values
         */
        restart: function() {
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
            let words = this.settings.wordSet.words;
            let filter = new RegExp(this.$refs.regexFilterBox.value);

            if (filter != "") {
                words = words.filter((word) => word.match(filter))
            }

            for (let rep = 0; rep < count; rep++) {
                let word = words[Math.floor(Math.random() * words.length)];
                this.totalCharCount += word.length + (rep < count - 1 ? 1 : 0); // count spaces in the total char count if it's not the last char
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
        calculateCPM: function() {
            return Math.round(this.totalCharCount * 60 / (this.endTime - this.startTime));
        },

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
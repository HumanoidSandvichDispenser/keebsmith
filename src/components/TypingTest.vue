<template>
    <div id="typing-test">
        <div id="word-list" ref="wordList">
            <Word v-for="(word, index) in wordList" :key="index" :word="word" :ref="setWordRef" :isCurrentWord="index == currentWord"/>
        </div>
        <div id="textarea" style="float: left;">
            <input id="input-box" ref="inputBox" placeholder="Type words here..." onpaste="return false" @input="inputChanged($event.target, $event.target.value)">
        </div>
        <div id="btn-container">
            <div class="btn accent" @click="restart()">Restart</div>
            <div v-for="(count, index) in wordCountSettings" :key="index" class="btn" :ref="'word-count-' + count" @click="selectWordCount($event.target, count)"> {{ count }}</div>
            <div v-for="(wordSet, name) in wordSets" :key="name" class="btn" :ref="'word-set-' + name" @click="selectWordSet($event.target, name)"> {{ wordSet.metadata.name }}</div>
        </div>
        <span id="speed-counter" v-if="isCompleted">{{ Math.round(cpm / 5) }} WPM ({{ cpm }} CPM)</span>
    </div>
</template>

<script>
import Word from "./Word.vue"
// eslint-disable-next-line no-unused-vars
const words = require("../assets/words.json")

export default {
    data() {
        return {
            wordCountSettings: [ 2, 10, 25, 50, 100, 150 ],

            currentWord: 0,
            currentWordCountSetting: 25,
            currentWordCountSettingBtn: this.$refs["word-count-25"],

            totalCharCount: 0,

            currentSet: "commonWords",
            currentSetBtn: null,

            wordList: [],
            wordRefs: [],
            wordSets: words,

            isStarted: false,
            isCompleted: false,
            startTime: Number,
            endTime: Number,
            cpm: 0,
        }
    },

    components: {
        Word
    },

    methods: {
        setWordRef: function(element) {
            if (element) {
                console.log("set word ref, length")
                this.wordRefs.push(element);
            }
        },
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

            if (text.slice(-1) == " " && text.slice(0, -1) == currentWordText) {
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

            this.generateWords(this.currentWordCountSetting);

            this.$refs.wordList.scrollTop = 0;
        },
        generateWords: function(count) {
            for (let rep = 0; rep < count; rep++) {
                let set = words[this.currentSet];
                let word = set.words[Math.floor(Math.random() * set.words.length)];
                this.totalCharCount += word.length + 1; // count spaces in the total char count
                this.wordList.push(word);
            }

            this.resetWordElements();
        },
        resetWordElements: function() {
            console.log(`resetting ${this.wordRefs.length} word elements`)
            this.wordRefs.forEach((wordRef) => wordRef.reset());
            this.wordRefs = [];
        },
        calculateCPM: function() {
            return Math.round(this.totalCharCount * 60 / (this.endTime - this.startTime))
        },
        selectWordCount: function(btn, wordCountSetting) {
            if (this.currentWordCountSettingBtn) {
                this.currentWordCountSettingBtn.classList.remove("selected");
            }

            btn.classList.add("selected");
            this.currentWordCountSettingBtn = btn;

            this.currentWordCountSetting = wordCountSetting;
            this.restart();
        },
        selectWordSet: function(btn, setName) {
            if (this.currentSetBtn) {
                this.currentSetBtn.classList.remove("selected-alt");
            }

            btn.classList.add("selected-alt");
            this.currentSetBtn = btn;

            this.currentSet = setName;
            this.restart();
        }
    },

    mounted() {
        this.restart();

        this.$refs["word-count-25"].classList.add("selected");
        this.$refs["word-set-commonWords"].classList.add("selected-alt");

        this.currentWordCountSettingBtn = this.$refs["word-count-25"];
        this.currentSetBtn = this.$refs["word-set-commonWords"];
    },

    beforeUpdate() {
        this.wordRefs = [];
    },
}
</script>
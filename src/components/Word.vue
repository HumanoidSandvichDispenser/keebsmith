<template>
    <div class="word-container">
        <div class="cell-filler"></div>
        <div class="left" :style="[isCurrentWord ? 'text-decoration: underline;' : '']">
            {{ errorIndex == -1 ? word.slice(0, charIndex) : word.slice(0, errorIndex) }}
        </div>
        <div v-if="errorIndex != -1" class="error" :style="[isCurrentWord ? 'text-decoration: underline;' : '']">
            {{ word.slice(errorIndex, charIndex) + (charIndex > word.length  ? "●".repeat(charIndex - word.length) : "") }}
        </div>
        <div class="right" :style="[isCurrentWord ? 'text-decoration: underline;' : '']">
            {{ word.slice(charIndex) }}
        </div>
        <div v-show="isCurrentWord && errorIndex != -1" class="error-input"> {{ inputText.replaceAll(" ", "⎵") }} </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            charIndex: 0,
            errorIndex: -1,
            inputText: "",
        }
    },

    props:  {
        word: {
            type: String,
            default: "",
        },
        isCurrentWord: Boolean,
    },
    methods: {
        getWord: function() {
            return this.word;
        },
        reset: function() {
            this.charIndex = 0;
            this.errorIndex = -1;
            this.inputText = "";
        },
    }
    
}
</script>
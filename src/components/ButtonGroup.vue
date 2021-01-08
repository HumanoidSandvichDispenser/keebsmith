<template>
    <div>
        <PolyButton v-for="(radioOption, displayText) in radioOptions" :key="displayText" :selectedClass="selectedClass"
            :ref="'radioButton_' + displayText" type="radio" @toggle="radioButtonChanged($event.target, radioOption)">{{ displayText }}</PolyButton>

        <PolyButton v-for="(checkboxOption, displayText) in checkboxOptions" :key="displayText" :selectedClass="selectedClass"
            type="checkbox" @toggle="checkboxChanged($event.target, checkboxOption)">{{ displayText }}</PolyButton>
    </div>
</template>

<script>
import PolyButton from "./PolyButton"

export default {
    data() {
        return {
            currentRadioButton: null,            
            radioValue: null,
            checkboxValues: {},
        }
    },

    components: {
        PolyButton
    },

    props: {
        radioOptions: Object,
        checkboxOptions: Object,
        selectedClass: String,
        default: String,
        value: Object,
    },

    methods: {
        /**
         * Fired when user clicks a radio button
         */
        radioButtonChanged: function(radioButton, option) {
            if (this.currentRadioButton != radioButton) {
                if (this.currentRadioButton != null) {
                    console.log("toggle off previous radio button");
                    this.currentRadioButton.toggleOff();
                    this.currentRadioButton.isSelected = false;
                }

                this.currentRadioButton = radioButton;
                this.radioValue = option;
            }

            this.$emit("changed", { target: this });
        },

        /**
         * Fired when user clicks a checkbox
         */
        checkboxChanged: function(checkbox, option) {
            if (checkbox.isSelected) {
                this.checkboxValues[option] = true;
            } else if (this.checkboxValues[option] !== undefined) {
                this.checkboxValues[option] = false;
            }

            this.$emit("changed", { target: this });
        }
    },

    mounted() {
        let defaultButton = this.$refs["radioButton_" + this.default];
        if (defaultButton) {
            defaultButton.toggleOn();
            defaultButton.isSelected = true;
            this.currentRadioButton = defaultButton;
        }
    },
}
</script>
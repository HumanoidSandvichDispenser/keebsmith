<template>
    <div class="btn" ref="btn" @click="click()">
        <slot></slot>
    </div>
</template>

<script>
export default {
    name: "PolyButton",

    props: {
        type: String,
        selectedClass: String,
    },

    data() {
        return {
            isSelected: false,
        }
    },

    methods: {
        /**
         * Fired when the button is clicked
         */
        click: function() {
            if (this.type == "radio" && this.isSelected) // radio buttons can not untoggle
                return;

            this.isSelected = !this.isSelected;
            this.$emit("toggle", {
                target: this,
            });

            if (this.isSelected) {
                this.toggleOn();
            } else {
                this.toggleOff();
            }
        },

        /**
         * Applies the "selected" class to the button
         */
        toggleOn: function() {
            this.$refs.btn.classList.add(this.selectedClass);
        },

        /**
         * Removes the "selected" class to the button
         */
        toggleOff: function() {
            if (this.$refs.btn.classList.contains(this.selectedClass)) {
                this.$refs.btn.classList.remove(this.selectedClass);
            }
        }
    }
}
</script>
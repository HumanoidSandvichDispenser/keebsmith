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
        toggleOn: function() {
            this.$refs.btn.classList.add(this.selectedClass);
        },
        toggleOff: function() {
            if (this.$refs.btn.classList.contains(this.selectedClass)) {
                this.$refs.btn.classList.remove(this.selectedClass);
            }
        }
    }
}
</script>
<template>
    <li @click='onClick' tabindex="1" @focusout='onFocusOut' @focusin='onFocusIn' @mouseover='onMouseOver' @keydown='onKeyDown'
            :class="{ 'selected': menuState.selectedIndex == index }">
        <menu-item class=item :item='item' :menuState='menuState' />
        <sub-menu :keyDown='keyDown' v-if="show" @on-closing="onMenuClose" :items=subItems :menuState='menuState' ></sub-menu>
    </li>
</template>
    
<script>
import SubMenu from './SubMenu.vue'
import MenuItem from './MenuItem.vue'

export default {
    name: 'main-menu-item',
    components: {
        MenuItem,
        SubMenu
    },
    data() {
        return { 
            keyDown: null
        }
    },
    props: [ 
        'item', 
        'menuState',
        'index',
        'subItems'
    ],
    methods: {
        onMenuClose: function () {
            this.$emit('on-closing')
        },
        onKeyDown: function (evt) {
            switch (evt.which) {
                case 38: //  |^
                    this.keyDown = evt
                    break
                case 13: // Enter
                case 32: // Space
                case 40: //  |d
                    if (this.menuState.isKeyboardActivated)
                        this.menuState.isKeyboardActivated = false
                    else 
                        this.keyDown = evt
                    break;
                default:
                    if (this.menuState.accelerated)
                        this.keyDown = evt
            }
        },
        onClick: function () {
            if (this.menuState.isKeyboardActivated == true)
                this.menuState.isKeyboardActivated = false
            else {
                this.menuState.selectedIndex = 
                    this.menuState.selectedIndex != this.index
                    ? this.index
                    : -1
                if (this.menuState.selectedIndex == -1) 
                    this.close(this.menuState.lastActive)
            }
        },
        onFocusIn: function(evt) {
            if (!this.menuState.lastActive) 
                this.menuState.lastActive = evt.relatedTarget
        },
        onFocusOut: function(evt) {
            if (!this.menuState.menubar.contains(evt.relatedTarget)) 
                this.close(!evt.relatedTarget)
        },
        onMouseOver: function () {
            if (this.menuState.selectedIndex != this.index && this.menuState.selectedIndex != -1) {
                this.menuState.selectedIndex = this.index
                this.$el.focus()
            }
        },
        close: function (focus) {
            this.menuState.selectedIndex = -1
            if (focus && this.menuState.lastActive)
                this.menuState.lastActive.focus()
            this.menuState.lastActive = null
            this.$emit('keyboard-activated-stopped')
        }
    },
    computed: {
        show: function () {
            const result = this.menuState.selectedIndex == this.index
            if (result) {
                const recentIndex = this.menuState.selectedIndex
                this.$el.focus()
                const state = this.menuState
                state.selectedIndex = recentIndex
            }
            return !this.menuState.isKeyboardActivated && result
        }
    }
}
</script>

<style scoped>
    li {
        cursor: pointer;
        display: block;
        float: left;
        position: relative;
    }
    li:focus {
        outline: none;
    }

    li:hover {
        background-color: var(--vue-menu-hover-color);
    }

    li.selected {
        background-color: var(--vue-menu-selected-background-color);
        color: var(--vue-menu-selected-color);
    }
    .item {
        margin-left: 5px;
        margin-top: 2px;
        margin-right: 5px;
        margin-bottom: 2px;
    }
</style>

<template>
    <div class="autocomplete" @click="isOpen = true">
        <div class="typeahead-presentation" :class="{ open: isOpen }">
            <span class="typeahead-label">{{ getLabel() }}</span>
            <div class="input-arrow">
                <i class="fa fa-times close-icon" @click="resetValue" v-if="!isOpen && selected !== null"></i>

                <i class="fa fa-chevron-down" v-if="!isOpen"></i>
                <i class="fa fa-chevron-up" v-else></i>
            </div>
        </div>
        <div class="typeahead-list" v-if="isOpen">
            <input
                type="text"
                v-model="search"
                v-focus
                @input="onChange"
                @keydown.esc="isOpen = false"
                @keydown.down="onArrowDown"
                @keydown.up="onArrowUp"
                @keydown.enter.prevent="onEnter"
            />
            <ul class="results-list" v-if="results.length">
                <li
                    class="result-item"
                    v-if="results.length"
                    v-for="(item, i) in results"
                    :key="i"
                    :class="{ 'is-active': i === arrowCounter }"
                    @click="setResult(item)"
                >
                    {{ item[textColumn] }}
                </li>
            </ul>
            <ul class="results-list no-results" v-else>
                <li>No results found</li>
            </ul>
        </div>
    </div>
</template>

<script>

export default {
    props: {
        items: {
            type: Array,
            required: true,
        },
        valueColumn: {
            type: String,
            required: false,
            default: 'value',
        },
        textColumn: {
            type: String,
            required: false,
            default: 'text',
        },
    },

    directives: {
        focus: {
            inserted: function ( el ) {
                el.focus();
            }
        }
    },

    data() {
        return {
            search: '',
            isOpen: false,
            selected: null,
            arrowCounter: 0,
        };
    },

    computed: {
        results() {
            if (this.search !== '') {

                const valueColumn = this.valueColumn;
                const textColumn = this.textColumn;

                return this.items.filter(( item ) => {
                    if (item[valueColumn] !== null && item[textColumn]) {
                        return item[valueColumn].toString().toLowerCase().indexOf(this.search.toLowerCase()) > -1 ||
                            item[textColumn].toString().toLowerCase().indexOf(this.search.toLowerCase()) > -1;
                    }

                    return true;
                });
            }

            return this.items;
        }
    },

    watch: {
        isOpen(value){
            if(! value){
                this.search = '';
            }
        },

        selected(){
            if(this.selected !== null){
                this.$emit('input', this.selected[this.valueColumn]);
            } else {
                this.$emit('input', null);
            }
        },
    },

    methods: {
        getSelectedValue() {
            if(this.selected !== null){
                return this.selected[this.valueColumn];
            }

            return null;
        },

        getLabel() {
            if(this.selected !== null){
                return this.selected[this.textColumn];
            }

            return 'Select one option';
        },

        onChange() {
            this.isOpen = true;
        },

        setResult( result ) {
            this.search = '';
            this.selected = result;
            this.isOpen = false;
        },

        handleClickOutside( evt ) {
            if (!this.$el.contains(evt.target)) {
                this.isOpen = false;
                this.arrowCounter = -1;
            }
        },

        resetValue() {
            this.selected = null;
        },

        // Arrow support:

        onArrowDown() {
            if (this.arrowCounter < this.results.length - 1) {
                this.arrowCounter = this.arrowCounter + 1;
            } else {
                this.arrowCounter = 0;
            }
        },

        onArrowUp() {
            if (this.arrowCounter > 0) {
                this.arrowCounter = this.arrowCounter - 1;
            } else {
                this.arrowCounter = this.results.length - 1;
            }
        },

        onEnter() {
            this.selected = this.results[this.arrowCounter];
            this.search = '';
            this.isOpen = false;
        },
    },

    mounted() {
        document.addEventListener('click', this.handleClickOutside);
    },

    destroyed() {
        document.removeEventListener('click', this.handleClickOutside);
    }
};
</script>

<style scoped lang="scss">

.autocomplete {
    display: flex;
    flex-direction: column;
    position: relative;

    .typeahead-presentation {
        display: flex;
        flex-direction: row;
        align-items: center;
        justify-content: space-between;
        background-color: #FFFFFF;
        border: 1px solid #AAAAAA;
        border-radius: 4px;
        height: 28px;
        width: 100%;
        padding: 0 .33rem;

        .typeahead-label {
            font-size: .75rem;
        }

        .input-arrow {
            display: flex;
            flex-direction: row;
            align-items: center;

            i {
                color: #666666;

                &.close-icon {
                    color: #CCCCCC;
                    margin-right: .3rem;
                }
            }
        }

        &.open {
            border-bottom-left-radius: 0;
            border-bottom-right-radius: 0;
        }
    }

    .typeahead-list {
        position: absolute;
        background-color: #FFFFFF;
        right: 0;
        left: 0;
        top: 28px;
        border: 1px solid #AAAAAA;
        border-top: 0;
        padding: .3rem .2rem .2rem;
        border-bottom-left-radius: .3rem;
        border-bottom-right-radius: .3rem;
        z-index: 999999;

        input {
            margin-bottom: .3rem;
            height: 28px;
            border: 1px solid #aaa;
            border-radius: 0.2rem;
            width: 100%;
            padding: 0 .33rem;
        }

        .results-list {
            padding: .2rem;
            margin: 0;
            border: 1px solid #eeeeee;
            height: 120px;
            overflow: auto;

            &.no-results {
                height: auto;
            }

            .result-item {
                list-style: none;
                text-align: left;
                padding: 4px 2px;
                cursor: pointer;

                &.is-active {
                    background-color: lighten(#00A38D, 5);
                    color: white;
                }

                &:hover{
                    background-color: #00A38D;
                    color: white;
                }
            }
        }
    }
}
</style>

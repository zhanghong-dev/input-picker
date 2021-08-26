<style scoped>

    @import url('./iconfont.css');



    .portal-component__input-picker {
        width: 100%;
    }



    /**
     * inner
     */

    .portal-component__input-picker__inner {
        width: 100%;
        border-bottom: .1333333333333333vw solid #CCCCCC;
        background-color: #FFFFFF;
    }

    .portal-component__input-picker__inner input {
        display: block;
        width: 100%;
        padding: 2.666666666666667vw;
        border-width: 0;
        font-size: 4vw;
        line-height: 4vw;
        color: #1A1A1A;
        background-color: transparent;
    }

    .portal-component__input-picker__inner input:focus {
        outline: none;
    }

    .portal-component__input-picker__inner input::placeholder {
        color: #616161;
    }



    /**
     * drop-list
     */

    .portal-component__input-picker__drop-list {
        position: fixed;
        z-index: 1000;
        left: 0;
        top: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, .2);
    }

    .portal-component__input-picker__drop-list > div {
        position: absolute;
        left: 0;
        bottom: 0;
        width: 100%;
        padding: 2.666666666666667vw .5333333333333333vw 2.666666666666667vw 0;
        box-shadow: 0 0 1.333333333333333vw rgba(0, 0, 0, .3);
        background-color: #FFFFFF;
    }

    .portal-component__input-picker__drop-list .at__slot {
        overflow-x: hidden;
        overflow-y: auto;
        max-height: 74.66666666666667vw;
    }



    /**
     * filter
     */

    .portal-component__input-picker__drop-list .at__filter {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin: 0 5.333333333333333vw 2.666666666666667vw 5.333333333333333vw;
        border-bottom: .1333333333333333vw solid #CCCCCC;
    }

    .portal-component__input-picker__drop-list .at__filter:before {
        content: "\e6a6";
        font-family: "iconfont" !important;
        font-size: 5.066666666666667vw;
        line-height: 5.066666666666667vw;
        color: #616161;
    }

    .portal-component__input-picker__drop-list .at__filter input {
        width: 100%;
        padding: 2.666666666666667vw;
        border-width: 0;
        font-size: 4vw;
        line-height: 4vw;
        color: #1A1A1A;
        background-color: transparent;
    }

    .portal-component__input-picker__drop-list .at__filter input:focus {
        outline: none;
    }

    .portal-component__input-picker__drop-list .at__filter input::placeholder {
        color: #616161;
    }



    /**
     * deep
     */

    .portal-component__input-picker__drop-list /deep/ .portal-component__input-option {
        padding: 1.6vw 5.333333333333333vw;
        font-size: 4vw;
        line-height: 6.133333333333333vw;
        text-align: center;
        color: #606266;
    }

    .portal-component__input-picker__drop-list /deep/ .portal-component__input-option.__picked {
        font-weight: 800;
        color: #409EFF;
    }

    .portal-component__input-picker__drop-list /deep/ .portal-component__input-option.__hover {
        background-color: #F5F7FA;
    }

    .portal-component__input-picker__drop-list /deep/ .portal-component__input-option-group + .portal-component__input-option-group {
        position: relative;
        margin-top: 5.333333333333333vw;
    }

    .portal-component__input-picker__drop-list /deep/ .portal-component__input-option-group + .portal-component__input-option-group:before {
        content: "";
        position: absolute;
        left: 5.333333333333333vw;
        right: 5.333333333333333vw;
        top: -2.666666666666667vw;
        border-top: .1333333333333333vw solid #E4E7ED;
    }

    .portal-component__input-picker__drop-list /deep/ .portal-component__input-option-group > .at__header {
        padding: 2.666666666666667vw 5.333333333333333vw;
        font-style: italic;
        font-size: 3.2vw;
        line-height: 3.2vw;
        color: #909399;
    }

</style>

<template>
    <div class="portal-component__input-picker">



        <div class="portal-component__input-picker__inner">
            <input
                ref="input"
                autocomplete="off"
                readonly
                :value=" pickedOption && pickedOption.label "
                :placeholder="placeholder">
            </input>
        </div>



        <div v-show=" showDropList " ref="dropList" class="portal-component__input-picker__drop-list" :class="dropListClass">
            <div @click.stop>

                <div v-if=" filterable " class="at__filter">
                    <input ref="filterInput" :placeholder="filterPlaceholder"
                        @input="($event) => {
                            $emit('filter', $event.target.value);
                        }">
                    </input>
                </div>

                <div class="at__slot">
                    <slot></slot>
                </div>
            </div>
        </div>
    </div>
</template>

<script>

    export default {

        props: {

            value: {
                type: [ String, Number, Boolean ]
            },

            placeholder: {
                type: String
            },

            filterPlaceholder: {
                type: String
            },

            readonly: {
                type: Boolean
            },

            filterable: {
                type: Boolean
            },

            dropListClass: {
                type: String
            }

        },

        computed: {

            option() {

                let h = ($children) => {

                    let option = [];

                    for (let a = null, i = 0; a = $children[i]; i++) {

                        if ( a.isInputOptionComponent )
                            option.push(a);

                        else if ( a.$children.length )
                            option = option.concat( h(a.$children) );

                    }

                    return option;

                };

                return h(this.$children);

            }

        },

        methods: {



            /**
             * MVC 的 C 层方法
             */

            init() {

                for (let a = null, i = 0; a = this.option[i]; i++) {

                    if ( a.value === this.value )
                        this.pickedOption = a,
                        this.pickedOption.$el.classList.add('__picked'),
                        this.hoverOption = a,
                        this.hoverOption.$el.classList.add('__hover');

                }

            },

            handleEvent_showDropList($event) {

                if ( !this._isDestroyed && !this.readonly ) {

                    // 显示
                    if ( $event.target === this.$refs.input ) {

                        if ( this.pickedOption )
                            setTimeout(
                                () => {

                                    this.pickedOption.$el.scrollIntoView( { block: 'center' } );
                                    this.pickedOption.$el.focus();

                                }
                            );

                        this.$emit('filter', this.$refs.filterInput.value = '' );
                        this.showDropList = true;

                    }

                    // 隐藏
                    else
                        this.showDropList = false;

                }

            },

            handleEvent_picked($option) {

                if ( this.pickedOption )
                    this.pickedOption.$el.classList.remove('__picked');

                this.pickedOption = $option;
                this.pickedOption.$el.classList.add('__picked');
                this.$emit('input', $option.value);
                this.$emit('change', $option.value);

                document.dispatchEvent( new Event('click') );

            },

            handleEvent_hover($option) {

                if ( this.hoverOption )
                    this.hoverOption.$el.classList.remove('__hover');

                this.hoverOption = $option;
                this.hoverOption.$el.classList.add('__hover');

            }

        },

        mounted() {

            document.addEventListener('click', this.handleEvent_showDropList);
            this.$on('picked', this.handleEvent_picked);
            this.$on('hover', this.handleEvent_hover);
            this.$root.$el.appendChild( this.$refs.dropList );
            this.init();
            this.$emit('mounted');

        },

        beforeDestroy() {

            document.removeEventListener('click', this.handleEvent_showDropList);
            this.$root.$el.removeChild( this.$refs.dropList );

        },

        data() {

            return {
                isInputOptionParent: true,
                showDropList: false,
                pickedOption: null,
                hoverOption: null
            };

        }

    }

</script>

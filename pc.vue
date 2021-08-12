<style scoped>

    .portal-component__input-picker {
        position: relative;
        width: 100%;
    }



    /**
     * inner
     */

    .portal-component__input-picker__inner {
        width: 100%;
        border-bottom: 1px solid #CCCCCC;
        background-color: #FFFFFF;
    }

    .portal-component__input-picker__inner input {
        display: block;
        width: 100%;
        padding: 10px;
        border-width: 0;
        font-size: 16px;
        line-height: 16px;
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
        position: absolute;
        z-index: 1000;
        left: 0;
        top: 48px;
        min-width: 240px;
        padding: 10px 4px 10px 0;
        border: 1px solid #E4E7ED;
        box-shadow: 0 0 6px rgba(0, 0, 0, .1);
        background-color: #FFFFFF;
    }

    .portal-component__input-picker__drop-list:before {
        content: "";
        position: absolute;
        left: 12%;
        top: -12px;
        border-width: 6px;
        border-style: solid;
        border-color: transparent transparent #E4E7ED transparent;
    }

    .portal-component__input-picker__drop-list:after {
        content: "";
        position: absolute;
        left: calc(12% + 1px);
        top: -10px;
        border-width: 5px;
        border-style: solid;
        border-color: transparent transparent #FFFFFF transparent;
    }

    .portal-component__input-picker__drop-list > div {
        overflow-x: hidden;
        overflow-y: auto;
        max-height: 288px;
    }



    /**
     * deep
     */

    .portal-component__input-picker__drop-list /deep/ .portal-component__input-option {
        padding: 6px 20px;
        font-size: 16px;
        line-height: 24px;
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
        margin-top: 20px;
    }

    .portal-component__input-picker__drop-list /deep/ .portal-component__input-option-group + .portal-component__input-option-group:before {
        content: "";
        position: absolute;
        left: 20px;
        right: 20px;
        top: -10px;
        border-top: 1px solid #E4E7ED;
    }

    .portal-component__input-picker__drop-list /deep/ .portal-component__input-option-group > .at__header {
        padding: 10px 20px;
        font-style: italic;
        font-size: 12px;
        line-height: 12px;
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



        <div v-show=" showDropList " ref="dropList" class="portal-component__input-picker__drop-list" :class="dropListClass" @click.stop>
            <div>
                <slot></slot>
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

            readonly: {
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

            handleEvent_dropList($event) {

                if ( !this._isDestroyed && !this.readonly ) {

                    // 显示
                    if ( $event.target === this.$refs.input ) {

                        this.showDropList = true;

                        if ( this.pickedOption )
                            setTimeout(
                                () => {

                                    this.handleEvent_hover( this.pickedOption );
                                    this.pickedOption.$el.scrollIntoView( { block: 'center' } );
                                    this.pickedOption.$el.focus();

                                }
                            );

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
                this.handleEvent_hover($option);
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

            document.addEventListener('click', this.handleEvent_dropList);
            this.$on('picked', this.handleEvent_picked);
            this.$on('hover', this.handleEvent_hover);
            this.init();
            this.$emit('mounted');

        },

        beforeDestroy() {

            document.removeEventListener('click', this.handleEvent_dropList);

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

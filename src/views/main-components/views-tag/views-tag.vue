<style lang="less" scoped>
   .ivu-tag{
       .ivu-icon-ios-refresh-empty{
           color: #666!important;
           margin-left: 12px!important;
           font-size: 14px;
           opacity: .66;
           position: relative;
           top: 1px;
           transform: scale(1.42857143) rotate(0);
           cursor: pointer;
           display: inline-block;
           &:hover{
                opacity: 1;
            }
       }
       &:hover{
            border-color: #c5c5c5 !important;
        }
       &.view-tag-moved{
            border-style: dashed !important;
            border-color: #c5c5c5 !important;
            background: rgba(255, 255, 255, 0.5) !important;
            *{
                visibility: hidden !important;
            }
        }
   }
</style>
<template>
    <transition name="fade">
        <div :class="classes" @click.stop="check" :style="wraperStyles">
            <span :class="dotClasses" v-if="showDot" :style="bgColorStyle"></span>
            <span :class="textClasses" :style="textColorStyle"><slot></slot></span>
            <Icon v-if="color!=='default'" :class="iconClass" :color="lineColor" type="ios-refresh-empty" @click.native.stop="refresh"></Icon>
            <Icon v-if="closable" :class="iconClass" :color="lineColor" type="ios-close-empty" @click.native.stop="close"></Icon>
        </div>
    </transition>
</template>
<script>
    import util from '@/libs/util.js';
    const oneOf = util.oneOf;
    const prefixCls = 'ivu-tag';
    const initColorList = ['blue', 'green', 'red', 'yellow', 'default'];
    export default {
        name: 'Tag',
        props: {
            closable: {
                type: Boolean,
                default: false
            },
            checkable: {
                type: Boolean,
                default: false
            },
            checked: {
                type: Boolean,
                default: true
            },
            color: {
                type: String,
                default: 'default'
            },
            type: {
                validator (value) {
                    return oneOf(value, ['border', 'dot']);
                }
            },
            name: {
                type: [String, Number]
            },
            argu: {
                type: Object,
                default: {}
            },
            query: {
                type: Object,
                default: {}
            }
        },
        data () {
            return {
                isChecked: this.checked
            };
        },
        computed: {
            classes () {
                return [
                    `${prefixCls}`,
                    {
                        [`${prefixCls}-${this.color}`]: !!this.color && oneOf(this.color, initColorList),
                        [`${prefixCls}-${this.type}`]: !!this.type,
                        [`${prefixCls}-closable`]: this.closable,
                        [`${prefixCls}-checked`]: this.isChecked
                    }
                ];
            },
            wraperStyles () {
                return oneOf(this.color, initColorList) ? {} : {background: this.isChecked ? this.defaultTypeColor : 'transparent', borderWidth: '1px', borderStyle: 'solid', borderColor: ((this.type !== 'dot' && this.type !== 'border' && this.isChecked) ? this.borderColor : this.lineColor), color: this.lineColor};
            },
            textClasses () {
                return [
                    `${prefixCls}-text`,
                    this.type === 'border' ? (oneOf(this.color, initColorList) ? `${prefixCls}-color-${this.color}` : '') : '',
                    (this.type !== 'dot' && this.type !== 'border' && this.color !== 'default') ? (this.isChecked ? `${prefixCls}-color-white` : '') : ''
                ];
            },
            dotClasses () {
                return `${prefixCls}-dot-inner`;
            },
            iconClass () {
                if (this.type === 'dot') {
                    return '';
                } else if (this.type === 'border') {
                    return oneOf(this.color, initColorList) ? `${prefixCls}-color-${this.color}` : '';
                } else {
                    return this.color !== undefined ? (this.color === 'default' ? '' : 'rgb(255, 255, 255)') : '';
                }
            },
            showDot () {
                return !!this.type && this.type === 'dot';
            },
            lineColor () {
                if (this.type === 'dot') {
                    return '';
                } else if (this.type === 'border') {
                    return this.color !== undefined ? (oneOf(this.color, initColorList) ? '' : this.color) : '';
                } else {
                    return this.color !== undefined ? (this.color === 'default' ? '' : 'rgb(255, 255, 255)') : '';
                }
            },
            borderColor () {
                return this.color !== undefined ? (this.color === 'default' ? '' : this.color) : '';
            },
            dotColor () {
                return this.color !== undefined ? (oneOf(this.color, initColorList) ? '' : this.color) : '';
            },
            textColorStyle () {
                return oneOf(this.color, initColorList) ? {} : ((this.type !== 'dot' && this.type !== 'border') ? (this.isChecked ? {color: this.lineColor} : {}) : {color: this.lineColor});
            },
            bgColorStyle () {
                return oneOf(this.color, initColorList) ? {} : {background: this.dotColor};
            },
            defaultTypeColor () {
                return (this.type !== 'dot' && this.type !== 'border') ? (this.color !== undefined ? (oneOf(this.color, initColorList) ? '' : this.color) : '') : '';
            }
        },
        methods: {
            refresh () {
                this.$emit('refresh', this.name);
            },
            close (event) {
                if (this.name === undefined) {
                    this.$emit('on-close', event);
                } else {
                    this.$emit('on-close', event, {name: this.name, query: this.query});
                }
            },
            check () {
                if (!this.checkable) return;
                const checked = !this.isChecked;
                this.isChecked = checked;
                if (this.name === undefined) {
                    this.$emit('on-change', checked);
                } else {
                    this.$emit('on-change', checked, this.name);
                }
            }
        }
    };
</script>
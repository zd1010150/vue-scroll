<style scoped lang="sass">
    @import ../sass/bar

    .bar
        &--container
            position: relative
            height: 100%
            width: 100%
            overflow: hidden
            box-sizing: border-box

        &--wrapper
            position: absolute
            height: calc(100% + #{$pixel-proportion * 3})
            width: calc(100% + #{$pixel-proportion * 3})
            top: 0
            overflow-y: scroll
            overflow-x: scroll
            box-sizing: border-box

            > *
                position: relative
                padding-right: $pixel-proportion * 3 !important
                padding-bottom: $pixel-proportion * 3 !important


        &--vertical,
        &--horizontal
            position: absolute
            background-color: $black4
            z-index: 9
            border-radius: $pixel-proportion * 2
            overflow: hidden

            &-internal
                position: relative
                background-color: $black2
                opacity: 0.5
                border-radius: $pixel-proportion * 2
                transition-timing-function: ease-in-out
                transition-duration: .2s
                transition-property: opacity
                +cursor-pointer

                &:hover
                    opacity: 1

        &--vertical
            right: 0
            top: 0
            height: calc(100% - #{$pixel-proportion})
            // height: 100%
            width: $pixel-proportion
            margin-right: $pixel-proportion / 2
            margin-top: $pixel-proportion / 2
            margin-bottom: $pixel-proportion / 2

            &-internal
                height: 0px
                width: 100%

        &--horizontal
            left: 0
            bottom: 0
            width: calc(100% - #{$pixel-proportion})
            // width: 100%
            height: $pixel-proportion
            margin-right: $pixel-proportion / 2
            margin-left: $pixel-proportion / 2
            margin-bottom: $pixel-proportion / 2

            &-internal
                height: 100%
                width: 0px

</style>

<template lang="pug">
    #vbar(:class="propWrapperSize")
        .bar--container(ref="container",
            @wheel="scroll",
            @touchmove="scroll")

            .bar--vertical(ref="verticalBar", v-show="bars.vertical.size",
                :style="barSizeVertical",
                :class="propBarVertical",
                @touchstart="startDrag",
                @mousedown="startDrag",
                data-axis="Y",
                data-drag-source="bar")

                .bar--vertical-internal(ref="verticalInternalBar",
                    :style="barInternalVertical",
                    :class="propBarInternalVertical",
                    @touchstart="startDrag",
                    @mousedown="startDrag",
                    data-axis="Y",
                    data-drag-source="internal")

            .bar--horizontal(ref="horizontalBar", v-show="bars.horizontal.size",
                :style="barSizeHorizontal",
                :class="propBarHorizontal",
                @touchstart="startDrag",
                @mousedown="startDrag",
                data-axis="X",
                data-drag-source="bar")

                .bar--horizontal-internal(ref="horizontalInternalBar",
                    :style="barInternalHorizontal",
                    :class="propBarInternalHorizontal",
                    @touchstart="startDrag",
                    @mousedown="startDrag",
                    data-axis="X",
                    data-drag-source="internal")

            .bar--wrapper(ref="wrapperRef",
                :style="validationScrolls")
                slot
</template>

<script>
// **************************************************************************************** //
// V-BAR - LUIGUI DELYER | FRONT-END DEV @ WEBRADAR - NEW PRODUCTS, RESEARCH AND INNOVATION //
// GITHUB = https://github.com/luiguild/v-bar.git                                           //
// **************************************************************************************** //

import { addResizeListener, removeResizeListener } from 'detect-resize'

export default {
    data: () => ({
        dragging: {
            enable: false,
            axis: '',
            offset: ''
        },
        bars: {
            horizontal: {
                elm: '',
                parent: '',
                size: 0
            },
            vertical: {
                elm: '',
                parent: '',
                size: 0
            }
        },
        wrapperObj: {
            elm: '',
            scrollHeight: '',
            scrollWidth: '',
            scrollLeft: '',
            scrollTop: ''
        },
        container: {
            elm: '',
            scrollHeight: '',
            scrollWidth: ''
        },
        barInternalVertical: {
            top: 0
        },
        barInternalHorizontal: {
            left: 0
        },
        hiddenMaxStyle: {}
    }),
    mounted () {
        addResizeListener(this.$refs.container, this.resize)
        addResizeListener(this.$refs.wrapperRef.children[0], this.resize)

        document.addEventListener('mousemove', this.onDrag)
        document.addEventListener('touchmove', this.onDrag)
        document.addEventListener('mouseup', this.stopDrag)
        document.addEventListener('touchend', this.stopDrag)

        this.getEles()
        this.resize()
        // this.calculateHiddenMax()
    },
    beforeDestroy () {
        removeResizeListener(this.$refs.container, this.resize)
        removeResizeListener(this.$refs.wrapperRef.children[0], this.resize)

        document.removeEventListener('mousemove', this.onDrag)
        document.removeEventListener('touchmove', this.onDrag)
        document.removeEventListener('mouseup', this.stopDrag)
        document.removeEventListener('touchend', this.stopDrag)
    },
    computed: {
        propWrapperSize () {
            return this.wrapper ? this.wrapper : ''
        },
        propBarVertical () {
            return this.vBar ? this.vBar : ''
        },
        propBarInternalVertical () {
            return this.vBarInternal ? this.vBarInternal : ''
        },
        propBarHorizontal () {
            return this.hBar ? this.hBar : ''
        },
        propBarInternalHorizontal () {
            return this.hBarInternal ? this.hBarInternal : ''
        },
        barSizeVertical () {
            if (this.bars.horizontal.size && this.bars.vertical.size) {
                return {
                    height: 'calc(100% - 16px)'
                }
            }
        },
        barSizeHorizontal () {
            if (this.bars.horizontal.size && this.bars.vertical.size) {
                return {
                    width: 'calc(100% - 16px)'
                }
            }
        },
        validationScrolls () {
            if (!this.bars.horizontal.size) {
                return 'overflowX: hidden'
            }
            if (!this.bars.vertical.size) {
                return 'overflowY: hidden'
            }
        }
    },
    methods: {
        getEles () {
            this.$set(this.bars.horizontal, 'elm', this.$refs.horizontalInternalBar)
            this.$set(this.bars.horizontal, 'parent', this.$refs.horizontalBar)
            this.$set(this.bars.vertical, 'elm', this.$refs.verticalInternalBar)
            this.$set(this.bars.vertical, 'parent', this.$refs.verticalBar)
            this.$set(this.wrapperObj, 'elm', this.$refs.wrapperRef)
            this.$set(this.container, 'elm', this.$refs.container)
        },
        getSizes () {
            let wrapperObj = this.wrapperObj.elm,
                container = this.container.elm
            // this.bars.horizontal.size = wrapperObj.scrollWidth - container.scrollWidth > 24 &&
            //     wrapperObj.scrollWidth - container.scrollWidth !== 0
            //     ? (container.scrollWidth / wrapperObj.scrollWidth) * container.scrollWidth
            //     : 0
            // this.bars.vertical.size = wrapperObj.scrollHeight - container.scrollHeight > 24 &&
            //     wrapperObj.scrollHeight - container.scrollHeight !== 0
            //     ? (container.scrollHeight / wrapperObj.scrollHeight) * container.scrollHeight
            //     : 0
            this.bars.horizontal.size = wrapperObj.scrollWidth - container.scrollWidth !== 0
                ? (container.scrollWidth / wrapperObj.scrollWidth) * container.scrollWidth
                : 0
            this.bars.vertical.size = wrapperObj.scrollHeight - container.scrollHeight !== 0
                ? (container.scrollHeight / wrapperObj.scrollHeight) * container.scrollHeight
                : 0
        },
        scroll (e) {
            this.calculateInternalBar()
        },
        resize (e) {
            console.log('resize', e)
            this.getSizes()
            this.calculateInternalBar()
            // this.calculateHiddenMax()
        },
        calculateHiddenMax () {
            console.log('hiddenmax resize', this.wrapperObj.elm.scrollWidth)
            if (this.bars.horizontal.size > 0) {
                this.hiddenMaxStyle = {
                    width: (this.wrapperObj.elm.scrollWidth + 20) + 'px',
                    visibility: 'hidden',
                    height: '1px',
                    boxSizing: 'border-box'
                }
            }
        },
        getBarInternal (axis) {
            let internalSize,
                positionWrapper,
                sizeWrapper,
                sizeBar,
                sizeContainer,
                regulatorSize
            if (this.bars.horizontal.size && this.bars.vertical.size) {
                regulatorSize = 40
            } else {
                regulatorSize = 32
            }
            if (axis === 'X') {
                positionWrapper = this.wrapperObj.elm.scrollLeft
                sizeWrapper = this.wrapperObj.elm.scrollWidth
                sizeBar = this.bars.horizontal.size + regulatorSize
                sizeContainer = this.container.elm.scrollWidth
            } else if (axis === 'Y') {
                positionWrapper = this.wrapperObj.elm.scrollTop
                sizeWrapper = this.wrapperObj.elm.scrollHeight
                sizeBar = this.bars.vertical.size + regulatorSize
                sizeContainer = this.container.elm.scrollHeight
            }
            internalSize = (positionWrapper / (sizeWrapper - (sizeContainer))) * (sizeContainer - sizeBar)
            return internalSize
        },
        getCoordinates (e, axis) {
            console.log('getCoordinates trigger', new Date())
            let coordinate,
                sizeWrapper,
                sizeBar,
                sizeContainer

            if (axis === 'X') {
                sizeWrapper = this.wrapperObj.elm.scrollWidth
                sizeBar = this.bars.horizontal.size
                sizeContainer = this.container.elm.scrollWidth
                let percentage = (e.clientX - this.dragging.offset) / (this.bars.horizontal.parent.offsetWidth - sizeBar)
                coordinate = (percentage > 1 ? 1 : percentage) * (sizeWrapper - sizeContainer)
                console.log(sizeWrapper, sizeContainer, e.clientX, this.dragging.offset, percentage, 'x-dragging')
            } else if (axis === 'Y') {
                sizeWrapper = this.wrapperObj.elm.scrollHeight
                sizeBar = this.bars.vertical.size
                sizeContainer = this.container.elm.scrollHeight
                let percentage = (e.clientY - this.dragging.offset) / (this.bars.vertical.parent.offsetHeight - sizeBar)
                coordinate = (percentage > 1 ? 1 : percentage) * (sizeWrapper - sizeContainer)
                console.log(e.clientY, this.dragging.offset, this.bars.vertical.elm.offsetTop, percentage, this.bars.vertical.parent.offsetHeight, sizeBar, 'y-dragging')
            }
            return coordinate
        },
        startDrag (e) {
            e.preventDefault()
            e.stopPropagation()

            e = e.changedTouches ? e.changedTouches[0] : e
            let axis = e.target.getAttribute('data-axis'), offset
            if (axis === 'Y') {
                offset = e.clientY - this.bars.vertical.elm.offsetTop
            } else if (axis === 'X') {
                offset = e.clientX - this.bars.horizontal.elm.offsetLeft
            }
            this.dragging = {
                enable: true,
                axis: axis,
                offset
            }
        },
        onDrag (e) {
            if (this.dragging.enable) {
                e.preventDefault()
                e.stopPropagation()
                e = e.changedTouches ? e.changedTouches[0] : e

                const wrapper = this.$refs.wrapperRef

                if (this.dragging.axis === 'X') {
                    wrapper.scrollLeft = this.getCoordinates(e, 'X')
                } else if (this.dragging.axis === 'Y') {
                    wrapper.scrollTop = this.getCoordinates(e, 'Y')
                }

                this.calculateInternalBar()
            }
        },
        calculateInternalBar () {
            this.barInternalVertical = {
                height: this.bars.vertical.size + 'px',
                top: this.getBarInternal('Y') + 'px'
            }
            this.barInternalHorizontal = {
                width: this.bars.horizontal.size + 'px',
                left: this.getBarInternal('X') + 'px'
            }
            console.log('calculateInternalBar')
        },
        stopDrag (e) {
            if (this.dragging.enable) {
                this.dragging = {
                    enable: false,
                    axis: ''
                }
            }
        }
    },
    props: ['wrapper', 'vBar', 'vBarInternal', 'hBar', 'hBarInternal']
}
</script>

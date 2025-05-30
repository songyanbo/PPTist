<template>
  <div 
    class="base-element-shape"
    :style="{
      top: elementInfo.top + 'px',
      left: elementInfo.left + 'px',
      width: elementInfo.width + 'px',
      height: elementInfo.height + 'px',
    }"
  >
    <div
      class="rotate-wrapper"
      :style="{ transform: `rotate(${elementInfo.rotate}deg)` }"
    >
      <div 
        class="element-content"
        :style="{
          opacity: elementInfo.opacity,
          filter: shadowStyle ? `drop-shadow(${shadowStyle})` : '',
          transform: flipStyle,
          color: text.defaultColor,
          fontFamily: text.defaultFontName,
        }"
      >
        <svg 
          overflow="visible" 
          :width="elementInfo.width"
          :height="elementInfo.height"
        >
          <defs>
            <PatternDefs
              v-if="elementInfo.pattern"
              :id="`base-pattern-${elementInfo.id}`" 
              :src="elementInfo.pattern"
            />
            <GradientDefs
              v-else-if="elementInfo.gradient"
              :id="`base-gradient-${elementInfo.id}`" 
              :type="elementInfo.gradient.type"
              :colors="elementInfo.gradient.colors"
              :rotate="elementInfo.gradient.rotate"
            />
          </defs>
          <g 
            :transform="`scale(${elementInfo.width / elementInfo.viewBox[0]}, ${elementInfo.height / elementInfo.viewBox[1]}) translate(0,0) matrix(1,0,0,1,0,0)`"
          >
            <path 
              vector-effect="non-scaling-stroke" 
              stroke-linecap="butt" 
              stroke-miterlimit="8"
              :d="elementInfo.path" 
              :fill="fill"
              :stroke="outlineColor"
              :stroke-width="outlineWidth" 
              :stroke-dasharray="strokeDashArray" 
            ></path>
          </g>
        </svg>

        <div class="shape-text" :class="text.align">
          <div class="ProseMirror-static" v-html="text.content"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { computed } from 'vue'
import { storeToRefs } from 'pinia'
import type { PPTShapeElement, ShapeText } from '@/types/slides'
import { useSlidesStore } from '@/store'
import useElementOutline from '@/views/components/element/hooks/useElementOutline'
import useElementShadow from '@/views/components/element/hooks/useElementShadow'
import useElementFlip from '@/views/components/element/hooks/useElementFlip'
import useElementFill from '@/views/components/element/hooks/useElementFill'

import GradientDefs from './GradientDefs.vue'
import PatternDefs from './PatternDefs.vue'

const props = defineProps<{
  elementInfo: PPTShapeElement
}>()


const { theme } = storeToRefs(useSlidesStore())

const element = computed(() => props.elementInfo)
const { fill } = useElementFill(element, 'base')

const outline = computed(() => props.elementInfo.outline)
const { outlineWidth, outlineColor, strokeDashArray } = useElementOutline(outline)

const shadow = computed(() => props.elementInfo.shadow)
const { shadowStyle } = useElementShadow(shadow)

const flipH = computed(() => props.elementInfo.flipH)
const flipV = computed(() => props.elementInfo.flipV)
const { flipStyle } = useElementFlip(flipH, flipV)

const text = computed<ShapeText>(() => {
  const defaultText: ShapeText = {
    content: '',
    align: 'middle',
    defaultFontName: theme.value.fontName,
    defaultColor: theme.value.fontColor,
  }
  if (!props.elementInfo.text) return defaultText

  return props.elementInfo.text
})
</script>

<style lang="scss" scoped>
.base-element-shape {
  position: absolute;
}
.rotate-wrapper {
  width: 100%;
  height: 100%;
}
.element-content {
  width: 100%;
  height: 100%;
  position: relative;

  svg {
    transform-origin: 0 0;
    overflow: visible;
    display: block;
  }
}
.shape-text {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  display: flex;
  flex-direction: column;
  padding: 10px;
  line-height: 1.2;
  word-break: break-word;

  &.top {
    justify-content: flex-start;
  }
  &.middle {
    justify-content: center;
  }
  &.bottom {
    justify-content: flex-end;
  }
}
</style>

<template lang="pug">
.organism-user-name-input(:style="styles")
  .flex
    .focus-marker-base
      transition(name="focus-marker")
        .focus-marker(v-if="isFocused")
    .container
      input.input(
        ref="input"
        v-model="inputText"
        type="text"
        spellcheck="false"
        :placeholder="placeholder"
        :maxlength="maxlength"
        @mouseover="onHover"
        @mouseleave="onMouseLeave"
        @focus="onFocus"
        @blur="onBlur"
        @keydown.prevent.enter="onEnter"
        )
      .base-border
        .accent-border
  .button-container
    transition(name="button")
      .button(
        v-if="isFocused"
        @click="onEnter"
        )
        transition(name="fade")
          p.button-text(v-if="!isFetching") Fetch
          .stripe-loading(v-else)
</template>

<script lang="ts">
import { Component, Prop, Vue, Emit, Watch } from 'vue-property-decorator'

const MAX_WIDTH = '100%'
const MIN_WIDTH = '0'
const MAX_OPACITY = '0.7'
const MIN_OPACITY = '0'

@Component
export default class UserNameInput extends Vue {
  maxlength = 16

  placeholder = 'Username'

  opacity = MAX_OPACITY

  borderWidth = MIN_WIDTH

  inputText = ''

  isFocused = false

  isFetchDisabled = false

  @Prop({ default: '80px' })
  marginTop!: string

  @Prop({ default: '800px' })
  inputWidth!: string

  @Prop({ default: false })
  isFetching!: boolean

  focus() {
    const { input } = this.$refs

    if (input instanceof HTMLElement) {
      input.focus()
    }
  }

  get styles() {
    const { isFetchDisabled, opacity, borderWidth, marginTop, inputWidth } = this

    return {
      '--opacity': opacity,
      '--border-width': borderWidth,
      '--margin-top': marginTop,
      '--input-width': inputWidth,
      '--cursor': isFetchDisabled ? 'auto' : 'pointer',
      '--button-opacity': isFetchDisabled ? 0.3 : 1,
      '--button-pointer-events': isFetchDisabled ? 'none' : 'auto',
    }
  }

  onHover() {
    this.borderWidth = MAX_WIDTH
  }

  onMouseLeave() {
    const { isFocused } = this
    if (!isFocused) this.borderWidth = MIN_WIDTH
  }

  onFocus() {
    this.isFocused = true
    this.opacity = MIN_OPACITY
  }

  onBlur() {
    const { inputText } = this

    const matched = inputText.match(/^\S+/g)

    if (matched !== null && matched.length > 0) return

    this.isFocused = false
    this.opacity = MAX_OPACITY
    this.borderWidth = MIN_WIDTH
  }

  @Emit('input')
  emitInputText(text: string) {
    return text
  }

  onEnter() {
    const { inputText, isFetching, isFetchDisabled } = this

    if (isFetching || isFetchDisabled) return

    this.emitInputText(inputText)
  }

  @Watch('inputText', { immediate: true })
  watchInputText(newText: string) {
    const matched = newText.match(/^\w{3,16}$/g)

    this.isFetchDisabled = matched === null
  }
}
</script>

<style scoped lang="scss">
.organism-user-name-input {
  --border-width: initial;
  --opacity: initial;
  --margin-top: initial;
  --input-width: initial;
  --input-height: 50px;
  --cursor: initial;
  --button-opacity: initial;
  --button-pointer-events: initial;
  width: var(--input-width);
  margin: var(--margin-top) auto 0;
  overflow: hidden;
  transition: all 0.5s ease;
}

.flex {
  display: flex;
}

.focus-marker-base {
  margin: 0 20px 0 0;
  width: 10px;
  height: calc(var(--input-height) + 2px);
  background: #c4c4c4;
  overflow: hidden;
}

.focus-marker {
  width: 100%;
  height: 100%;
  background: repeating-linear-gradient(
    45deg,
    var(--teal),
    var(--teal) 5px,
    var(--white) 5px,
    var(--white) 8px
  );
}

.focus-marker-enter-active,
.focus-marker-leave-active {
  transition: all 0.3s cubic-bezier(0.6, 0.04, 0.98, 0.335);
}
.focus-marker-enter,
.focus-marker-leave-to {
  height: 0;
}

.container {
  width: calc(var(--input-width) - 30px);
}

.input {
  padding: 5px 10px;
  width: 100%;
  height: var(--input-height);
  line-height: var(--input-height);
  color: var(--black);
  font-family: 'Poppins', sans-serif;
  font-weight: bold;
  background: var(--white);
  border: none;
  font-size: 25px;
  outline: none;
}

.input::placeholder {
  opacity: var(--opacity);
  transition: all 0.35s ease;
}

.base-border {
  display: flex;
  justify-content: center;
  width: 100%;
  height: 2px;
  background: #c4c4c4;
}

.accent-border {
  width: var(--border-width);
  height: 2px;
  background: var(--teal);
  opacity: 1;
  transition: width 0.5s cubic-bezier(0.645, 0.045, 0.355, 1);
}

.button-container {
  display: flex;
  justify-content: flex-end;
  height: 50px;
  overflow: hidden;
}

.button {
  position: relative;
  margin: 15px 0 0;
  width: 150px;
  height: 35px;
  line-height: 35px;
  color: #fff;
  background: var(--deep-teal);
  text-align: center;
  font-size: 16px;
  border-radius: 2px;
  user-select: none;
  transition: all 0.3s ease;
  cursor: var(--cursor, pointer);
  opacity: var(--button-opacity);
  pointer-events: var(--button-pointer-events, auto);
}

.button:hover {
  background: var(--teal);
}

.button:active {
  background: var(--black);
}

.button-enter-active,
.button-leave-active {
  transition: all 0.8s ease;
}
.button-enter,
.button-leave-to {
  opacity: 0;
  margin-right: 50px;
}

@keyframes animate-stripes {
  0% {
    background-position: 0 0;
  }

  100% {
    background-position: 35px 0;
  }
}

.stripe-loading {
  position: absolute;
  top: 0;
  width: 150px;
  height: 35px;
  line-height: 35px;
  background-size: 35px;
  background-image: linear-gradient(
    45deg,
    var(--teal) 25%,
    #fff 25%,
    #fff 50%,
    var(--teal) 50%,
    var(--teal) 75%,
    #fff 75%,
    #fff
  );
  border-radius: 2px;
  user-select: none;
  animation: animate-stripes 0.7s linear infinite;
}

.fade-enter-active,
.fade-leave-active {
  transition: all 0.2s ease;
}

.fade-enter,
.fade-leave-to {
  opacity: 0;
}
</style>

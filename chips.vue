<template>
  <div class="chips-wrapper" @click.self="focusOnInput">
    <div class="chips-container">
      <div
        v-for="(chip, index) of chips"
        :key="chip.value || chip"
        :class="[
          'chip',
          {
            [chipClass]: !!chipClass,
            [invalidClass || 'invalid']: chip.invalid,
            disabled: disabled
          }
        ]"
        :tabindex="disabled || !removable ? undefined : 0"
        @keyup.delete="removeChip(index)"
      >
        <span class="chip-text">
          <slot :chip="chip" :index="index">{{ chip.value || chip }}</slot>
        </span>
        <transition name="fade">
          <button
            v-if="removable && !disabled"
            tabindex="-1"
            class="button-chip-remove"
            @click="removeChip(index)"
          >
            <slot name="removeIcon">
              <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 409.806 409.806">
                <path d="M228.929,205.01L404.596,29.343c6.78-6.548,6.968-17.352,
              0.42-24.132c-6.548-6.78-17.352-6.968-24.132-0.42c-0.142,0.137-0.282,0.277-0.42,
              0.42L204.796,
              180.878L29.129,5.21c-6.78-6.548-17.584-6.36-24.132,0.42
              c-6.388,6.614-6.388,17.099,0,23.713L180.664,205.01L4.997,
              380.677c-6.663,6.664-6.663,17.468,0,24.132
              c6.664,6.662,17.468,6.662,24.132,0l175.667-175.667l175.667,
              175.667c6.78,6.548,17.584,6.36,24.132-0.42
              c6.387-6.614,6.387-17.099,0-23.712L228.929,205.01z"
                />
              </svg>
            </slot>
          </button>
        </transition>
      </div>
      <input
        type="text"
        class="chip-input"
        ref="newChipInput"
        v-if="!disabled && ((!!maxChips && chips.length < maxChips) || !maxChips)"
        :value="newChip"
        :id="$attrs.id"
        :name="$attrs.name"
        :placeholder="$attrs.placeholder"
        :disabled="disabled"
        @input="updateNewChipModel($event)"
        @keyup.esc="resetNewChipModel"
        @keyup.enter="addNewChips($event)"
        @keydown.tab="addNewChipsOnBlur($event)"
        @keyup.delete="checkAndRemovePrevious"
      />
    </div>
  </div>
</template>

<script>
export default {
  inheritAttrs: false,

  props: {
    value: {
      type: Array,
      required: true,
    },
    disabled: Boolean,
    removable: {
      type: Boolean,
      default: true,
    },
    addOnBlur: Boolean,
    splitOn: {
      type: String,
      default: ',',
    },
    maxChipLength: Number,
    minChipLength: {
      type: Number,
      default: 0,
    },
    maxChips: Number,
    invalidClass: String,
    chipClass: String,
  },

  data() {
    return {
      chips: [],
      newChip: '',
      allowEmptyDelete: true,
      lowerChips: [],
    };
  },

  created() {
    this.setModel(this.value);
  },

  methods: {
    addNewChips() {
      if (!this.newChip || this.chips.length >= this.maxChips) return;

      const newChipsArray = this.newChip.trim().split(this.splitOn);
      const possibleAdditions = this.maxChips
        ? Math.min(this.maxChips - this.chips.length, newChipsArray.length)
        : newChipsArray.length;
      const addedChips = [];

      // eslint-disable-next-line no-plusplus
      for (let i = 0; i < possibleAdditions; i++) {
        const newChip = newChipsArray[i].trim();
        if (
          (this.minChipLength >= 0 && newChip.length < this.minChipLength)
          || (this.maxChipLength && newChip.length > this.maxChipLength)
        ) {
          // eslint-disable-next-line no-continue
          continue;
        }

        const newChipLower = newChip.toLowerCase();
        if (
          newChipLower.length
          && this.lowerChips.findIndex(
            (chip) => chip.value === newChipLower || chip === newChipLower,
          ) === -1
        ) {
          addedChips.push(newChip);
          this.chips.push(newChip);
          this.lowerChips.push(newChipLower);
        }
      }

      if (addedChips.length) {
        this.emitAdd(addedChips);
        this.emitModelChange();
      }

      this.resetNewChipModel();
    },

    addNewChipsOnBlur() {
      if (this.addOnBlur && this.newChip.length) {
        this.addNewChips();
        this.focusOnInput();
      }
    },

    checkAndRemovePrevious() {
      if (!this.removable) {
        return;
      }
      if (this.newChip.length === 0) {
        if (this.allowEmptyDelete) {
          this.removeChip();
        } else {
          this.allowEmptyDelete = true;
        }
      }
    },

    emitAdd(newChips) {
      this.$emit('add', newChips);
    },

    emitModelChange() {
      this.$emit('input', this.chips);
    },

    emitRemove(chip, index) {
      this.$emit('remove', chip, index);
    },

    focusOnInput() {
      if (this.$refs.newChipInput) {
        this.$refs.newChipInput.focus();
      }
    },

    removeChip(index = this.chips.length - 1) {
      if (!this.removable || this.disabled) {
        return;
      }
      if (this.chips.length) {
        const removed = this.chips.splice(index, 1);
        this.lowerChips.splice(index);
        this.emitModelChange();
        this.emitRemove(removed, index);
      }
    },

    resetNewChipModel() {
      this.newChip = '';
      this.allowEmptyDelete = true;
    },

    setModel(newValue) {
      this.chips = [...newValue];

      this.lowerChips = this.chips.map((chip) => (
        chip.value
          ? chip.value.toLowerCase()
          : chip.toLowerCase()
      ));
    },

    updateNewChipModel($event) {
      this.newChip = $event.target.value;

      if (this.newChip.length) {
        this.allowEmptyDelete = false;
      }
    },
  },

  watch: {
    value(newValue) {
      // Make sure new Array is actually different
      if (newValue && newValue !== this.chips) {
        this.setModel(newValue);
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.2s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}

.chips-wrapper {
  font-size: inherit;
  box-sizing: border-box;
  background-color: #f6f6f6;
  border: solid 1px #dfdfdf;
  border-radius: 3px;
  min-height: 36px;
  margin-bottom: 1rem
}

.chips-container {
  box-sizing: border-box;
  display: flex;
  flex-wrap: wrap;
}

.chip {
  box-sizing: border-box;
  display: flex;
  align-items: center;
  padding: 6px 8px;
  margin: 4px;
  font-size: 14px;
  line-height: 1;
  border-radius: 3px;
  background-color: #409fec;
  color: #fff;
  cursor: default;
  transition: background-color 0.3s;

  &:focus,
  &:hover {
    background-color: rgba(#409fec, 0.8);
  }

  &.disabled {
    background-color: #409fec;
    pointer-events: none;
  }

  &.invalid {
    background-color: #dd0036;

    &:focus,
    &:hover {
      background-color: #aa0036;
    }

    &.disabled {
      background-color: #770036;
    }
  }
}

.button-chip-remove {
  box-sizing: border-box;
  padding: 0;
  margin: 0 0 0 8px;
  background-color: transparent;
  border: none;
  line-height: 1;
  cursor: pointer;
  outline: none;

  svg {
    min-width: 9px;
    width: 9px;
    height: 9px;
    fill: #fff;
  }
}

.chip-input {
  box-sizing: border-box;
  background-color: #f6f6f6;
  border: 0;
  max-height: 36px;
  border-radius: 3px;
  font-family: inherit;
  font-size: 14px;
  line-height: 1;
  text-align: left;
  color: #0f232e;
  flex: 1;
  outline: none;
  padding: 10px;
  min-width: 100px;
}
</style>

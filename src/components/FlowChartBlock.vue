<template>
  <div
    :id="blockData.id"
    :class="`flowChartBlock--${blockData.type}`"
    class="flowChartBlock"
    @click="$emit('delete-block', blockData.id)"
  >
    <span v-if="!blockInputType">{{ blockData.value }}</span>

    <input
      v-else
      type="text"
      class="blockInput"
      :placeholder="placeholderText"
      :value="value"
      @input="$emit('input', $event.target.value)"
    />
  </div>
</template>

<script>
export default {
  name: 'FlowChartBlock',
  emits: ['delete-block'],
  props: {
    blockData: {
      type: Object,
      default: null,
    },
    blockInputType: {
      type: Boolean,
      default: false,
    },
    placeholderText: {
      type: String,
      default: 'Enter Text',
    },
    value: {
      type: String,
      default: '',
    },
  },
};
</script>

<style lang="scss" scoped>
.flowChartBlock {
  padding: 1rem;
  margin: 1.5rem;
  font-size: 1.6rem;
  text-align: initial;
  border-radius: 5px;

  &--action {
    background-color: #1abc9c;
    box-shadow: 0 0.5rem 1rem rgba(#1abc9c, 0.3);
    color: #fff;
    align-self: flex-end;

    .blockInput {
      color: #fff;
    }

    .blockInput::-webkit-input-placeholder {
      color: #fff;
    }
  }
  &--utterance {
    background-color: #3498db;
    box-shadow: 0 0.5rem 1rem rgba(#3498db, 0.3);
    color: #fff;
    align-self: flex-end;

    .blockInput {
      color: #fff;
    }

    .blockInput::-webkit-input-placeholder {
      color: #fff;
    }
  }

  &--usermessage {
    background-color: #ecf0f1;
    box-shadow: 0 0 1rem rgba(#ecf0f1, 0.3);
    color: #333;
    align-self: flex-start;

    .blockInput::-webkit-input-placeholder {
      color: #333;
    }
  }

  &:first-of-type {
    margin-top: 0;
  }

  &:last-of-type {
    margin-bottom: 0;
  }

  &:hover {
    cursor: pointer;
  }
}

.blockInput {
  height: 100%;
  width: 100%;
  background: transparent;
  border: none;
  outline: none;
}
</style>

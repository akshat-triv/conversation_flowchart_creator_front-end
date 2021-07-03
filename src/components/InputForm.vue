<template>
  <form
    autocomplete="off"
    action="#"
    class="inputForm"
    @submit.prevent="addNewBlock"
  >
    <select ref="inputType" name="inputType" id="inputType" required>
      <option value="UserMessage">UserMessage</option>
      <option value="Utterance">Utterance</option>
      <option value="Action">Action</option>
    </select>
    <input
      ref="inputValue"
      type="text"
      class="inputValue"
      id="inputValue"
      placeholder="Enter Text"
      required
    />
    <button class="submit">Add in flowchart</button>
  </form>
</template>

<script>
export default {
  name: 'InputForm',
  emits: ['new-block'],
  methods: {
    addNewBlock() {
      this.$emit('new-block', [
        this.$refs.inputType.value,
        this.$refs.inputValue.value,
      ]);
      this.$refs.inputValue.value = '';
    },
  },
};
</script>

<style lang="scss" scoped>
.inputForm {
  background-color: #2c3e50;
  margin: 4rem;
  min-height: 10rem;
  border-radius: 5px;
  box-shadow: 0 0.3rem 0.75rem rgba(0, 0, 0, 0.3);
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  justify-content: space-between;

  .submit {
    color: #fff;
    background: #34495e;
    border-radius: 3px;
    padding: 1rem;
    margin: 1rem;
    font-size: 1.2rem;
    outline: none;
    border: none;
    align-self: flex-end;
    box-shadow: 0 0.1rem 0.2rem rgba(#34495e, 0.1);
    transition: all 0.3s;

    &:hover {
      cursor: pointer;
      box-shadow: 0 0.1rem 0.2rem rgba(#34495e, 0.5);
    }
  }
}

#inputType {
  outline: none;
  border: none;
  background: #34495e;
  color: #fff;
  padding: 0.5rem;
  font-size: 1.2rem;
  margin: 1rem 1rem 1.5rem;
  border-radius: 3px;
  box-shadow: 0 0.1rem 0.2rem rgba(#34495e, 0.1);
  transition: all 0.3s;

  &:hover {
    cursor: pointer;
    box-shadow: 0 0.1rem 0.2rem rgba(#34495e, 0.3);
  }
}

#inputValue {
  align-self: stretch;
  padding: 0 1rem;
  font-size: 1.7rem;
  color: #fff;
  background: #2c3e50;
  outline: none;
  border: none;

  &::-webkit-input-placeholder {
    color: #fcfcfc;
  }
}
</style>

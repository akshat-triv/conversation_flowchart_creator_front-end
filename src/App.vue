<template>
  <div id="app">
    <div class="inputFormWrapper">
      <input-form @new-block="updateflowChartBlocks" />
      <button class="btn btnPrimary" @click="createJson">Create Json</button>
      <a href="/getJson">
        <button class="btn btnSecondary" @click="downloadJson">
          Download Json
        </button>
      </a>
      <span class="info">*{{ dataOutputted }}</span>
    </div>
    <div class="flowChartOutputWrapper">
      <div class="flowChartOutput">
        <flow-chart-block
          v-for="blockItem in flowChartBlocks"
          :key="`blockItem${blockItem.type}${blockItem.id}`"
          :block-data="blockItem"
          @delete-block="deleteBlock"
        />
      </div>
      <span class="info">*Click on the block to delete it</span>
    </div>
  </div>
</template>

<script>
import InputForm from '@/components/InputForm.vue';
import FlowChartBlock from '@/components/FlowChartBlock.vue';
import axios from 'axios';

export default {
  name: 'App',
  components: {
    InputForm,
    FlowChartBlock,
  },
  data() {
    return {
      flowChartBlocks: [
        {
          type: 'usermessage',
          id: 1,
          value: 'dummy text - user message block',
        },
        { type: 'action', id: 2, value: 'dummy text - action block' },
        { type: 'utterance', id: 3, value: 'dummy text - utterance block' },
      ],
      newId: 4,
      dataOutputted: "Data's been updated, create Json again.",
    };
  },
  methods: {
    updateflowChartBlocks(blockData) {
      const [type, value] = blockData;
      this.flowChartBlocks.push({
        type: type.toLowerCase(),
        id: this.newId,
        value,
      });
      this.newId++;
      this.dataOutputted = "Data's been updated, create Json again.";
    },
    deleteBlock(id) {
      this.flowChartBlocks = this.flowChartBlocks.filter(
        (curr) => curr.id != id
      );
      this.dataOutputted = "Data's been updated, create Json again.";
    },
    downloadJson() {
      this.dataOutputted = 'Downloading the JSON file';
    },
    createJson() {
      const flowChartBlocks = this.flowChartBlocks.map((curr) => {
        return { id: curr.id, type: curr.type, value: curr.value };
      });
      let newUserInputFlag = false,
        groupingInputOutput = { response: [] },
        arrayItemCounter = 1;

      const formattedJson = [];

      for (let block of flowChartBlocks) {
        arrayItemCounter++;
        if (block.type === 'usermessage') {
          if (!newUserInputFlag) {
            groupingInputOutput.query = block;
            newUserInputFlag = true;
          } else {
            formattedJson.push(groupingInputOutput);
            groupingInputOutput = { query: block, response: [] };
          }
        } else {
          groupingInputOutput.response.push(block);
        }
        if (arrayItemCounter == flowChartBlocks.length) {
          formattedJson.push(groupingInputOutput);
        }
      }

      axios
        .post('/saveData', formattedJson)
        .then(() => {
          this.dataOutputted = "Data's been saved in Json format.";
        })
        .catch((err) => {
          if (err) this.dataOutputted = 'Something went wrong.';
        });
    },
  },
};
</script>

<style lang="scss">
*,
*::after,
*::before {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html {
  font-size: 62.5%;
}

body {
  width: 100%;
  height: 100vh;
}

a:link,
a:visited,
a:active {
  text-decoration: none;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: flex-start;
  overflow: hidden;
}

.inputFormWrapper {
  width: 100%;
  height: 100%;
  background: #34495e;
  flex: 0 0 40%;
  position: relative;
}

.flowChartOutput {
  width: 100%;
  height: 100%;
  overflow-y: scroll;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  justify-content: flex-start;

  & > * {
    width: 50%;
  }

  &::-webkit-scrollbar {
    width: 14px;
  }

  &::-webkit-scrollbar-track {
    background: transparent;
  }

  &::-webkit-scrollbar-thumb {
    background: rgba(#aaa, 0.5);
    border-radius: 9999px;
    border: 5px solid transparent;
    background-clip: padding-box;
  }
}

.flowChartOutputWrapper {
  background-color: #2c3e50;
  height: 100%;
  overflow: hidden;
  flex: 1;
  padding: 4rem 0 10rem 2rem;
  position: relative;
}

.info {
  display: block;
  position: absolute;
  bottom: 4rem;
  left: 50%;
  transform: translateX(-50%);
  color: #95a5a6;
  font-size: 1.3rem;
}

.btn {
  outline: none;
  border: none;
  padding: 1rem;
  margin: 2rem auto;

  &:first-of-type {
    margin-right: 2rem;
  }

  &:hover {
    cursor: pointer;
  }
  &Primary {
    background: #ecf0f1;
    color: #333;
    box-shadow: 0 0 1rem rgba(#ecf0f1, 0.3);
  }
  &Secondary {
    color: #ecf0f1;
    background: #2c3e50;
    box-shadow: 0 0 1rem rgba(#2c3e50, 0.3);
  }
}
</style>

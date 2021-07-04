<template>
  <div id="app">
    <div class="inputFormWrapper">
      <input-form @new-block="addAtEnd" />
      <button class="btn btnPrimary" @click="createJson">Create Json</button>
      <a href="/getJson">
        <button class="btn btnSecondary" @click="downloadJson">
          Download Json
        </button>
      </a>
      <div class="blocksWrapper">
        <flow-chart-block
          v-model="userMessage"
          :blockInputType="true"
          :placeholderText="`Enter text for usermessage block.`"
          draggable="true"
          @dragstart.native="
            draggingStarted($event, {
              type: 'usermessage',
              value: userMessage,
            })
          "
          :block-data="{ type: 'usermessage', id: 999, value: 'Dummy Text' }"
        />
        <flow-chart-block
          v-model="utterance"
          :blockInputType="true"
          :placeholderText="`Enter text for utterance block.`"
          draggable="true"
          @dragstart.native="
            draggingStarted($event, {
              type: 'utterance',
              value: utterance,
            })
          "
          :block-data="{ type: 'utterance', id: 998, value: 'Hello' }"
        />
        <flow-chart-block
          v-model="action"
          :blockInputType="true"
          :placeholderText="`Enter text for action block.`"
          draggable="true"
          @dragstart.native="
            draggingStarted($event, { type: 'action', value: action })
          "
          :block-data="{ type: 'action', id: 997, value: 'Hello' }"
        />
      </div>
      <span class="info">*{{ dataOutputted }}</span>
    </div>
    <div class="flowChartOutputWrapper">
      <div
        class="flowChartOutput"
        ref="flowChartContainer"
        @drop="onDrop"
        @dragenter.prevent
        @dragover.prevent
      >
        <flow-chart-block
          v-for="blockItem in flowChartBlocks"
          :key="`blockItem${blockItem.type}${blockItem.id}`"
          :block-data="blockItem"
          class="draggable"
          @delete-block="deleteBlock"
          draggable="true"
          @dragstart.native="draggingStarted($event, blockItem.id)"
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
      newId: 3,
      dataOutputted: "Data's been updated, create Json again.",
      userMessage: '',
      utterance: '',
      action: '',
    };
  },
  methods: {
    draggingStarted(event, blockId) {
      if (blockId instanceof Object) {
        blockId = JSON.stringify(blockId);
        event.dataTransfer.setData('blockData', blockId);
        return;
      }
      event.dataTransfer.setData('blockId', blockId);
    },
    onDrop(event) {
      const blockId = event.dataTransfer.getData('blockId');
      const afterElement = this.getDragAfterElement(event.clientY);
      let blockIndex = NaN,
        blockData;

      if (!blockId) {
        blockData = JSON.parse(event.dataTransfer.getData('blockData'));
        blockData = this.createflowChartBlocks(blockData);
      } else {
        blockData = this.flowChartBlocks.find(
          (el) => el.id === parseInt(blockId)
        );
        blockIndex = this.flowChartBlocks.findIndex(
          (el) => el.id === parseInt(blockId)
        );
      }

      if (afterElement === undefined) {
        if (!isNaN(blockIndex)) {
          this.flowChartBlocks.splice(blockIndex, 1);
        }
        this.flowChartBlocks.push(blockData);
      } else {
        const afterElementId = this.flowChartBlocks.findIndex(
          (el) => el.id === afterElement.id
        );
        if (!isNaN(blockIndex)) {
          this.flowChartBlocks.splice(blockIndex, 1);
        }
        this.flowChartBlocks.splice(afterElementId, 0, blockData);
      }
    },
    getDragAfterElement(y) {
      const allDraggableNodes = Array.from(
        this.$refs.flowChartContainer.querySelectorAll('.draggable')
      );
      return allDraggableNodes.reduce(
        (nearest, curr) => {
          const box = curr.getBoundingClientRect();
          const offset = y - box.top - box.height / 2;

          if (offset < 0 && offset > nearest.offset) {
            return { offset, element: curr };
          } else {
            return nearest;
          }
        },
        {
          offset: Number.NEGATIVE_INFINITY,
        }
      ).element;
    },
    createflowChartBlocks(blockDataObject) {
      this.newId++;
      return Object.assign(blockDataObject, { id: this.newId });
    },
    addAtEnd(blockData) {
      const [type, value] = blockData;
      const blockDataObject = this.createflowChartBlocks({
        type: type.toLowerCase(),
        value,
      });
      this.flowChartBlocks.push(blockDataObject);
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

  .blocksWrapper {
    margin: 3rem 6rem;
    margin-bottom: 0;
    .flowChartBlock {
      &:not(:last-of-type) {
        margin-bottom: 3rem;
      }
    }
  }
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

<template>
  <v-container
    class="d-flex flex-column align-center"
    style="width: fit-content"
  >
    <v-container
      style="width: 250px; height: 300px"
      fluid
      class="vertical-progress-container"
    >
      <div class="d-flex flex-column">
        <div class="mb-12">
          <v-progress-linear
            v-model="value1"
            color="#410e73"
            height="80"
            class="progressBar"
          ></v-progress-linear>
        </div>
        <v-spacer class="mb-12"></v-spacer>
        <div>
          <v-progress-linear
            v-model="value2"
            color="#a8a873"
            height="80"
            class="progressBar"
          >
          </v-progress-linear>
        </div>
      </div>
    </v-container>
    <div style="width: 288px" class="d-flex justify-space-between mb-6 px-6">
      <div class="font-weight-regular w-25 my-n6">
        <v-text-field
          class="input1"
          v-model="option1"
          variant="plain"
          ref="input1"
          @click="selectAllText('input1')"
          @change="emitInput1"
        ></v-text-field>
      </div>
      <div class="font-weight-regular w-25 my-n6">
        <v-text-field
          class="input2"
          v-model="option2"
          variant="plain"
          ref="input2"
          @click="selectAllText('input2')"
          @change="emitInput2"
        ></v-text-field>
      </div>
    </div>
    <div class="d-flex justify-center" v-if="value1 !== 50">
      <v-chip variant="outlined" :color="value1 > 50 ? '#410e73' : '#a8a873'">
        <b>{{ value1 > 50 ? option1 : option2 }}&nbsp;</b>{{ result }}
      </v-chip>
    </div>
    <div class="d-flex justify-center hidden">
      <v-chip variant="outlined" :color="value1 > 50 ? '#410e73' : '#a8a873'">
        <b>{{ value1 > 50 ? option1 : option2 }}&nbsp;</b>{{ result }}
      </v-chip>
    </div>
  </v-container>
</template>

<script>
import { ref, watch, getCurrentInstance } from "vue";

export default {
  /**
   * Props:
   * - value1: The first input value (Type: Number, Required: true)
   * - value2: The second input value (Type: Number, Required: true)
   * - result: The result string (Type: String, Required: true)
   */
  props: {
    value1: {
      type: Number,
      required: true,
    },
    value2: {
      type: Number,
      required: true,
    },
    result: {
      type: String,
      required: true,
    },
  },
  setup(props) {
    const value1 = ref(props.value1);
    const value2 = ref(props.value2);
    const option1 = ref("Option 1");
    const option2 = ref("Option 2");
    // Watch the prop values for changes and update the refs accordingly
    watch(
      () => props.value1,
      (newValue) => {
        value1.value = newValue;
      }
    );
    watch(
      () => props.value2,
      (newValue) => {
        value2.value = newValue;
      }
    );
    // Get the current Vue component instance
    const instance = getCurrentInstance();
    // Emit input1 value to the parent component
    const emitInput1 = () => {
      instance.emit("input1", option1.value);
    };
    // Emit input2 value to the parent component
    const emitInput2 = () => {
      instance.emit("input2", option2.value);
    };
    return { value1, value2, option1, option2, emitInput1, emitInput2 };
  },
  methods: {
    /**
     * Method: selectAllText
     * Description: Selects all text in the referenced element.
     * @param {string} refName - The name of the ref referencing the element.
     */
    selectAllText(refName) {
      if (this.$refs[refName]) {
        this.$refs[refName].select();
      }
    },
  },
};
</script>

<style scoped lang="scss">
.hidden {
  visibility: hidden;
}
.vertical-progress-container {
  transform: rotate(-90deg);
}
.input1:deep(input) {
  color: #410e73;
}
.input2:deep(input) {
  color: #a8a873;
}
</style>


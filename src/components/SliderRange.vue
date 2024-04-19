<template>
  <v-container
    fluid
    :class="width > 426 ? 'mx-8 pt-16' : 'mt-n16'"
    style="width: fit-content"
    class="mx-auto"
  >
    <v-row no-gutters>
      <v-col order-sm="first">
        <div
          class="d-flex flex-column"
          :style="width < 500 ? 'width: max-content' : ''"
        >
          <div class="d-flex text-h5 font-weight-bold title">
            {{ title }}
          </div>
          <div
            v-if="width > 675"
            class="d-flex text-body text-justify font-weight-light"
          >
            {{ description1 }} {{ description2 }} <br />{{ description3 }}
          </div>
          <div
            v-if="width < 675"
            class="d-flex text-body text-justify font-weight-light"
          >
            <v-btn color="#00d1b2" class="white-text" @click="dialog = true">{{
              instruction[0]
            }}</v-btn>
            <v-dialog v-model="dialog" width="auto">
              <v-card>
                <v-card-text>
                  {{ description1 }} {{ description2 }} <br />{{ description3 }}
                </v-card-text>
                <v-card-actions>
                  <v-btn color="#00d1b2" block @click="dialog = false">{{
                    instruction[1]
                  }}</v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>
          </div>
          <div class="d-flex justify-space-between mb-n9">
            <div class="font-weight-regular mb-2 w-50">
              <v-textarea
                v-model="text1"
                variant="plain"
                rows="1"
                no-resize
                auto-grow
                ref="input1"
                @click="selectAllText('input1')"
                @change="updateText1"
              ></v-textarea>
            </div>
            <div class="font-weight-regular mb-2 w-50">
              <v-textarea
                v-model="text2"
                variant="plain"
                class="right-input"
                rows="1"
                no-resize
                auto-grow
                ref="input2"
                @click="selectAllText('input2')"
                @change="updateText2"
              ></v-textarea>
            </div>
          </div>
          <div class="colored-box1 mb-2" style="z-index: 10">
            <div class="d-flex flex-column pa-4">
              <v-slider
                v-model="slider1"
                thumb-label
                color="#410e73"
                thumb-color="white"
                thumb-size="40"
                track-size="30"
                track-color="white"
                rounded="xl"
                step="1"
                @end="updateData1()"
                @start="startValue1()"
                @click="addData1()"
              ></v-slider>
              <div class="font-weight-regular my-n6">
                <v-textarea
                  class="center-input"
                  v-model="text3"
                  variant="plain"
                  rows="1"
                  no-resize
                  auto-grow
                  ref="input3"
                  @click="selectAllText('input3')"
                  @change="updateText3"
                ></v-textarea>
              </div>
            </div>
          </div>
          <div class="colored-box2">
            <div class="d-flex flex-column pa-4">
              <v-slider
                v-model="slider2"
                thumb-label
                color="#a8a873"
                thumb-color="white"
                thumb-size="40"
                track-size="30"
                track-color="white"
                rounded="xl"
                step="1"
                @end="updateData2()"
                @start="startValue2()"
                style="z-index: 100"
                @click="addData2()"
              ></v-slider>
              <div class="font-weight-regular my-n6">
                <v-textarea
                  class="center-input"
                  v-model="text4"
                  variant="plain"
                  rows="1"
                  no-resize
                  auto-grow
                  ref="input4"
                  @click="selectAllText('input4')"
                  @change="updateText4"
                ></v-textarea>
              </div>
            </div>
          </div>
        </div>
      </v-col>
      <v-col order="first">
        <div>
          <vertical-progress-bar
            :value1="slider1"
            :value2="slider2"
            :result="result"
            @input1="handleInput1"
            @input2="handleInput2"
          ></vertical-progress-bar>
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
/**
 * This component represents a Vue setup for managing a vertical progress bar and related functionalities.
 * It includes reactive data, computed properties, and methods for handling user input, calculating durations,
 * updating data, and managing the lifecycle of the component.
 */
import VerticalProgressBar from "./VerticalProgressBar.vue";
import textEn from "../assets/json/textEn.json";
import textFr from "../assets/json/textFr.json";
import { ref, watch, onBeforeUnmount, computed, reactive } from "vue";
import { useWindowSize } from "@vueuse/core";
import { saveAs } from "file-saver";

export default {
  components: {
    VerticalProgressBar,
  },
  props: {
    /**
     * The language of the text data.
     */
    language: {
      type: String,
      required: true,
    },
    /**
     * The unique identifier of the user.
     */
    uid: {
      type: Number,
    },
    returnUrl: {
      type: URL,
    },
  },
  setup(props) {
    // Computed property to dynamically select text data based on the language
    const textData = computed(() => {
      return props.language === "en" ? textEn : textFr;
    });
    // Watcher to update text data when language changes
    watch(
      () => props.language,
      () => {
        title.value = textData.value.title;
        description1.value = textData.value.description1;
        description2.value = textData.value.description2;
        description3.value = textData.value.description3;
        leftScaleLabel.value = textData.value.leftScaleLabel;
        rightScaleLabel.value = textData.value.rightScaleLabel;
        topSliderLabel.value = textData.value.topSliderLabel;
        result.value = textData.value.result;
        instruction.value = textData.value.instruction;
        data.value.language = props.language;
      }
    );
    let activeSlider = null;
    const graph = ref({}); // Initialize the graph dictionary
    const title = ref(textData.value.title);
    const description1 = ref(textData.value.description1);
    const description2 = ref(textData.value.description2);
    const description3 = ref(textData.value.description3);
    const leftScaleLabel = ref(textData.value.leftScaleLabel);
    const rightScaleLabel = ref(textData.value.rightScaleLabel);
    const topSliderLabel = ref(textData.value.topSliderLabel);
    const bottomSliderLabel = ref(textData.value.topSliderLabel);
    const result = ref(textData.value.result);
    const instruction = ref(textData.value.instruction);
    const startTimeApp = ref(new Date());
    const startTime = ref(new Date());
    const sliders = ref([]);
    const slider1 = ref(50);
    const slider2 = ref(50);
    const text1 = ref(leftScaleLabel);
    const text2 = ref(rightScaleLabel);
    const text3 = ref(topSliderLabel);
    const text4 = ref(bottomSliderLabel);
    const choice1 = ref(slider1.value);
    const choice2 = ref(slider2.value);
    const { width, height } = useWindowSize();
    const dialog = ref(false);
    const text5 = ref("Option 1");
    const text6 = ref("Option 2");
    let startSlider1 = 50;
    let startSlider2 = 50;
    /**
     * Handles input for text input field 1.
     * If the input value is different from the current value, updates the value and pushes it to option1 array in data.
     * @param {string} value - The new input value.
     */
    const handleInput1 = (value) => {
      if (value !== text5.value) {
        text5.value = value;
        data.value.option1.push(text5.value);
      }
    };
    /**
     * Handles input for text input field 2.
     * If the input value is different from the current value, updates the value and pushes it to option2 array in data.
     * @param {string} value - The new input value.
     */
    const handleInput2 = (value) => {
      if (value !== text6.value) {
        text6.value = value;
        data.value.option2.push(text6.value);
      }
    };
    /**
     * Sets the activeSlider to Slider_1 and updates the startSlider1 and startTime values.
     * @param {number} value - The initial value of slider1.
     */
    const startValue1 = (value) => {
      activeSlider = "Slider_1";
      startSlider1 = slider1.value;
      startTime.value = new Date();
    };
    /**
     * Sets the activeSlider to Slider_2 and updates the startSlider2 and startTime values.
     * @param {number} value - The initial value of slider2.
     */
    const startValue2 = (value) => {
      activeSlider = "Slider_2";
      startSlider2 = slider2.value;
      startTime.value = new Date();
    };
    // Create dictionaries to store the arrays
    /**
     * Initializes the data object with the provided props and other default values.
     * @returns {object} The initialized data object.
     */
    const data = ref({
      userID: props.uid,
      returnUrl: props.returnUrl,
      duration_m_s: 0,
      EndTimeVCM: new Date(),
      sliders,
      leftScaleLabel: [],
      rightScaleLabel: [],
      topSliderLabel: [],
      bottomSliderLabel: [],
      option1: [],
      option2: [],
      language: props.language,
      graph:graph.value
    });
    /**
     * Generates an array of numbers within a specified range.
     * @param {number} start - The start of the range.
     * @param {number} stop - The end of the range.
     * @param {number} step - The step size between numbers in the range.
     * @returns {number[]} The array of numbers within the specified range.
     */
    function range(start, stop, step) {
      if (typeof stop == "undefined") {
        // one param defined
        stop = start;
        start = 0;
      }
      if (typeof step == "undefined") {
        step = 1;
      }
      if ((step > 0 && start >= stop) || (step < 0 && start <= stop)) {
        return [];
      }
      var result = [];
      for (var i = start; step > 0 ? i < stop : i > stop; i += step) {
        result.push(i);
      }
      return result;
    }
    watch(slider1, (newValue) => {
      slider2.value = 100 - newValue;
    });
    watch(slider2, (newValue) => {
      slider1.value = 100 - newValue;
    });
    /**
     * Calculates the duration between two timestamps.
     * @param {Date} startTime - The start timestamp.
     * @param {Date} endTime - The end timestamp.
     * @returns {string} The duration formatted as "Xm Ys" or "Z ms".
     */
    const calculateDuration = (startTime, endTime) => {
      const durationInMillis = endTime - startTime;
      const minutes = Math.floor(durationInMillis / (1000 * 60));
      const seconds = Math.floor((durationInMillis / 1000) % 60);
      if (durationInMillis >= 1000) {
        return `${minutes}m ${seconds}s`;
      } else {
        return `${durationInMillis} ms`;
      }
    };
    // Methods to update data and text labels
    const updateData1 = () => {
      activeSlider = null;
      if (startSlider1 < slider1.value) {
        const values = range(startSlider1, slider1.value + 1);
        const endTime = new Date();
        const duration = calculateDuration(startTime.value, endTime);
        sliders.value.push({
          name: "Slider_1",
          values,
          startTime,
          endTime: endTime,
          duration: duration,
        });
      }
      if (startSlider1 > slider1.value) {
        const values = range(slider1.value, startSlider1 + 1).reverse();
        const endTime = new Date();
        const duration = calculateDuration(startTime.value, endTime);
        sliders.value.push({
          name: "Slider_1",
          values,
          startTime,
          endTime: endTime,
          duration: duration,
        });
      }
      // data.value.topSlider.push(slider1.value);
      // data.value.bottomSlider.push(slider2.value);
    };
    const updateData2 = () => {
      activeSlider = null;
      if (startSlider2 < slider2.value) {
        const values = range(startSlider2, slider2.value + 1);
        const endTime = new Date();
        const duration = calculateDuration(startTime.value, endTime);
        sliders.value.push({
          name: "Slider_2",
          values,
          startTime,
          endTime: endTime,
          duration: duration,
        });
      }
      if (startSlider2 > slider2.value) {
        const values = range(slider2.value, startSlider2 + 1).reverse();
        const endTime = new Date();
        const duration = calculateDuration(startTime.value, endTime);
        sliders.value.push({
          name: "Slider_2",
          values,
          startTime,
          endTime: endTime,
          duration: duration,
        });
      }
    };
    /**
     * Updates the left scale label data with the current value of text1.
     */
    const updateText1 = () => {
      data.value.leftScaleLabel.push(text1.value);
    };
    /**
     * Updates the right scale label data with the current value of text2.
     */
    const updateText2 = () => {
      data.value.rightScaleLabel.push(text2.value);
    };
    /**
     * Updates the top slider label data with the current value of text3.
     */
    const updateText3 = () => {
      data.value.topSliderLabel.push(text3.value);
    };
    /**
     * Updates the bottom slider label data with the current value of text4.
     */
    const updateText4 = () => {
      data.value.bottomSliderLabel.push(text4.value);
    };
    // Save the data to a file when the page is unloaded (refreshed or closed)
    /**
     * Saves the data to a file.
     * Updates the end time and duration of the VCM session in the data object.
     * Converts the data object to JSON format and saves it as a text file.
     */
    const saveDataToFile = () => {
      const endTimeVCM = new Date();
      data.value.EndTimeVCM = endTimeVCM;
      const duration = calculateDuration(startTimeApp.value, endTimeVCM);
      data.value.duration_m_s = duration;
      const jsonData = JSON.stringify(data.value, null, 2);
      const blob = new Blob([jsonData], { type: "text/plain;charset=utf-8" });
      saveAs(blob, "data.txt");
    };
    /**
     * Lifecycle hook executed before the component is destroyed.
     */
    onBeforeUnmount(() => {
      saveDataToFile();
      clearInterval(timerInterval);
    });
    // Register the window.onbeforeunload event to save the data when the page is refreshed or closed
    window.onbeforeunload = () => {
      saveDataToFile();
    };
    /**
     * Adds data for Slider_2 to the sliders array.
     * Sets activeSlider to "Slider_2".
     * Calculates duration based on the start and end time.
     * Pushes a new object containing slider information to the sliders array.
     */
    const addData2 = () => {
      activeSlider = "Slider_2";
      const endTime = new Date();
      const duration = calculateDuration(startTime.value, endTime);
      sliders.value.push({
        name: "Slider_2",
        values: slider2.value,
        startTime,
        endTime: endTime,
        duration: duration,
      });
    };
    /**
     * Adds data for Slider_1 to the sliders array.
     * Sets activeSlider to "Slider_1".
     * Calculates duration based on the start and end time.
     * Pushes a new object containing slider information to the sliders array.
     */
    const addData1 = () => {
      activeSlider = "Slider_1";
      const endTime = new Date();
      const duration = calculateDuration(startTime.value, endTime);
      sliders.value.push({
        name: "Slider_1",
        values: slider1.value,
        startTime,
        endTime: endTime,
        duration: duration,
      });
    };
    let currentSecond = 0;
    // Set up a timer to check and reset activeSlider every second
    const timerInterval = setInterval(() => {
      currentSecond++;
      if (activeSlider) {
        graph.value[currentSecond*500] = {
          slider: activeSlider,
          value: activeSlider === "Slider_1" ? slider1.value : slider2.value,
        };
      }else{
        graph.value[currentSecond*500] = null
      }
    }, 500);
    // const timerInterval = setInterval(() => {
    //   console.log(activeSlider) // Reset activeSlider to null every second
    // }, 1000);
    return {
      title,
      description1,
      description2,
      description3,
      leftScaleLabel,
      rightScaleLabel,
      topSliderLabel,
      result,
      instruction,
      slider1,
      slider2,
      choice1,
      choice2,
      text1,
      text2,
      text3,
      text4,
      width,
      height,
      dialog,
      updateData1,
      updateData2,
      updateText1,
      updateText2,
      updateText3,
      updateText4,
      handleInput1,
      handleInput2,
      startValue1,
      startValue2,
      addData2,
      addData1,
    };
  },
  methods: {
    /**
     * Selects all text within the specified reference.
     * @param {string} refName - The name of the reference.
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
.colored-box1 {
  background-color: #e6e5ff;
}
.colored-box2 {
  background-color: #f1f4b6;
}
.right-input:deep(textarea) {
  text-align: right;
}
.center-input:deep(textarea) {
  text-align: center;
}
.title {
  color: #00d1b2;
}
.white-text:deep() {
  color: white;
}
</style>

<template>
  <Experiment title="Mental-Rotation-Homework">
    <InstructionScreen :title="'Welcome'">
      Welcome to this experiment.
      <br />
      <br />
      This was created for the course "Introduction to reproducible research practices via browser-based replication" and
      is based on the mental rotation study.
    </InstructionScreen>

    <InstructionScreen :title="'General Instructions'">
      In the following screens, you will be presented with two 3-dimensional forms from different angles. You will need
      to decide as quick and accurate as possible, whether both forms are identical objects or not.
    </InstructionScreen>

    <!-- Here we create screens in a loop for every entry in forced_choice -->
    <template v-for="(trainingTask, i) of trainingTrials">
      <KeypressScreen
        :feedback-time="1000"
        :keys="keyboardKeys"
        :fixation-time="getRandomFixation()"
        :responseTime="7500"
        :feedbackTime="800"
      >
        <template #feedback>
          <p v-if="!$magpie.measurements.hasOwnProperty('response')">Faster</p>
          <p v-else-if="$magpie.measurements.response === trainingTask.expected">
            Correct answer
          </p>
          <p v-else>
            Wrong answer
          </p>
        </template>
        <template #stimulus>
          <Record
            :data="{ ...trainingTask, trialNumber: i, trialType: 'practice' }"
          />
          <img :src="trainingTask.picture" />
        </template>
      </KeypressScreen>
    </template>
    <PostTestScreen />

    <!-- While developing your experiment, using the DebugResults screen is fine,
      once you're going live, you can use the <SubmitResults> screen to automatically send your experimental data to the server. -->
    <SubmitResultsScreen />
  </Experiment>
</template>

<script>
// Load data from csv files as javascript arrays with objects
import mainTrails from '../trials/mr_main_trials.csv';
import trainingTrials from '../trials/mr_training_trials.csv';
import _ from 'lodash';

export default {
  name: 'App',
  data() {
    return {
      mainTrails,
      trainingTrials: _.shuffle(trainingTrials),
      // Expose lodash.range to template above
      range: _.range,
      keyboardKeys: _.sample([
        { f: 'same', j: 'different' },
        { f: 'different', j: 'same' }
      ])
    };
  },
  methods: {
    getRandomFixation() {
      return _.sample([300, 270, 320, 310, 290]);
    }
  },
  mounted() {
    this.$magpie.addExpData({
      f_key: this.keyboardKeys.f,
      j_key: this.keyboardKeys.j
    });
  }
};
</script>

<template>
  <div id="app">
    <div id="desc">
      <h1>UUID Binning Algorithm</h1>

      <!-- DESCRIPTION -->
      This is an algorithm for binning UUID/GUID strings into a roughly uniform
      distribution. The essense of the algorithm is to:
      <ol>
        <li>Strip the dashes (-) out of the string<br /></li>
        <li>For each digit, calculate the char code and the hex code<br /></li>
        <li>
          Sum these digits up and take the remainder of the number of bins
        </li>
      </ol>


      <!-- CODE SNIPPET -->
      In Code:
      <pre><code>

        function binUUID(uuid) {

          const str = uuid.replaceAll("-", "");
          let n = 0;

          for (let i = 0; i &lt; str.length; i++) {
            const code = str.charCodeAt(i);
            const hex = parseInt(str.charAt(i), 16);
            n += hex + code;
          }

          return n % {{numBins}};
        }

      </code></pre>
    </div>
      <!-- CONTROLS -->
    <div id="controls">
      <label for="numBins">Number of bins</label>
      <input
        id="numBins"
        v-model="numBinsInput"
        @input="validateNumBinsInput"
      />
      <label for="numTests">Number of test UUIDS</label>
      <input
        id="numTests"
        type="text"
        v-model="numTestsInput"
        @input="validateNumTestsInput"
      />
      <button class="button is-success" @click="runTest">Run Test</button>
    </div>

    <!-- TABLE -->
    <div id="table">
      <div class="row">
        <div>Bin</div>
        <div>Number in Bin</div>
        <div>Cumulative up to Bin</div>
        <div>Number Expected up to Bin</div>
        <div>Cumulative / Expected</div>
      </div>
      <div class="row" v-for="i in binIndex" :key="i">
        <div>{{ i }}</div>
        <div>{{ bins[i] }}</div>
        <div>{{ cumulative[i] }}</div>
        <div>{{ (i + 1) * expectedPerBin }}</div>
        <div>
          {{ (cumulative[i] / ((i + 1) * expectedPerBin)).toFixed(4) }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { v4 as uuidv4 } from "uuid";
export default {
  data: () => ({
    numTests: 100000,
    numTestsInput: "100000",
    numBins: 100,
    numBinsInput: "100",
    bins: {},
  }),
  mounted() {
    this.runTest();
  },
  computed: {
    binIndex() {
      const index = [];
      for (let i = 0; i < this.numBins; i++) {
        index.push(i);
      }
      return index;
    },
    expectedPerBin() {
      return this.numTests / this.numBins;
    },
    cumulative() {
      let total = 0;
      let ary = [];
      for (let i = 0; i < this.numBins; i++) {
        const numInBin = this.bins[i] || 0;
        total += numInBin;
        ary.push(total);
      }
      return ary;
    },
  },
  methods: {
    runTest() {
      this.validateTestVals();

      this.numBins = this.numBinsInput;
      this.numTests = this.numTestsInput;
      this.bins = {};

      // Generate UUID's and bin them
      for (let i = 0; i < this.numTests; i++) {
        const bin = this.getIntFromUUID(uuidv4());
        if (this.bins[bin]) {
          this.bins[bin] += 1;
        } else {
          this.bins[bin] = 1;
        }
      }
      console.log(this.bins);
    },

    /**
     * UUID Binning algorithm
     */
    getIntFromUUID(uuid) {
      const str = uuid.replaceAll("-", "");
      let n = 0;
      for (let i = 0; i < str.length; i++) {
        const code = str.charCodeAt(i);
        const hex = parseInt(str.charAt(i), 16);
        n += hex + code;
      }

      return n % this.numBins;
    },

    validateTestVals() {
      if (this.numBinsInput < 2 || !this.numBinsInput) {
        this.numBinsInput = 2;
      }

      if (this.numBinsInput > 10000) {
        this.numBinsInput = 10000;
      }

      if (this.numTestsInput < 1000 || !this.numTestsInput) {
        this.numTestsInput = 1000;
      }

      if (this.numTestsInput > 500000) {
        this.numTestsInput = 500000;
      }
    },

    validateNumBinsInput() {
      const val = this.numBinsInput;
      const cleaned = parseInt(val.replaceAll(/[^0-9]/g, ""), 10);

      if (cleaned > 5000) {
        this.numBinsInput = 5000;
        return;
      }
      this.numBinsInput = cleaned;
    },
    validateNumTestsInput() {
      const val = this.numTestsInput;
      const cleaned = parseInt(val.replaceAll(/[^0-9]/g, ""), 10);

      if (cleaned > 500000) {
        this.numTestsInput = 500000;
        return;
      }

      this.numTestsInput = cleaned;
    },
  },
};
</script>
<style scoped>
#app {
  margin: auto;
  padding: 40px;
  text-align: left;
  font-family: "Yomogi", sans-serif;
  max-width: 1200;
}
h1 {
  font-size: 150%;
  margin-bottom: 20px;
}
#desc {
  font-size: 115%;
  max-width: 800px;
  margin: auto;
  padding-bottom: 15px;
}
#desc ol{
  padding:20px 0;
}
#table {
  margin-bottom: 20px;
  border-left: 1px solid #eee;
}
.row {
  display: flex;
}
.row div {
  width: 300px;
  border-right: 1px solid #eee;
  padding: 0 15px;
}
code {
  display: block;
}
#controls {
  display: flex;
  justify-content: center;
  padding: 20px;
  background: #eee;
}
#controls label {
  padding: 0 10px;
}
#controls button {
  margin: 0 10px;
}
</style>

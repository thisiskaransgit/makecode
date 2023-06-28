<template>
  <div id="app">
    <h1>Code Compiler</h1>
    <div>
      <select v-model="languageId" @change="changeLanguage">
        <option value="70">Python</option>
        <option value="76">C++</option>
        <option value="62">Java</option>
      </select>
    </div>
    <div>
      <Codemirror
        v-model="requestBody.source_code"
        :options="{
          mode: currentLanguage,
          theme: 'material',
          lineNumbers: true,
          line: true,
          tabSize: 4,
          indentWithTabs: true,
          smartIndent: true,
          matchBrackets: true,
          autofocus: true,
          extraKeys: { 'Ctrl-Space': 'autocomplete' },
          hintOptions: {
            tables: {
              users: { name: null, score: null, birthDate: null },
              countries: { name: null, population: null, size: null },
            },
          },
        }"
      />
    </div>
    <div>
      <button @click="makeSubmission">Run</button>
    </div>
    <div v-if="isLoading">
      <h3>Loading...</h3>
    </div>
    <!-- Print Response -->
    <div v-if="response">
      <h3>Output</h3>
      <p>{{ response }}</p>
    </div>
  </div>

</template>

<script setup>
import { ref } from "vue";
import { Codemirror } from "vue-codemirror";
import axios from "axios";

const isLoading = ref(false);
const languageId = ref(70);
const currentLanguage = ref("python");
const baseURL = ref("http://localhost:2358");
const languageMap = ref({
  70: "python",
  76: "cpp",
  62: "java",
});
const requestBody = ref({
  source_code: 'print "hello in python simplecoding"',
  language_id: "70",
  number_of_runs: null,
  stdin: "Judge0",
  expected_output: null,
  cpu_time_limit: null,
  cpu_extra_time: null,
  wall_time_limit: null,
  memory_limit: null,
  stack_limit: null,
  max_processes_and_or_threads: null,
  enable_per_process_and_thread_time_limit: null,
  enable_per_process_and_thread_memory_limit: null,
  max_file_size: null,
  enable_network: null,
});
const token = ref(null);
const response = ref(null);
const waitFor3second = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve("done");
    }, 3000);
  });
};

// improve the code
const makeSubmission = () => {
  isLoading.value = true;
  console.log("requestBody", requestBody.value);
  axios
    .post(`${baseURL.value}/submissions/`, requestBody.value)
    .then((res) => {
      console.log("res", res);
      token.value = res.data.token;
      return waitFor3second();
    })
    .then((res) => {
      return axios.get(`${baseURL.value}/submissions/${token.value}`);
    })
    .then((res) => {
      console.log("res", res);
      response.value = res.data.stdout;
      isLoading.value = false;
    })
    .catch((err) => {
      console.log("err", err);
      isLoading.value = false;
    });
};


// on selecting different language
const changeLanguage = () => {
  currentLanguage.value = languageMap.value[languageId.value];
  requestBody.value.language_id = languageId.value;
  console.log("currentLanguage", currentLanguage.value);
  console.log("requestBody", requestBody.value);
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>

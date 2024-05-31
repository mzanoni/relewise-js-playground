<template>
  <main class="page">
    <div style="width: 50%; display: flex;  flex-direction: column;">

      <div style="display: flex; flex-direction: column; margin-bottom: 20px; padding-left: 15px; padding-right: 15px;">
        <div class="inputs">
          <div>
            <label for="datasetid">Dataset Id</label>
            <input type="text" id="datasetid" autocomplete="off" v-model="datasetId">
          </div>

          <div>
            <label for="apikey">Api Key</label>
            <input type="password" id="apikey" autocomplete="off" v-model="apiKey">
          </div>

          <div>
            <label for="serverurl">Server Url</label>
            <input type="text" id="serverurl" autocomplete="off" v-model="serverUrl">
          </div>

        </div>

        <div style="text-align: right; margin-top: 15px;">
          <button @click="runCode" :disabled="runningCode">
            <svg width="18" height="18" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg" fill="none"
              stroke="currentcolor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"
              class="feather feather-terminal">
              <polyline points="4 17 10 11 4 5"></polyline>
              <line x1="12" y1="19" x2="20" y2="19"></line>
            </svg>
            {{ runningCode ? 'Executing' : 'Run' }}
          </button>
        </div>

      </div>

      <CodeMirror v-model="jsCode" :lang="javascriptLang" wrap basic class="code-wrapper" />
    </div>

    <div class="output">
      <div style="color: green;">// Server Time in MS:
        <template v-if="statistics">{{ statistics.serverTimeInMs }}</template>
        <i v-else>no request made yet</i>
      </div>
      <pre><template
    v-if="result?.success">{{ result.value ?? 'The code must contain a single return statement to show results' }}</template><template
    v-else-if="result?.error && result.error.kind !== 'error-eval-timeout'"><span style="color: red;">{{ result.error.error }}</span></template><template
    v-else>... run query to see results</template></pre>
    </div>
  </main>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { createInitminalRun, defaultSafeObjects, type EvalResult } from '@initminal/run';
import CodeMirror from 'vue-codemirror6';
import { javascript } from '@codemirror/lang-javascript';
import type { TimedResponse, Statistics } from '@relewise/client';

const InitminalRun = createInitminalRun({ whitelists: ['fetch'].concat(defaultSafeObjects) });
const jsCode = ref(
  `import { UserFactory, ProductSearchBuilder, Searcher } from '@relewise/client';

const settings = {
    language: 'en-gb',
    currency: 'DKK',
    displayedAtLocation: 'Search Page',
    user: UserFactory.anonymous()
};

const builder = new ProductSearchBuilder(settings)
    .setSelectedProductProperties({
        displayName: true,
        dataKeys: ['Description', 'ImageUrl']
    });

const searcher = new Searcher(RELEWISE_DATASET_ID, RELEWISE_API_KEY, {
    serverUrl: RELEWISE_SERVER_URL,
});

return await searcher.searchProducts(builder.build());`);
const result = ref<EvalResult>();
const runningCode = ref(false);
const statistics = ref<Statistics | null | undefined>(null);

const javascriptLang = javascript();

const datasetId = ref('');
const apiKey = ref('');
const serverUrl = ref('https://api.relewise.com/');

async function runCode() {
  if (!jsCode.value) return;

  statistics.value = null;

  if (!apiKey.value) {
    result.value = {
      success: false,
      error: {
        kind: 'error-eval-get-dependencies',
        error: 'Api Key is required'
      }
    }
    return;
  }
  if (!apiKey.value) {
    result.value = {
      success: false,
      error: {
        kind: 'error-eval-get-dependencies',
        error: 'Dataset Id is required'
      }
    }
    return;
  }

  if (!serverUrl.value) {
    result.value = {
      success: false,
      error: {
        kind: 'error-eval-get-dependencies',
        error: 'Server Url is required'
      }
    }
    return;
  }

  runningCode.value = true;

  const regex = new RegExp(/import([ \n\t]*(?:[^ \n\t\{\}]+[ \n\t]*,?)?(?:[ \n\t]*\{(?:[ \n\t]*[^ \n\t"'\{\}]+[ \n\t]*,?)+\})?[ \n\t]*)from[ \n\t]*(['"])([^'"\n]+)(?:['"])/gi);
  const matches = jsCode.value.matchAll(regex);
  const importStatements: string[] = [];

  let innerCode = jsCode.value;
  for (const importStatement of matches ?? []) {
    innerCode = innerCode.replace(importStatement[0], '');
    importStatements.push(importStatement[0]);
  }

  const jsToBeRun = `
${importStatements.join(';')}
const RELEWISE_DATASET_ID = '${datasetId.value}';
const RELEWISE_API_KEY = '${apiKey.value}';
const RELEWISE_SERVER_URL = '${serverUrl.value}';
    
export const initminal = async () => {
    ${innerCode}
}`;

  result.value = await InitminalRun.run(
    jsToBeRun,
    {
      "@relewise/client": 'https://cdn.jsdelivr.net/npm/@relewise/client@latest/+esm',
      "@relewise/integrations": 'https://cdn.jsdelivr.net/npm/@relewise/integrations@latest/+esm',
    });
  runningCode.value = false;
  if (result.value.success) {
    statistics.value = (result.value.value as TimedResponse)!.statistics;
  }
}

</script>

<style>
textarea {
  width: 100%;
}

.output {
  width: 50%;
  display: flex;
  flex-direction: column;
  background-color: #eeeeee;
  padding: 10px 0px 0px 20px;
}

pre {
  flex: 1 1 auto;
  overflow-y: auto;
  height: 0px;
  margin-bottom: 0;
  text-wrap: wrap;
}

.code-wrapper {
  flex: 1 1 auto;
  overflow-y: auto;
  height: 0px;
}

.page {
  display: flex;
  flex-grow: 1;
  max-height: 100%;
}

div[data-lastpass-icon-root] {
  display: none;
}
</style>

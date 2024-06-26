<template>
  <div class="flex flex-col gap-y-6">
    <Panel
      v-for="pipelineConfig in pipelineConfigs || []"
      :key="pipelineConfig.hash"
      :collapsable="pipelineConfigs && pipelineConfigs.length > 1"
      collapsed-by-default
      :title="pipelineConfigs && pipelineConfigs.length > 1 ? pipelineConfig.name : ''"
    >
      <SyntaxHighlight class="font-mono whitespace-pre overflow-auto" language="yaml" :code="pipelineConfig.data" />
    </Panel>
  </div>
</template>

<script lang="ts" setup>
import { decode } from 'js-base64';
import { inject, onMounted, Ref, ref, watch } from 'vue';

import SyntaxHighlight from '~/components/atomic/SyntaxHighlight';
import Panel from '~/components/layout/Panel.vue';
import useApiClient from '~/compositions/useApiClient';
import { Pipeline, PipelineConfig, Repo } from '~/lib/api/types';

const pipeline = inject<Ref<Pipeline>>('pipeline');
const apiClient = useApiClient();
const repo = inject<Ref<Repo>>('repo');
if (!repo || !pipeline) {
  throw new Error('Unexpected: "repo" & "pipeline" should be provided at this place');
}

const pipelineConfigs = ref<PipelineConfig[]>();
async function loadPipelineConfig() {
  if (!repo || !pipeline) {
    throw new Error('Unexpected: "repo" & "pipeline" should be provided at this place');
  }

  pipelineConfigs.value = (await apiClient.getPipelineConfig(repo.value.id, pipeline.value.number)).map((i) => ({
    ...i,
    data: decode(i.data),
  }));
}

onMounted(() => {
  loadPipelineConfig();
});

watch(pipeline, () => {
  loadPipelineConfig();
});
</script>

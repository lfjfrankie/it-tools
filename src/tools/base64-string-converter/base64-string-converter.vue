<script setup lang="ts">
import JSON5 from 'json5';
import { useStorage } from '@vueuse/core';
import { useCopy } from '@/composable/copy';
import { base64ToText, isValidBase64, textToBase64 } from '@/utils/base64';
import { withDefaultOnError } from '@/utils/defaults';
import { formatJson } from './json.models';
import { useValidation } from '@/composable/validation';
import TextareaCopyable from '@/components/TextareaCopyable.vue';

const inputElement = ref<HTMLElement>();

const encodeUrlSafe = useStorage('base64-string-converter--encode-url-safe', false);
const decodeUrlSafe = useStorage('base64-string-converter--decode-url-safe', false);

const convTypes = [{ label: 'String to base64', value: 's2b64'}, {label: 'Base64 to string', value: 'b642s'}] as const;
const convType = useStorage<typeof convTypes[number]['value']>('base64-string-converter--conv-type', convTypes[0].value);

const formats = [{ label: 'Raw', value: 'raw' }, { label: 'JSON', value: 'json' }] as const;
const format = useStorage<typeof formats[number]['value']>('base64-string-converter--format', formats[0].value);

const textInput = ref('');
const base64Output = computed(() => textToBase64(textInput.value, { makeUrlSafe: encodeUrlSafe.value }));
const { copy: copyTextBase64 } = useCopy({ source: base64Output, text: 'Base64 string copied to the clipboard' });

const base64Input = ref('');
const textOutput = computed(() =>
  withDefaultOnError(() => base64ToText(base64Input.value.trim(), { makeUrlSafe: decodeUrlSafe.value }), '')
);
const indentSize = useStorage('json-prettify:indent-size', 4);
const sortKeys = useStorage('json-prettify:sort-keys', true);
const cleanJson = computed(() => withDefaultOnError(() => {
    const rawJson = base64ToText(base64Input.value.trim(), { makeUrlSafe: decodeUrlSafe.value });
    return formatJson({ rawJson, indentSize, sortKeys})
  }, 'Provided JSON is not valid.')
);

const { copy: copyText } = useCopy({ source: textOutput, text: 'String copied to the clipboard' });
const b64ValidationRules = [
  {
    message: 'Invalid base64 string',
    validator: (value: string) => isValidBase64(value.trim(), { makeUrlSafe: decodeUrlSafe.value }),
  },
];
const b64ValidationWatch = [decodeUrlSafe];
</script>

<template>
  <div style="flex: 0 0 100%">
    <c-buttons-select v-model:value="convType" :options="convTypes" label="Type: " label-width="75px" />
  </div>
  <c-card v-if="convType === 's2b64'">
    <n-form-item label="Encode URL safe" label-placement="left">
      <n-switch v-model:value="encodeUrlSafe" />
    </n-form-item>
    <c-input-text
      v-model:value="textInput"
      label="String to encode"
      multiline
      placeholder="Put your string here..."
      rows="20"
      raw-text
      mb-5
    />
  </c-card>

  <c-card v-if="convType === 's2b64'">
    <c-input-text
      :value="base64Output"
      label="Base64 of string"
      multiline
      readonly
      placeholder="The base64 encoding of your string will be here"
      rows="20"
      mb-5
    />
    <div flex justify-center>
      <c-button @click="copyTextBase64()">
        Copy base64
      </c-button>
    </div>
  </c-card>

  <c-card v-if="convType === 'b642s'">
    <n-form-item label="Decode URL safe" label-placement="left">
      <n-switch v-model:value="decodeUrlSafe" />
    </n-form-item>
    <c-input-text
      v-model:value="base64Input"
      multiline
      placeholder="Your base64 string..."
      rows="20"
      :validation-rules="b64ValidationRules"
      :validation-watch="b64ValidationWatch"
      label="Base64 string to decode"
      mb-5
    />
  </c-card>
  <c-card v-if="convType === 'b642s'">
    <c-buttons-select v-model:value="format" :options="formats" label="Format: " label-width="75px" />
    <div v-if="format === 'json'">
      <div style="margin: 0 auto; max-width: 600px" flex justify-center gap-3>
        <n-form-item label="Sort keys :" label-placement="left" label-width="100">
          <n-switch v-model:value="sortKeys" />
        </n-form-item>
        <n-form-item label="Indent size :" label-placement="left" label-width="100" :show-feedback="false">
          <n-input-number v-model:value="indentSize" min="0" max="10" style="width: 100px" />
        </n-form-item>
      </div>
      <n-form-item>
        <TextareaCopyable 
          :value="cleanJson" 
          language="json" 
          :follow-height-of="inputElement"/>
        </n-form-item>
    </div>

    <div v-if="format === 'raw'">
      <c-input-text
        v-model:value="textOutput"
        label="Decoded string"
        placeholder="The decoded string will be here"
        multiline
        rows="20"
        readonly
        mb-5
      />

      <div flex justify-center>
        <c-button @click="copyText()">
          Copy decoded string
        </c-button>
      </div>
    </div>
  </c-card>
</template>

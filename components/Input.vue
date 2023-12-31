<template>
  <label class="text-left block text-sm font-medium leading-6 text-gray-900">{{ label }}</label>
  <div class="mt-2">
    <component
      :is="variant ?? 'input'"
      :id="id"
      :ref="(el: any) => {
        if(el && blockPaste) el.onpaste = (e: Event) => e.preventDefault();
        if(el && blockCopy) el.oncopy = (e: Event) => e.preventDefault();
      }"
      :name="name"
      :type="type ?? 'text'"
      :placeholder="placeholder"
      :class="`
        block
        w-full
        rounded-md
        border-0
        py-1.5
        text-gray-900
        shadow-sm
        ring-1
        ring-inset
        ring-gray-300
        placeholder:text-gray-400
        focus:ring-2
        focus:ring-inset
        focus:ring-primary-600
        sm:text-sm
        sm:leading-6
        ${isError && 'border-2 focus:ring-red-500 border-red-500 focus:border-red-500'}
        ${isError && 'border-2 focus:ring-[#42d392] border-[#42d392]'}
        ${disabled ? 'disabled:cursor-not-allowed disabled:bg-gray-50 disabled:text-gray-500 disabled:ring-gray-200' : ''}
      `"
      :value="value"
      :disabled="disabled"

      @input="_onChange($event)"

      @blur="_onBlur($event)"
    />

    <p
      v-if="showHelperText || _showHelperText"
      class="text-sm mt-1"
      :class="{'text-red-500': isError || _isError}"
    >
      {{ helperText }}
    </p>
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted, onUpdated, } from "vue";

const _value = ref("");
const _isError = ref(false);
const _showHelperText = ref(false);

const props = defineProps<{
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  onChange?:(value: string, event: any) => void;
  variant?: string;
  id?: string;
  name?: string;
  value?: string;
  label?: string;
  type?: string;
  placeholder?: string;
  isError?: boolean;
  disabled?: boolean;
  showHelperText?: boolean;
  helperText?: string;
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  onBlur?: (value: string, event: any) => void;
  validateWithOnBlur?: boolean;
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  validateWhen?: (value: string) => boolean;
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  validate?: (value: string) => boolean;
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  onValidate?: (isError: boolean) => void;
  validateOnUpdate?: boolean;
  blockPaste?: boolean;
  blockCopy?: boolean;
}>();

onMounted(() => {
  _value.value = props?.value ?? "";
});

function _onValidate () {
  if (props.onValidate && props.validate) {
    let error = true;

    if (props.validate(_value.value)) {
      error = false;
    }

    props?.onValidate(error);

    _showHelperText.value = error;
    _isError.value = error;
  }
}

function _onChange (event: any) {
  _value.value = event.target.value;

  if (props?.onChange) {
    props?.onChange(event.target.value, event);

    if (props.onValidate) {
      if (props.validateWhen && props.validateWhen(_value.value)) {
        _onValidate();
      }
    }
  }
}

function _onBlur (event: any) {
  if (props.onBlur) {
    props?.onBlur(event.target.value, event);

    if (props.validateWithOnBlur) {
      _onValidate();
    }
  }
}

onUpdated(() => {
  if (props.validateOnUpdate) {
    _onValidate();
  }
});

</script>

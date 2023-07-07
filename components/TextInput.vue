<script setup>

    const props = defineProps(['input', 'inputType', 'placeholder', 'error', 'max']);

    const { inputType, placeholder, error, input, max } = toRefs(props);

    const emit = defineEmits(['update:input']);

    const isFocused = ref(false);
    const inputComputed = computed({
        get: () => input.value,
        set: (val) => emit('update:input', val) 
    })

</script>

<template>
    <div>
        <client-only>
            <input :type="inputType" :placeholder="placeholder" class="w-full bg-white text-gray-800 border text-sm border-[#eff0eb] rounded-lg p-3 placeholder-gray-500 focus:outline-none" :maxlength="max" @focus="isFocused=true" @blur="isFocused=false" :class="{'border-gray-900': isFocused, 'border-red-500': error}" v-model="inputComputed" autocomplete="off" />
        </client-only>
        <span v-if="error" class="text-red-500 text-[14px] font-semibold ml-2">
            {{ error }}
        </span>
    </div>
    

</template>
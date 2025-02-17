<script setup>
import { decodeTypes, formatTypes } from '@/consts/value_view_type.js'
import Code from '@/components/icons/Code.vue'
import Conversion from '@/components/icons/Conversion.vue'
import DropdownSelector from '@/components/common/DropdownSelector.vue'
import { includes, isEmpty, map, pull, some, values } from 'lodash'
import { computed } from 'vue'
import usePreferencesStore from 'stores/preferences.js'
import useDialogStore from 'stores/dialog.js'

const props = defineProps({
    decode: {
        type: String,
        default: decodeTypes.NONE,
    },
    format: {
        type: String,
        default: formatTypes.RAW,
    },
    disabled: Boolean,
})

const prefStore = usePreferencesStore()
const dialogStore = useDialogStore()

const formatTypeOption = computed(() => {
    return map(formatTypes, (t) => t)
})

const decodeTypeOption = computed(() => {
    const buildinTypes = [decodeTypes.NONE],
        customTypes = []
    const typs = values(decodeTypes)
    // build-in decoder
    for (const typ of typs) {
        if (includes(prefStore.buildInDecoder, typ)) {
            buildinTypes.push(typ)
        }
    }
    // custom decoder
    if (!isEmpty(prefStore.decoder)) {
        for (const decoder of prefStore.decoder) {
            // replace build-in decoder if name conflicted
            pull(buildinTypes, decoder.name)
            customTypes.push(decoder.name)
        }
    }
    return [buildinTypes, customTypes]
})

const decodeMenuOption = computed(() => {
    return [
        {
            key: 'new_rdm_decoder',
            label: 'interface.custom_decoder',
        },
    ]
})

const emit = defineEmits(['formatChanged', 'update:decode', 'update:format'])
const onFormatChanged = (selDecode, selFormat) => {
    const [buildin, external] = decodeTypeOption.value
    if (!some([...buildin, ...external], (val) => val === selDecode)) {
        selDecode = decodeTypes.NONE
    }
    if (!some(formatTypes, (val) => val === selFormat)) {
        // set to auto chose format
        selFormat = ''
    }
    emit('formatChanged', selDecode, selFormat)
    if (selDecode !== props.decode) {
        emit('update:decode', selDecode)
    }
    if (selFormat !== props.format) {
        emit('update:format', selFormat)
    }
}

const onDecodeMenu = (key) => {
    switch (key) {
        case 'new_rdm_decoder':
            dialogStore.openPreferencesDialog('decoder')
            break
    }
}
</script>

<template>
    <n-space :size="0" :wrap="false" :wrap-item="false" align="center" justify="start">
        <dropdown-selector
            :default="formatTypes.RAW"
            :disabled="props.disabled"
            :icon="Code"
            :options="formatTypeOption"
            :tooltip="$t('interface.view_as')"
            :value="props.format || formatTypes.RAW"
            @update:value="(f) => onFormatChanged(props.decode, f)" />
        <n-divider vertical />
        <dropdown-selector
            :default="decodeTypes.NONE"
            :disabled="props.disabled"
            :icon="Conversion"
            :menu-option="decodeMenuOption"
            :options="decodeTypeOption"
            :tooltip="$t('interface.decode_with')"
            :value="props.decode || decodeTypes.NONE"
            @menu="onDecodeMenu"
            @update:value="(d) => onFormatChanged(d, '')" />
    </n-space>
</template>

<style lang="scss" scoped></style>

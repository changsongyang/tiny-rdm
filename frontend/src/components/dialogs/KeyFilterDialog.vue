<script setup>
import { computed, reactive, ref, watch } from 'vue'
import useDialog from '../../stores/dialog'
import { useI18n } from 'vue-i18n'
import useConnectionStore from '../../stores/connections.js'
import { types } from '../../consts/support_redis_type.js'

const i18n = useI18n()
const filterForm = reactive({
    server: '',
    db: 0,
    type: '',
    pattern: '',
})
const filterFormRef = ref(null)
const typeOptions = computed(() => {
    const options = Object.keys(types).map((t) => ({
        value: t,
        label: t,
    }))
    options.splice(0, 0, {
        value: '',
        label: i18n.t('all'),
    })
    return options
})

const formLabelWidth = '100px'
const dialogStore = useDialog()
watch(
    () => dialogStore.keyFilterDialogVisible,
    (visible) => {
        if (visible) {
            const { server, db, type, pattern } = dialogStore.keyFilterParam
            filterForm.server = server
            filterForm.db = db || 0
            filterForm.type = type || ''
            filterForm.pattern = pattern || '*'
        }
    }
)

const connectionStore = useConnectionStore()
const onConfirm = () => {
    const { server, db, type, pattern } = filterForm
    connectionStore.setKeyFilter(server, db, pattern, type)
    connectionStore.reopenDatabase(server, db)
}

const onClose = () => {
    dialogStore.closeKeyFilterDialog()
}
</script>

<template>
    <n-modal
        v-model:show="dialogStore.keyFilterDialogVisible"
        :closable="false"
        :close-on-esc="false"
        :mask-closable="false"
        :negative-button-props="{ size: 'medium' }"
        :negative-text="$t('cancel')"
        :positive-button-props="{ size: 'medium' }"
        :positive-text="$t('confirm')"
        :show-icon="false"
        :title="$t('set_key_filter')"
        preset="dialog"
        style="width: 450px"
        transform-origin="center"
        @positive-click="onConfirm"
        @negative-click="onClose"
    >
        <n-form
            ref="filterFormRef"
            :label-width="formLabelWidth"
            :model="filterForm"
            :show-require-mark="false"
            label-align="right"
            label-placement="left"
            style="padding-right: 15px"
        >
            <n-form-item :label="$t('server')" path="key">
                <n-text>{{ filterForm.server }}</n-text>
            </n-form-item>
            <n-form-item :label="$t('db_index')" path="db">
                <n-text>{{ filterForm.db }}</n-text>
            </n-form-item>
            <n-form-item :label="$t('type')" path="type" required>
                <n-select v-model:value="filterForm.type" :options="typeOptions" />
            </n-form-item>
            <n-form-item :label="$t('filter_pattern')" required>
                <n-input-group>
                    <n-tooltip trigger="focus">
                        <template #trigger>
                            <n-input v-model:value="filterForm.pattern" placeholder="Filter Pattern" clearable />
                        </template>
                        <div class="text-block">{{ $t('filter_pattern_tip') }}</div>
                    </n-tooltip>
                    <n-button secondary type="primary" @click="filterForm.pattern = '*'">
                        {{ $t('restore_defaults') }}
                    </n-button>
                </n-input-group>
            </n-form-item>
        </n-form>
    </n-modal>
</template>

<style lang="scss" scoped></style>
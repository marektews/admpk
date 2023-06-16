<script setup>
import { ref, computed } from 'vue'

defineEmits(['ok'])
const props = defineProps(['record'])
const confirmationData = ref('')

const isOkEnabled = computed(() => {
    return confirmationData.value === props.record?.regnum1
})
</script>

<template>
    <div class="modal fade" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5>Kasowanie</h5>
                </div>

                <div class="modal-body">
                    <p>
                        <b>Czy na pewno skasować ten wpis?</b>
                    </p>
                    <p>
                        <div>Potwierdź wpisując numer rejestracyjny pojazdu: <b>{{ props.record?.regnum1 }}</b></div>
                        <div>
                            <input class="form-control" v-model="confirmationData" type="text" />
                        </div>
                    </p>
                </div>

                <div class="modal-footer">
                    <button 
                        type="button"
                        class="btn btn-secondary"
                        data-bs-dismiss="modal"
                        aria-label="Zamknij"
                    >
                        Zamknij
                    </button>
                    <button
                        type="button"
                        class="btn btn-danger"
                        data-bs-dismiss="modal"
                        :disabled="!isOkEnabled"
                        @click="$emit('ok', props.record)"
                    >
                        <i class="fa-solid fa-trash" />
                        Usuń
                    </button>
                </div>
            </div>
        </div>
    </div>
</template>

<style>
@media (prefers-color-scheme: dark) {
    div.modal-content {
        background-color: var(--color-background) !important;
    }

    .btn-close {
        color: var(--color-text) !important;
    }
}
</style>
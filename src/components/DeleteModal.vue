<script setup>
import { ref, computed } from 'vue'
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome';
import { faTrash } from '@fortawesome/free-solid-svg-icons';

const emit = defineEmits(['ok'])
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
                    <div>
                        <div>Potwierdź wpisując numer rejestracyjny pojazdu: <b>{{ props.record?.regnum1 }}</b></div>
                        <div class="mt-2">
                            <input 
                                class="form-control" 
                                v-model="confirmationData" 
                                type="text"
                            >
                        </div>
                    </div>
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
                        @click="confirmationData = ''; emit('ok', props.record);"
                    >
                        <FontAwesomeIcon :icon="faTrash" /> Usuń
                    </button>
                </div>
            </div>
        </div>
    </div>
</template>

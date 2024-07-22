<script setup>
import { ref, watch, computed, onMounted, onUnmounted } from 'vue'
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome';
import { faMagnifyingGlass, faXmark, faTrash } from '@fortawesome/free-solid-svg-icons';
import DeleteModal from './components/DeleteModal.vue'

const pk_list_orig = ref(undefined)
const pk_list_filtered = ref(undefined)
const departments_list = ref([])
const deletingItem = ref(undefined)
const search = ref('')
const timer = ref(null)
const filter_tura = ref('ALL')

const pk_list = computed(() => {
    return pk_list_filtered.value != undefined ? pk_list_filtered.value : (pk_list_orig.value || [])
})

onMounted(() => {
    fetch('/api/pk/hints')
    .then(response => response.json())
    .then(d => departments_list.value = d)
    .catch(err => console.error('Load all departments list exception:', err))

    load_all_pk()
})

onUnmounted(() => {
    if(timer.value != null) {
        clearTimeout(timer.value)
        timer.value = null
    }
})

watch(search, (nv) => {
    if(timer.value != null)
        clearTimeout(timer.value)
    timer.value = setTimeout(() => {
        filtering(nv, filter_tura.value)
        timer.value = null
    }, 500)
})

watch(filter_tura, (nv) => {
    if(timer.value != null)
        clearTimeout(timer.value)
    timer.value = setTimeout(() => {
        filtering(search.value, nv)
        timer.value = null
    }, 500)
})

function filtering(pattern, tura) {
    let list = []

    if(pattern.length) {
        pattern = pattern.toLowerCase()
        list = pk_list_orig.value.filter((item) => {
            if(tura !== "ALL") {
                let dep_tura = department_tura(item.dep_id)
                if(dep_tura !== tura)
                    return false
            }

            if(item.regnum1.toLowerCase().includes(pattern)) return true
            if(item.regnum2?.toLowerCase().includes(pattern)) return true
            if(item.regnum3?.toLowerCase().includes(pattern)) return true
            if(item.pass_nr == pattern) return true
            
            let tmp = department_info(item.dep_id)
            if(tmp?.toLowerCase().includes(pattern)) return true
            return false
        })
    }
    else {
        list = pk_list_orig.value.filter((item) => {
            if(tura !== "ALL") {
                let dep_tura = department_tura(item.dep_id)
                if(dep_tura !== tura)
                    return false
            }
            return true
        })
    } 
    
    list.sort((a,b) => {
        let dep_tura_a = department_tura(a.dep_id)
        let dep_tura_b = department_tura(b.dep_id)
        let tmp = dep_tura_a.localeCompare(dep_tura_b)
        if(tmp !== 0) return tmp

        let zb_name_a = department_name(a.dep_id)
        let zb_name_b = department_name(b.dep_id)
        return zb_name_a.localeCompare(zb_name_b)
    })

    pk_list_filtered.value = list
}

function load_all_pk() {
    fetch('/api/pk/all')
    .then(response => response.json())
    .then(d => {
        pk_list_orig.value = d
        if(search.value.length > 0)
            filtering(search.value, filter_tura.value)
    })
    .catch(err => console.error('Load all PK exception:', err))
}

function department_info(dep_id) {
    let d = departments_list.value.find((item) => item.id === dep_id)
    if(d == undefined) return ""
    return `${d.lang} - ${d.name}`
}

function department_tura(dep_id) {
    let d = departments_list.value.find((item) => item.id === dep_id)
    if(d == undefined) return ""
    return `W${d.tura}`
}

function department_name(dep_id) {
    let d = departments_list.value.find((item) => item.id === dep_id)
    if(d == undefined) return ""
    return d.name
}

function onStartDelete(pk) {
    console.log('Start deleting:', pk)
    deletingItem.value = pk
}
function onDelete(pk) {
    console.log('Deleting:', pk)

    fetch(`/api/pk/delete/${deletingItem.value.id}`)
    .then(response => {
        if(response.status === 200) {
            pk_list_filtered.value = undefined
            pk_list_orig.value = undefined
            load_all_pk()
        }
    })
    .catch(err => console.error('Delete PK exception:', err))
}
</script>

<template>
    <header>
        <div class="d-flex flex-row align-items-center">
            <img src="@/assets/Parking_icon.svg" width="40" height="40" />
            <span class="ms-2 title">Parking Torwar - tryb moderatora</span>
        </div>
        <div class="tura-radio-group">
            <div class="form-check">
                <input v-model="filter_tura" value="ALL" class="form-check-input" type="radio" name="tura" id="tura_all">
                <label class="form-check-label" for="tura_all">Wszystko</label>
            </div>
            <div class="form-check">
                <input v-model="filter_tura" value="W2" class="form-check-input" type="radio" name="tura" id="tura_w2">
                <label class="form-check-label" for="tura_w2">W2</label>
            </div>
            <div class="form-check">
                <input v-model="filter_tura" value="W3" class="form-check-input" type="radio" name="tura" id="tura_w3">
                <label class="form-check-label" for="tura_w3">W3</label>
            </div>
        </div>
        <div>
            <div class="input-group ">
                <span class="input-group-text">
                    <FontAwesomeIcon :icon="faMagnifyingGlass" />
                </span>
                <input
                    v-model="search"
                    type="text"
                    class="form-control"
                    placeholder="Wpisz ciąg filtrowania" 
                />
                <button class="btn btn-secondary" @click="search = ''">
                    <FontAwesomeIcon :icon="faXmark" />
                </button>
            </div>
        </div>
    </header>

    <main class="mt-3">
        <table class="table table-bordered">
            <thead>
                <tr>
                    <th rowspan="2">#</th>
                    <th rowspan="2">Dział</th>
                    <th rowspan="2">Tura</th>
                    <th rowspan="2">Nr identyfikatora</th>
                    <th colspan="3">Nr rejestracyjny pojazdu</th>
                    <th rowspan="2">Ops</th>
                </tr>
                <tr>
                    <th>Piątek</th>
                    <th>Sobota</th>
                    <th>Niedziela</th>
                </tr>
            </thead>
            <tbody v-if="pk_list_orig === undefined">
                <tr>
                    <td colspan="7">
                        <div class="d-inline-flex flex-row align-items-center">
                            <div class="spinner-border" role="status" />
                            <div class="ms-3">Proszę czekać. Trwa ładowanie danych ...</div>
                        </div>
                    </td>
                </tr>
            </tbody>
            <tbody v-else>
                <tr v-for="(pk, index) in pk_list" :key="index">
                    <th>{{ index+1 }}</th>
                    <td>{{ department_info(pk.dep_id) }}</td>
                    <td>{{ department_tura(pk.dep_id) }}</td>
                    <td>{{ pk.pass_nr }}</td>
                    <template v-if="pk.regnum2?.length">
                        <td>{{ pk.regnum1 }}</td>
                        <td>{{ pk.regnum2 }}</td>
                        <td>{{ pk.regnum3 }}</td>
                    </template>
                    <template v-else>
                        <td colspan="3">{{ pk.regnum1 }}</td>
                    </template>
                    <td>
                        <button type="button"
                            class="btn btn-danger btn-sm" 
                            title="Kasowanie"
                            @click="onStartDelete(pk)"
                            data-bs-toggle="modal"
                            data-bs-target="#deleteModal"
                        >
                            <FontAwesomeIcon :icon="faTrash" />
                        </button>
                    </td>
                </tr>
            </tbody>
        </table>

    </main>

    <DeleteModal
        id="deleteModal"
        :record="deletingItem"
        @ok="onDelete"
    />
</template>

<style scoped>
header {
    line-height: 1.5;
    place-items: center;
    display: flex;
    flex-direction: row;
    justify-content: space-between;
}

.title {
    font-size: 1.5rem;
}

td, th {
    vertical-align: middle;
}

.tura-radio-group {
    display: flex;
    flex-direction: row;
    gap: 20pt;
}
</style>

<script setup>
import {useToast} from 'primevue/usetoast'
import { FilterMatchMode } from 'primevue/api'
import { ref, onMounted, onBeforeMount, watch } from 'vue'
import axios from "axios";
import {useRouter} from "vue-router";
const router = useRouter()
const countries = ref([])
const toast = useToast()
const price_id = ref(null)
const loading = ref(true)
const delete_id = ref('')
const model = ref({})
const status = ref(true)
const error = ref('')
const models = ref(null)
const productDialog = ref(false)
const deleteDialog = ref(false)
const deleteModelsDialog = ref(false)
const product = ref({})
const selectedModels = ref(null)
const dt = ref(null)
const filters = ref({})
const searchQuery = ref('')

// Pagination variables
const currentPage = ref(1)
const totalRecords = ref(0)
const rowsPerPage = ref(10)
const totalPages = ref(0)
const firstPageUrl = ref('')
const lastPageUrl = ref('')
const nextPageUrl = ref('')
const prevPageUrl = ref('')
const from = ref(0)
const to = ref(0)
const links = ref([])

const exportCSV = () => {
  dt.value.exportCSV()
}

const delet = (id) => {
  delete_id.value = id
  deleteModelsDialog.value = true
}

const deleteSelectedModels = () => {
  axios.delete(`/api/model/${delete_id.value}`)
    .then(() => {
      fetchData()
      deleteModelsDialog.value = false
      toast.add({severity: 'success', summary: 'Successful', detail: 'Model Deleted', life: 3000})
    })
}

const initFilters = () => {
  filters.value = {
    global: {value: null, matchMode: FilterMatchMode.CONTAINS},
  }
}

onBeforeMount(() => {
  initFilters()
})

const fetchData = () => {
  loading.value = true
  axios.get("/api/model", {
    params: {
      page: currentPage.value,
      limit: rowsPerPage.value,
      search: searchQuery.value
    }
  }).then((res) => {
    loading.value = false
    models.value = res.data.data.data
    totalRecords.value = res.data.data.total
    totalPages.value = res.data.data.last_page
    firstPageUrl.value = res.data.data.first_page_url
    lastPageUrl.value = res.data.data.last_page_url
    nextPageUrl.value = res.data.data.next_page_url
    prevPageUrl.value = res.data.data.prev_page_url
    from.value = res.data.data.from
    to.value = res.data.data.to
    links.value = res.data.data.links
  }).catch(error => {
    loading.value = false
    console.error("Error fetching data:", error)
  })
}

watch(searchQuery, (newVal) => {
  currentPage.value = 1
  fetchData()
})

const goToPage = (page) => {
  if (page >= 1 && page <= totalPages.value) {
    currentPage.value = page
    fetchData()
  }
}

const changeRowsPerPage = (rows) => {
  rowsPerPage.value = rows.value
  currentPage.value = 1
  fetchData()
}

onMounted(() => {
  fetchData()
})

const openNew = () => {
  router.push({name:'model-create'})
}

const confirmEdit = (id) => {
  router.push({name:'model-edit',params:{id:id}})
};
</script>

<template>
  <div class="grid">
    <div class="col-12">
      <va-card class="card">
        <Toolbar class="mb-4">
          <template #start>
            <div class="my-2 ">
              <Button v-can="'create models'" label="New" icon="pi pi-plus" class="new mr-2" @click="openNew"/>
            </div>
          </template>
          <template #end>
            <div  v-can="'list models'" class="my-2 flex gap-2">
              <span class="p-input-icon-left">
                <i class="pi pi-search"/>
                <InputText v-model="searchQuery" placeholder="Search..." />
              </span>
              <Button label="Export" icon="pi pi-upload" class="new" @click="exportCSV($event)"/>
            </div>
          </template>
        </Toolbar>
        <Toast/>
        <DataTable
          ref="dt"
          v-model:selection="selectedModels"
          :value="models"
          :loading="loading"
          data-key="id"
          :paginator="false"
          :rows="rowsPerPage"
          :filters="filters"
          responsive-layout="scroll"
          v-can="'list models'"
        >
          <template #header>
            <div class="flex flex-column md:flex-row md:justify-between md:align-items-center">
              <h5 class="m-0 my-auto px-2">Models</h5>
            </div>
          </template>
          <Column selection-mode="multiple" header-style="width: 3rem"></Column>
          <Column field="name_ar" header="Name (ar)" :sortable="true" header-style="width:14%; min-width:10rem;">
            <template #body="slotProps">
              {{ slotProps.data.name_ar }}
            </template>
          </Column>
          <Column field="name_en" header="Name (en)" :sortable="true" header-style="width:14%; min-width:10rem;">
            <template #body="slotProps">
              {{ slotProps.data.name_en }}
            </template>
          </Column>
          <Column field="brand.name_en" header="Brand" :sortable="true" header-style="width:14%; min-width:10rem;">
            <template #body="slotProps">
              {{ slotProps.data.brand?.name_en }}
            </template>
          </Column>

          <Column header-style="min-width:10rem;">
            <template #body="slotProps">
              <Button
               v-can="'edit models'"
                icon="pi pi-pencil"
                class="p-button-rounded p-button-success mr-2"
                @click="confirmEdit(slotProps.data.id)"
              />
              <Button
              v-can="'delete models'"
                icon="pi pi-trash"
                class="delete mt-2"
                @click="delet(slotProps.data.id)"
              />
            </template>
          </Column>
        </DataTable>

        <!-- Custom Pagination -->
        <div class="p-paginator p-component p-unselectable-text mt-3">
          <div class="p-paginator-left-content">
            <span class="p-paginator-current">Showing {{ from }} to {{ to }} of {{ totalRecords }} entries</span>
          </div>
          <div class="p-paginator-right-content">
            <span class="p-paginator-pages">
              <button
                class="p-paginator-first p-paginator-element p-link"
                :disabled="currentPage === 1"
                @click="goToPage(1)"
              >
                <span class="p-paginator-icon pi pi-angle-double-left"></span>
              </button>
              <button
                class="p-paginator-prev p-paginator-element p-link"
                :disabled="!prevPageUrl"
                @click="goToPage(currentPage - 1)"
              >
                <span class="p-paginator-icon pi pi-angle-left"></span>
              </button>

              <template v-for="(link, index) in links" :key="index">
                <button
                  v-if="link.label && !isNaN(link.label)"
                  class="p-paginator-page p-paginator-element p-link"
                  :class="{ 'p-highlight': link.active }"
                  @click="goToPage(parseInt(link.label))"
                >
                  {{ link.label }}
                </button>
                <span v-else-if="link.label === '...'" class="p-paginator-dots">...</span>
              </template>

              <button
                class="p-paginator-next p-paginator-element p-link"
                :disabled="!nextPageUrl"
                @click="goToPage(currentPage + 1)"
              >
                <span class="p-paginator-icon pi pi-angle-right"></span>
              </button>
              <button
                class="p-paginator-last p-paginator-element p-link"
                :disabled="currentPage === totalPages"
                @click="goToPage(totalPages)"
              >
                <span class="p-paginator-icon pi pi-angle-double-right"></span>
              </button>
            </span>

            <span class="p-paginator-rpp-options">
              <Dropdown
                v-model="rowsPerPage"
                :options="[5, 10, 20, 30]"
                @change="changeRowsPerPage"
                class="ml-2"
                style="width: 100px"
              />
            </span>
          </div>
        </div>

        <Dialog v-model:visible="deleteModelsDialog" :style="{ width: '450px' }" header="Confirm" :modal="true">
          <div class="flex align-items-center justify-content-center">
            <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem"/>
            <span v-if="product">Are you sure you want to delete the selected model?</span>
          </div>
          <template #footer>
            <Button label="No" icon="pi pi-times" class="p-button-text" @click="deleteModelsDialog = false"/>
            <Button label="Yes" icon="pi pi-check" class="p-button-text" @click="deleteSelectedModels"/>
          </template>
        </Dialog>
      </va-card>
    </div>
  </div>
</template>

<style scoped lang="scss">
.new {
  background: #3b82f6;
  border: none;
}

.delete {
  background: #ef4444;
  border: none;
}

.p-paginator {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0.5rem;
  background: #ffffff;
  border: 1px solid #dee2e6;
  border-radius: 3px;

  .p-paginator-left-content {
    color: #6c757d;
  }

  .p-paginator-right-content {
    display: flex;
    align-items: center;

    .p-paginator-pages {
      display: flex;
      margin: 0 0.5rem;

      button {
        text-align: center;
        min-width: 2.357rem;
        height: 2.357rem;
        margin: 0.143rem;
        border: 0 none;
        color: #6c757d;
        background: transparent;
        border-radius: 50%;
        transition: background-color 0.2s;

        &:hover {
          background: #e9ecef;
        }

        &.p-highlight {
          color: #ffffff;
          background: #E28C3F;
        }

        &:disabled {
          opacity: 0.5;
          cursor: default;
        }
      }
    }

    .p-paginator-dots {
      min-width: 2.357rem;
      height: 2.357rem;
      margin: 0.143rem;
      display: flex;
      align-items: center;
      justify-content: center;
    }
  }
}
</style>

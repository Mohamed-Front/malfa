<script setup>
import { ref, onMounted } from 'vue'
import axios from "axios";
import { useI18n } from 'vue-i18n';

const { t } = useI18n();
const permissionsData = ref({});
const groupedPermissions = ref([]);
const loading = ref(true);
const activeTab = ref(0);
const visibleDialog = ref(false);
const currentPermission = ref({
  id: '',
  name: '',
  description: '',
  translated_name: ''
});

// Group permissions by their module
const groupPermissions = (data) => {
  const groups = [];

  for (const [module, permissions] of Object.entries(data)) {
    groups.push({
      name: t(`permissions.modules.${module}`),
      value: module,
      permissions: permissions
    });
  }

  // Sort groups alphabetically
  groups.sort((a, b) => a.name.localeCompare(b.name));

  return groups;
};

const fetchData = () => {
  loading.value = true;
  axios.get("api/role/get/permissions").then((res) => {
    permissionsData.value = res.data.data;
    groupedPermissions.value = groupPermissions(res.data.data);
    loading.value = false;
  }).catch(error => {
    console.error("Error fetching permissions:", error);
    loading.value = false;
  });
};

const openEditDialog = (permission) => {
  visibleDialog.value = true;
  currentPermission.value = {
    id: permission.id,
    name: permission.name,
    description: permission.description || '',
    translated_name: permission.translated_name || ''
  };
};

const updateDescription = () => {
  loading.value = true;
  axios.put(`/api/role/permission/${currentPermission.value.id}`, {
    description: currentPermission.value.description,
    translated_name: currentPermission.value.translated_name
  })
  .then(() => {
    fetchData();
    visibleDialog.value = false;
  })
  .catch(error => {
    console.error("Update failed:", error);
  })
  .finally(() => {
    loading.value = false;
  });
};

onMounted(() => {
  fetchData();
});
</script>

<template>
  <div v-can="'list permissions'" class="permissions-container" :class="{'rtl': $i18n.locale === 'ar'}">
    <div class="card">
      <div class="header">
        <h1>{{ t('permissions.title') }}</h1>
        <p class="subtitle">{{ t('permissions.subtitle') }}</p>
      </div>

      <div class="toolbar">
        <Button icon="pi pi-refresh" :label="t('permissions.refresh')"
                @click="fetchData" :loading="loading" />
      </div>

      <TabView v-model:activeIndex="activeTab" class="grouped-tabs">
        <TabPanel v-for="group in groupedPermissions"
                 :key="group.value"
                 :header="group.value">
          <DataTable :value="group.permissions" stripedRows class="p-datatable-sm"
                    :loading="loading" dataKey="id" scrollable scrollHeight="flex">
            <Column :header="t('permissions.columns.id')" field="id" sortable style="width: 80px"></Column>
            <Column :header="t('permissions.columns.name')" field="name" sortable>
              <template #body="{data}">
                <Tag :value="data.translated_name"
                     :severity="data.name.includes('delete') ? 'danger' :
                              data.name.includes('edit') ? 'warning' :
                              data.name.includes('create') ? 'success' : 'info'" />
              </template>
            </Column>

            <Column :header="t('permissions.columns.description')" field="description">
              <template #body="{data}">

                  {{ data.description || t('permissions.no_description') }}


              </template>
            </Column>
            <Column :header="t('permissions.columns.actions')" style="width: 100px">
              <template #body="{data}">
                <Button v-can="'edit permissions'" icon="pi pi-pencil"
                        class="p-detail"
                        @click="openEditDialog(data)" />
              </template>
            </Column>
            <template #empty>
              <div class="empty-message">
                <i class="pi pi-database" style="font-size: 2rem"></i>

              </div>
            </template>
          </DataTable>
        </TabPanel>
      </TabView>

      <!-- Update Description Dialog -->
      <Dialog v-model:visible="visibleDialog"
              :header="t('permissions.dialog.title')"
              :style="{ width: '50vw' }"
              :breakpoints="{ '960px': '75vw', '640px': '90vw' }">
        <div class="p-fluid">
          <div class="p-field">
            <label for="permissionName">{{ t('permissions.dialog.name') }}</label>
            <InputText id="permissionName" v-model="currentPermission.name" disabled />
          </div>
          <div class="p-field">
            <label for="translatedName">{{ t('permissions.dialog.translated_name') }}</label>
            <InputText id="translatedName" v-model="currentPermission.translated_name"
                      :placeholder="t('permissions.dialog.translated_name')" />
          </div>
          <div class="p-field">
            <label for="description">{{ t('permissions.dialog.description') }}</label>
            <Textarea id="description" v-model="currentPermission.description"
                     rows="5" autoResize
                     :placeholder="t('permissions.dialog.description')" />
          </div>
        </div>
        <template #footer>
          <Button :label="t('permissions.dialog.cancel')" icon="pi pi-times"
                  @click="visibleDialog = false"
                  class="p-button-text" />
          <Button :label="t('permissions.dialog.save')" icon="pi pi-check"
                  @click="updateDescription"
                  :loading="loading" autofocus />
        </template>
      </Dialog>
    </div>
  </div>
</template>

<style scoped>
.permissions-container {
  padding: 2rem;
}

.card {
  background: #ffffff;
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.header {
  margin-bottom: 1rem;
}

h1 {
  color: #2c3e50;
  margin-bottom: 0.5rem;
}

.subtitle {
  color: #7f8c8d;
  margin-bottom: 0;
}

.toolbar {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 1rem;
}

.description-cell {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.5rem;
}

.empty-message {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 2rem;
  color: #7f8c8d;
}

:deep(.grouped-tabs .p-tabview-nav) {
  flex-wrap: wrap;
}

:deep(.grouped-tabs .p-tabview-nav-link) {
  padding: 0.75rem 1.25rem;
}

:deep(.grouped-tabs .p-tabview-panels) {
  padding: 1rem 0;
}

:deep(.p-tag) {
  font-size: 0.85rem;
  padding: 0.25rem 0.5rem;
}

.p-field {
  margin-bottom: 1.5rem;
}

.p-field label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 500;
}

/* RTL Support */
.rtl {
  direction: rtl;
  text-align: right;
}

.rtl .toolbar {
  justify-content: flex-start;
}

.rtl .description-cell {
  flex-direction: row-reverse;
}

@media (max-width: 768px) {
  .permissions-container {
    padding: 1rem;
  }

  .card {
    padding: 1rem;
  }

  :deep(.grouped-tabs .p-tabview-nav-link) {
    padding: 0.5rem 0.75rem;
    font-size: 0.85rem;
  }
}
</style>

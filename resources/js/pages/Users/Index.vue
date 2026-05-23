<script setup lang="ts">
import InputError from '@/components/InputError.vue';
import { useInitials } from '@/composables/useInitials';
import AppLayout from '@/layouts/AppLayout.vue';
import { type BreadcrumbItem } from '@/types';
import { Head, router, useForm, usePage } from '@inertiajs/vue3';
import { FilterMatchMode } from '@primevue/core/api';
import Avatar from 'primevue/avatar';
import Button from 'primevue/button';
import Column from 'primevue/column';
import DataTable from 'primevue/datatable';
import Dialog from 'primevue/dialog';
import FileUpload from 'primevue/fileupload';
import IconField from 'primevue/iconfield';
import InputIcon from 'primevue/inputicon';
import InputText from 'primevue/inputtext';
import MultiSelect from 'primevue/multiselect';
import Password from 'primevue/password';
import Select from 'primevue/select';
import Tag from 'primevue/tag';
import ToggleButton from 'primevue/togglebutton';
import Toolbar from 'primevue/toolbar';
import Swal from 'sweetalert2';
import { computed, defineProps, reactive, ref, watch } from 'vue';
const page = usePage();
const filters = ref({
  global: { value: null, matchMode: FilterMatchMode.CONTAINS },
  user: { value: null, matchMode: FilterMatchMode.STARTS_WITH },
  role_ids: { value: null, matchMode: FilterMatchMode.CONTAINS },
});
const breadcrumbs: BreadcrumbItem[] = [
  {
    title: 'Users',
    href: '/users',
  },
];
const props = defineProps<{
  users: any[],
  roles: any[],
  filters: {
    data: any[],
  }
}>();


const flash = page.props?.flash?.message;
if (flash) {
  let timerInterval;
  Swal.fire({
    title: "Process Success",
    icon: "success",
    html: "User Succesfully added",
    timer: 1000,
    timerProgressBar: true,
    didOpen: () => {
      const timer = Swal.getPopup().querySelector("b");
      timerInterval = setInterval(() => {
        timer.textContent = `${Swal.getTimerLeft()}`;
      }, 100);
    },
    willClose: () => {
      clearInterval(timerInterval);
    }
  })
}

function deleteUser(id) {
  Swal.fire({
    title: 'Are you sure?',
    text: "You won't be able to revert this!",
    icon: 'warning',
    showCancelButton: true,
    confirmButtonColor: '#3085d6',
    cancelButtonColor: '#d33',
    confirmButtonText: 'Yes, delete it!'
  }).then((result) => {
    if (result.isConfirmed) {
      router.delete(route('users.destroy', id), {
        onSuccess: () => {
          Swal.fire({
            title: 'Deleted!',
            text: 'This user has been deleted.',
            icon: 'success',
            timer: 1000,
            timerProgressBar: true,
          });
        },
        onError: () => {
          Swal.fire({
            title: 'Failed!',
            text: 'Something went wrong. The role was not deleted.',
            icon: 'error',
          });
        },


      });
    }
  });
}
const { getInitials } = useInitials();

function toggleStatus(id) {
  router.put(route('users.toggleStatus', id), {}, {
    onSuccess: () => {

    },
    onError: (errors) => {
      console.error(errors)
    }
  })
}
const getSeverityStatus = (status) => {
  switch (status) {
    case true:
      return 'succes';
    case false:
      return 'danger';
  }
}
const formattedUsers = computed(() => {
  return props.users.map(user => ({
    ...user,
    role_ids: user.roles.map(role => role.id)
  }))
})
const user = ref({})
const form = useForm({
  name: '',
  username: '',
  email: '',
  password: '',
  password_confirmation: '',
  avatar: '',
  verified_email: false,
  roles: [],
  is_active: true,
})
const selectedUsers = ref()
const usersDialog = ref(false)
const submitted = ref(false)
const openNew = () => {
  user.value = {};
  form.reset();
  usersDialog.value = true;
  submitted.value = false;
}
const openEdit = (usr) => {
  user.value = Object.assign(form, usr)
  form.roles = form?.roles[0]?.id;
  usersDialog.value = true;
  console.log(user.value);

}
const hideDialog = () => {
  usersDialog.value = false;
  submitted.value = false
}
const saveUser = () => {
  submitted.value = true;
  if (form.name?.trim()) {
    if (user.value.id) {
      console.log("masuk edit coy");
      form.patch(route('users.update', user.value.id), {
        onSuccess: (success) => {
          usersDialog.value = false
          user.value = {}
        },
        onError: (errors) => {
          console.log(errors);
        }
      })
    }
    else {
      console.log("masuk new coy");
      form.post(route('users.store'), {
        onSuccess: (success) => {
          usersDialog.value = false
          user.value = {}
        },
        onError: (errors) => {
          console.log(errors);
        }
      });
    }

  }
}

const usernameWarning = ref('');
watch(() => form.username, (val) => {
  if (/\s/.test(val)) {
    usernameWarning.value = 'Username field cannot use spaces'
  } else {
    usernameWarning.value = ''
  }
})
// realtime validate password
const rules = reactive({
  minLength: false,
  hasUppercase: false,
  hasLowercase: false,
  hasNumber: false,
  hasSymbol: false,
  hasSpace: false
})
function validatePassword() {
  const val = form.password

  rules.minLength = val.length >= 8
  rules.hasUppercase = /[A-Z]/.test(val)
  rules.hasLowercase = /[a-z]/.test(val)
  rules.hasNumber = /[0-9]/.test(val)
  rules.hasSymbol = /[^A-Za-z0-9]/.test(val)
  rules.hasSpace = /\s/.test(val)
}
watch(() => form.password, validatePassword)

const inputFile = ref(null);
const openInputFile = () => {
  inputFile.value.click();
}

</script>

<template>

  <Head title="Users" />
  <AppLayout :breadcrumbs="breadcrumbs">
    <div class="card">
      <Toolbar>
        <template #start>
          <Button label="New" icon="pi pi-plus" class="mr-2" @click="openNew" />
          <Button label="Delete" icon="pi pi-trash" severity="danger" variant="outlined" @click="confirmDeleteSelected"
            :disabled="!selectedProducts || !selectedProducts.length" />
        </template>
        <template #end>
          <FileUpload mode="basic" accept="image/*" :maxFileSize="1000000" label="Import" customUpload
            chooseLabel="Import" class="mr-2" auto :chooseButtonProps="{ severity: 'secondary' }" />
          <Button label="Export" icon="pi pi-upload" severity="secondary" @click="exportCSV($event)" />
        </template>
      </Toolbar>
      <DataTable v-model:filters="filters" v-model:selection="selectedUsers" filterDisplay="menu"
        :globalFilterFields="['name', 'username', 'email']" :value="formattedUsers" table-style="min-widht: 50rem"
        paginator :rows="10" :rows-per-page-options="[5, 10, 20, 100]">
        <template #header>
          <div class="flex flex-wrap items-center justify-between gap-2">
            <span class="text-xl font-bold">Users</span>
            <!-- <Button icon="pi pi-refresh" rounded raised></Button> -->
            <IconField>
              <InputIcon>
                <i class="pi pi-search"></i>
              </InputIcon>
              <InputText v-model="filters['global'].value" placeholder="Keyword search" />
            </IconField>
          </div>
        </template>
        <Column selection-mode="multiple"></Column>
        <Column field="username" header="Username" sortable>
          <template #body="{ data }">
            <div class="flex items-center gap-2">
              <img rounded
                :src="data.avatar ? `/storage/` + data.avatar : 'https://ui-avatars.com/api/?name=' + getInitials(data.username) + '&background=random'"
                :alt="data.username" class="rounded-full w-[42px]">
              <span>{{ data.username }}</span>
            </div>
          </template>
        </Column>
        <Column field="name" header="Name" sortable></Column>
        <Column field="email" header="Email" sortable></Column>
        <Column field="role_ids" header="Role" :show-filter-menu="true" :show-filter-match-modes="false">
          <template #body="{ data }">
            <div class="flex">
              <span v-for="role in data.roles" class="bg-green-50  p-2 rounded-2xl">{{ role.name }}</span>
            </div>
          </template>
          <template #filter="{ filterModel, filterCallback }">
            <MultiSelect v-model="filterModel.value" @change="filterCallback()" :showClear="true" :options="roles"
              optionLabel="name" optionValue="id" />
          </template>
        </Column>
        <Column field="is_active" header="status" sortable>
          <template #body="{ data }">
            <Tag :value="data.is_active" :severity="getSeverityStatus(data.is_active)" @click="toggleStatus(data.id)" class="cursor-pointer"></Tag>
          </template>
        </Column>
        <Column field="action" header="Action">
          <template #body="{ data }">
            <div class="flex gap-2">
              <Button icon="pi pi-pencil" rounded severity="info" @click="openEdit(data)"></Button>
              <Button icon="pi pi-trash" rounded severity="danger" @click="deleteUser(data.id)"></Button>
            </div>
          </template>
        </Column>
      </DataTable>
      <!-- Dialog / Modal -->
      <Dialog v-model:visible="usersDialog" header="User Detail" :modal="true" :style="{ width: '650px' }">
        <div class="flex flex-col gap-6">
          <div class="flex flex-col items-center">
            <img src="http://localhost:8000/storage/avatar/373r6AnsdHlA4BbkT0iHEqds840ycjArpI2o7FoJ.jpg" alt=""
              class="w-50 rounded-full shadow-2xl cursor-pointer hover:opacity-95" @click="openInputFile" />
            <input type="file" id="avatar" name="avatar"  ref="inputFile" style="display: none;" accept="image/*">
          </div>
          <div>
            <label for="name">Name</label>
            <InputText id="name" v-model.trim="form.name" required fluid />
            <InputError :message="form.errors.name" />
          </div>
          <div>
            <label for="username">Username</label>
            <InputText id="username" v-model.trim="form.username" required fluid />
            <InputError :message="form.errors.username" />
          </div>
          <div>
            <label for="email">Email</label>
            <InputText type="email" v-model.trim="form.email" required fluid />
            <InputError :message="form.errors.email" />
          </div>
          <div>
            <label for="password">password</label>
            <Password v-model.trim="form.password" required fluid />
            <InputError :message="form.errors.password" />
          </div>
          <div>
            <label for="password_confirmation">Password Confirmation</label>
            <Password v-model.trim="form.password_confirmation" required fluid />
            <InputError :message="form.errors.password_confirmation" />
          </div>
          <div>
            <label for="roles">Roles</label>
            <Select v-model="form.roles" :options="roles" option-value="id" option-label="name" fluid />
            <InputError :message="form.errors.roles" />
          </div>
          <div class="flex justify-between">
            <label for="active">activate the user?</label>
            <ToggleButton v-model="form.is_active" onLabel="active" offLabel="inactive"></ToggleButton>
          </div>
          <!-- <span>{{ selectedUsers }}</span> -->
        </div>
        <template #footer>
          <Button label="Cancel" icon="pi pi-times" text @click="hideDialog" />
          <Button label=" Save" icon="pi pi-check" @click="saveUser" />
          <!-- <span>{{ user }}</span> -->
        </template>
      </Dialog>
    </div>
  </AppLayout>
</template>

<template>
  <div class="bg-gray-200 h-full min-h-screen">
    <h1 class="text-[30px] text-center w-full py-3">Todo List</h1>
    <div class="px-[20%] w-full">
      <form @submit.prevent="createTask" class="pb-5 px-auto w-full flex justify-center">
        <input type="text" :class="{ 'border-red-500': !formData.content.trim() }" class="w-2/3"
               v-model="formData.content">
        <button type="submit" class="bg-[rgb(13,110,253)] text-white w-[100px] h-[41.6px]">
          Thêm mới
        </button>
      </form>

      <div>
        <p class="text-center text-[20px]">List Task</p>

        <div class="flex m-10">
          <draggable
              v-if="filteredData.length > 0"
              class="dragArea list-group w-full"
              :list="filteredData"
              @end="onDragEnd"
          >
            <div
                class="list-group-item m-1 p-3 rounded-md text-center draggable-item flex items-center"
                v-for="element in filteredData"
                :key="element?.content"
            >
              <p class="flex-grow text-center">{{ element?.content }}</p>
              <div class="ml-auto flex space-x-2">
                <button type="button" class="btn btn-primary bg-white border-none" @click="setUpdateTask(element)" data-bs-toggle="modal" data-bs-target="#exampleModal">
                  <img src="../img/image 57.png" alt="">
                </button>
                <button @click="deleteTask(element?.id)"><img src="../img/image 58.png" alt="Delete"></button>
              </div>
            </div>
          </draggable>
          <div v-else>
            No data available.
          </div>
        </div>
      </div>

      <div class="modal fade" id="exampleModal" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true">
        <div class="modal-dialog">
          <div class="modal-content">
            <div class="modal-header">
              <h5 class="modal-title" id="exampleModalLabel">Update Task</h5>
              <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
            </div>
            <div class="modal-body">
              <form @submit.prevent="updateTask">
                <input type="text" class="form-control" v-model="formData.updatecontent">
                <button type="submit" class="btn btn-primary mt-3" data-bs-dismiss="modal">Save changes</button>
              </form>
            </div>
          </div>
        </div>
      </div>

    </div>
  </div>
</template>

<script setup>
import { reactive, ref, computed , watchEffect} from 'vue';
import { VueDraggableNext as draggable } from 'vue-draggable-next';
import axios from '@/plugins/axios';

const loading = ref(false);
const error = ref('');
const data = ref([]);
// const content = ref();
const formData = reactive({
  content: '',
  updatecontent: '',
  id: null,
});
const fetchData = async () => {
  loading.value = true;
  try {
    const response = await axios.get('/All');
    data.value = response.data.map((task, index) => ({ ...task, position: index }));
  } catch (err) {
    error.value = err.message;
  } finally {
    loading.value = false;
  }
};

fetchData();

const createTask = async () => {
  loading.value = true;
  try {
    await axios.post('/create', { content: formData.content });
    await fetchData();
    formData.content = '';
  } catch (err) {
    error.value = err.message;
  } finally {
    loading.value = false;
  }
};

const deleteTask = async (id) => {
  loading.value = true;
  try {
    await axios.delete(`http://localhost:8000/api/v1/delete/${id}`);
    await fetchData();
  } catch (err) {
    error.value = err.message;
  } finally {
    loading.value = false;
  }
};

const setUpdateTask = (element) => {
  formData.updatecontent = element.content;
  formData.id = element.id;
};


const updateTask = async () => {
  loading.value = true;
  try {
    await axios.post('/update', { id: formData.id, updatecontent: formData.updatecontent });
    await fetchData();
  } catch (err) {
    error.value = err.message;
  } finally {
    loading.value = false;
  }
};

const saveContentOrder = async (content) => {
  try {
    await axios.post('/updateLocationTask', { content });
  } catch (err) {
    console.error('Lỗi khi cập nhật thứ tự content:', err);
  }
};


const onDragEnd = (event) => {
  const newIndex = event.newIndex;
  const oldIndex = event.oldIndex;

  const newData = [...data.value];

  const [movedItem] = newData.splice(oldIndex, 1);
  newData.splice(newIndex, 0, movedItem);

  data.value = newData;

  const content = newData.map(task => task.content);

   saveContentOrder(content);

};

const filteredData = computed(() => {
  return data.value.filter(element => element !== null);
});

const isContentEmpty = ref(true);

watchEffect(() => {
  isContentEmpty.value = formData.content.trim() === '';
});

fetchData();
</script>

<style scoped>
.modal-backdrop {
  z-index: 1040 !important;
}
.modal-content {
  margin: 2rem auto;
  z-index: 1100 !important;
}
.modal-dialog {
  max-width: 500px;
  margin: 1.75rem auto;
}

.border-red-500 {
  border-color: #EF4444;
}

</style>

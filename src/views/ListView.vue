<template>
  <div class="container card p-sm-5 p-1 mt-4" style="background: rgb(225, 204, 245);">    
    <div class="row">
      <div class="mb-3">
      <label for="" class="form-label fs-5">Add Note</label>
      <form @submit.prevent="addItem">
        <input
          class="form-control"
          type="text"
          v-model="addNewNote"
          :disabled="isAddActive"
        />
      </form>
    </div>
      <div class="row col-12 col-sm-6">
        <div v-if="!isLoading">
          <div v-if="unChecked.length !== 0">
            <h1>Not Completed</h1>
            <ul
              class="list-group list-group-flush"
              v-for="list in unChecked"
              :key="list.title"
            >
              <li class="list-group-item pt-1 pb-1 pe-1 shadow mt-1 bg-success">
                <div class="row align-items-center">
                  <div class="col-auto">
                    <div class="form-check">
                      <input
                        class="form-check-input"
                        type="checkbox"
                        @click="updateItemStatus(list.id)"
                        :checked="list.isChecked"
                      />
                    </div>
                  </div>
                  <div class="col">
                    <input
                      type="text"
                      class="form-control"
                      style="border: none; width: 100%; font-size: 1em"
                      :value="list.title"
                      @blur="
                        updateTitle(list.id, list.title, $event.target.value)
                      "
                      @keyup.enter="
                        updateTitle(list.id, list.title, $event.target.value)
                      "
                    />
                  </div>
                  <div class="col-auto pe-4">
                    <button
                      @click="deleteItem(list.id)"
                      :disabled="isDeleting"
                      class="btn btn-danger"
                      style="border-radius: 5px"
                    >
                      <i class="fas fa-trash"></i>
                    </button>
                  </div>
                </div>
              </li>
            </ul>
          </div>
          <div v-else>No Note adde yeat</div>
        </div>
        <ScaleLoader :loading="isLoading" class="mb-5" />
      </div>
      <div class="row col-12 col-sm-6">
        <div v-if="!isLoading">
          <div v-if="checked.length !== 0">
            <h1>Completed</h1>
            <ul
              class="list-group list-group-flush"
              v-for="list in checked"
              :key="list.title"
            >
              <li class="list-group-item pt-1 pb-1 pe-1 shadow mt-1 bg-danger">
                <div class="row align-items-center">
                  <div class="col-auto">
                    <div class="form-check">
                      <input
                        class="form-check-input"
                        type="checkbox"
                        @click="updateItemStatus(list.id)"
                        :checked="list.isChecked"
                      />
                    </div>
                  </div>
                  <div class="col">
                    <input
                      type="text"
                      class="form-control"
                      style="border: none; width: 100%; font-size: 1em"
                      :value="list.title"
                      @blur="
                        updateTitle(list.id, list.title, $event.target.value)
                      "
                      @keyup.enter="
                        updateTitle(list.id, list.title, $event.target.value)
                      "
                    />
                  </div>
                  <div class="col-auto pe-4">
                    <button
                      @click="deleteItem(list.id)"
                      :disabled="isDeleting"
                      class="btn btn-danger"
                      style="border-radius: 5px"
                    >
                      <i class="fas fa-trash"></i>
                    </button>
                  </div>
                </div>
              </li>
            </ul>
          </div>
          <div v-else>No Note adde yeat</div>
        </div>
        <ScaleLoader :loading="isLoading" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import { toast } from "vue3-toastify";
import { ScaleLoader } from "vue3-spinner";

const lists = ref([]);
const checked = ref([]);
const unChecked = ref([]);
const isLoading = ref(false);
const isDeleting = ref(false);
const isAddActive = ref(false);
const isStatusUpdating = ref(false);
const addNewNote = ref("");

const getItem = async () => {
  await axios.get("/api/list").then(function (response) {
    lists.value = response.data.datas;
    checked.value = response.data.checked;
    unChecked.value = response.data.unChecked;
  });
};
onMounted(async () => {
  isLoading.value = true;
  await getItem();
  isLoading.value = false;
});
const updateTitle = async (id, title, value) => {
  if (title === value.trim()) return;
  else {
    await axios
      .put("/api/edit/list/" + id, { title: value.trim() })
      .then(function (response) {
        isStatusUpdating.value = false;
        toast.success(response.data.message, {
          autoClose: 1000,
          theme: "dark",
        });
        getItem();
      })
      .catch(function (error) {
        isStatusUpdating.value = false;
        toast.warning(error.response.data.message, {
          autoClose: 1000,
          theme: "dark",
        });
      });
  }
};

const addItem = async () => {
  let val = {
    title: addNewNote.value,
  };
  isAddActive.value = true;
  await axios
    .post("/api/add/list", val)
    .then(function (response) {
      isAddActive.value = false;
      getItem();
      addNewNote.value = "";
      toast.success(response.data.message, {
        autoClose: 1000,
        theme: "dark",
      });
    })
    .catch(function (error) {
      isAddActive.value = false;
      toast.warning(error.response.data.message, {
        autoClose: 1000,
        theme: "dark",
      });
    });
};
const deleteItem = async (id) => {
  isDeleting.value = true;
  await axios
    .delete("/api/delete/list/" + id)
    .then(function (response) {
      isDeleting.value = false;
      toast.success(response.data.message, {
        autoClose: 1000,
        theme: "dark",
      });
      getItem();
    })
    .catch(function (error) {
      isDeleting.value = false;
      toast.warning(error.response.data.message, {
        autoClose: 1000,
        theme: "dark",
      });
    });
};
const updateItemStatus = async (id) => {
  isStatusUpdating.value = true;
  await axios
    .put("/api/status/list/" + id)
    .then(function (response) {
      isStatusUpdating.value = false;
      toast.success(response.data.message, {
        autoClose: 1000,
        theme: "dark",
      });
      getItem();
    })
    .catch(function (error) {
      isStatusUpdating.value = false;
      toast.warning(error.response.data.message, {
        autoClose: 1000,
        theme: "dark",
      });
    });
};
</script>
<style scoped>
</style>
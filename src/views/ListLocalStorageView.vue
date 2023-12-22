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
  
  // get local storage data    
  const getItemFromLocalStorage = () => { // declaration 
                                  // if found , then ok             // if not then make it
    const storedData = JSON.parse(localStorage.getItem("notes")) || { datas: [], checked: [], unChecked: [] };
    lists.value = storedData.datas; // assign the data to list 
    checked.value = storedData.checked; //assign checked data 
    unChecked.value = storedData.unChecked; //assign unChecked data 
  };
  
  // changes will be updated by this funtion
  const saveItemToLocalStorage = () => {
    const dataToStore = { datas: lists.value, checked: checked.value, unChecked: unChecked.value };
    localStorage.setItem("notes", JSON.stringify(dataToStore)); // don't forget it to stringify
  };
  
  onMounted(() => {
    isLoading.value = true;
    getItemFromLocalStorage(); // calling  for local storage data
    isLoading.value = false;
  });
  
  
  const addItem = () => {
    const newItem = {
      id: Date.now(),
      title: addNewNote.value,
      isChecked: false,
    };
    lists.value.push(newItem);//  this list is not nessary but ok...
    unChecked.value.push(newItem);// adding new item as unchecked 
    saveItemToLocalStorage();// and store it to localstorage
    addNewNote.value = ""; // blank the add input field 
    toast.success("Note added successfully", {
      autoClose: 1000,
      theme: "dark",
    });
  };
  const updateTitle = (id, title, value) => {
    if (title === value.trim()) return;
    else {
      // Find the item in the appropriate list
      const listToUpdate = lists.value.find(item => item.id === id);
      if (listToUpdate) {
        listToUpdate.title = value.trim();
        saveItemToLocalStorage();
        toast.success("Title updated successfully", {
          autoClose: 1000,
          theme: "dark",
        });
      }
    }
  };    
  const deleteItem = (id) => {
    const indexToDelete = lists.value.findIndex(item => item.id === id);
    if (indexToDelete !== -1) {
      const deletedItem = lists.value.splice(indexToDelete, 1)[0];
      const listToUpdate = deletedItem.isChecked ? checked.value : unChecked.value;
      const indexToUpdate = listToUpdate.findIndex(item => item.id === id);
      if (indexToUpdate !== -1) {
        listToUpdate.splice(indexToUpdate, 1);
        saveItemToLocalStorage();
        toast.success("Item deleted successfully", {
          autoClose: 1000,
          theme: "dark",
        });
      }
    }
  };
  
  const updateItemStatus = (id) => {
    isStatusUpdating.value = true;
    const itemToUpdate = lists.value.find(item => item.id === id);
    if (itemToUpdate) {
      itemToUpdate.isChecked = !itemToUpdate.isChecked;
      const sourceList = itemToUpdate.isChecked ? unChecked.value : checked.value;
      const destinationList = itemToUpdate.isChecked ? checked.value : unChecked.value;
      const indexToUpdate = sourceList.findIndex(item => item.id === id);
      if (indexToUpdate !== -1) {
        const updatedItem = sourceList.splice(indexToUpdate, 1)[0];
        destinationList.push(updatedItem);
        saveItemToLocalStorage();
        toast.success("Item status updated successfully", {
          autoClose: 1000,
          theme: "dark",
        });
      }
    }
    isStatusUpdating.value = false;
  };
  </script>
  
  <style scoped>
  </style>
  
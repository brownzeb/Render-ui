<template>
  <div class="container" id="blur-body" @click="!addJob">
    <div class="search-option">
      <div class="toggle-buttons">
        <button @click="filter = 'all'" :class="{ active: filter === 'all' }">
          All Jobs
        </button>
        <button
          @click="filter = 'rendered'"
          :class="{ active: filter === 'rendered' }">
          Rendered Jobs
        </button>
        <button
          @click="filter = 'rendering'"
          :class="{ active: filter === 'rendering' }">
          Rendering Jobs
        </button>
        <button
          @click="filter = 'failed'"
          :class="{ active: filter === 'failed' }">
          Failed Jobs
        </button>
      </div>

      <div class="add-tasks">
        <button @click="toggleAddJob">Add New Task</button>
      </div>
    </div>
    <teleport to="body">
      <div v-if="addJob" ref="deleteModalRef">
        <form class="modals" @submit.prevent="handleSubmit">
          <div class="g">
            <div class="inputs">
              <div class="field-froup">
                <label for="name" class="input-label"> Job Type Name</label>

                <input
                  type="text"
                  name="name"
                  class="input-field"
                  v-model="jobName"
                  placeholder=" name" />
              </div>
              <div class="field-froup">
                <label for="file" class="input-label"> File path </label>
                <input
                  type="text"
                  name="file"
                  v-model="jobFilePath"
                  class="input-field"
                  placeholder=" file-path" />
              </div>
            </div>

            <div class="grids">
              <div class="input-modal">
                <h4>Preset</h4>
                <select v-model="selectedPreset">
                  <option
                    v-for="preset in taskStore.preset"
                    :key="preset.name"
                    :value="preset.id"
                    v-if="preset && preset.id">
                    {{ preset.name }}
                  </option>
                </select>
              </div>
              <div class="input-modal">
                <h4>Render Engine</h4>
                <select v-model="taskStore.renderEngines">
                  <option
                    v-for="engine in renderEngines"
                    :key="engine"
                    required>
                    {{ engine }}
                  </option>
                </select>
              </div>

              <div class="input-modal">
                <h4>Render Type</h4>
                <select v-model="presetKeys">
                  <option v-for="jobType in jobTypes" :key="jobType">
                    {{ jobType }}
                  </option>
                </select>
              </div>

              <!-- <div class="input-modal">
                <h4>File Format</h4>
                <select v-model="taskStore.fileFormat">
                  <option v-for="file in fileFormat" :key="file">
                    {{ file }}
                  </option>
                </select>
              </div> -->
              <div class="input-modal">
                <h4>priority</h4>
                <div class="render-input">
                  <input type="number" v-model="priority" />
                </div>
              </div>
            </div>

            <div class="grids">
              <div class="input-modal">
                <h4>Frame Range</h4>
                <div class="input-wrapper">
                  <input
                    type="number"
                    v-model="frameRangeX"
                    placeholder="frame-X" />
                  <input
                    type="number"
                    v-model="frameRangeY"
                    placeholder=" frame-Y" />
                </div>
              </div>

              <div class="input-modal">
                <h4>Render Samples</h4>
                <div class="render-input">
                  <input type="number" v-model="renderSamples" />
                </div>
              </div>

              <div class="input-modal">
                <h4>Resolution (%)</h4>
                <div class="render-input">
                  <input
                    type="number"
                    v-model="resolutionPercentage"
                    placeholder="resolution Percentage" />
                </div>
              </div>

              <div class="input-modal">
                <h4>Resolution</h4>
                <div class="input-wrapper">
                  <input
                    type="number"
                    v-model="resolutionX"
                    placeholder="resolution-X" />
                  <input
                    type="number"
                    v-model="resolutionY"
                    placeholder=" resolution-Y" />
                </div>
              </div>
            </div>
            <div class="buttons">
              <div class="first">
                <button @click="toggleAddJob">Cancel</button>
                <button type="submit">Render</button>
              </div>
            </div>
          </div>
        </form>
      </div>
    </teleport>

    <div class="main-container">
      <div class="task-container" v-if="filter === 'all'">
        <div
          class="task-item"
          v-for="(task, index) in taskStore.tasks"
          :key="index">
          <span class="item-text"
            ><p class="task-item-title">{{ task.job_type_name }}</p></span
          >
          <span class="item-text"
            ><p
              class="task-item-status"
              :class="{
                pending: task.jobStatus === 'pending',
                running: task.jobStatus === 'running',
                completed: task.jobStatus === 'completed',
                failed: task.jobStatus === 'failed',
              }">
              {{ task.jobStatus }}
            </p></span
          >

          <span class="item-text"
            ><button class="item-render" @click="removeTask(task.id)">
              Remove
            </button></span
          >
        </div>
      </div>

      <div class="task-container" v-if="filter === 'rendered'">
        <div
          class="task-item"
          v-for="(task, index) in taskStore.renderedTasks"
          :key="index">
          <span class="item-text">
            <p class="task-item-title">{{ task.job_name }}</p>
          </span>
          <span class="item-text">
            <p class="task-item-status">{{ task.job_status }}</p>
          </span>
          <span class="item-text">
            <button class="item-render">Remove</button>
          </span>
        </div>
      </div>

      <div class="task-container" v-if="filter === 'rendering'">
        <div
          class="task-item"
          v-for="(task, index) in taskStore.renderingTasks"
          :key="index">
          <span class="item-text"
            ><p class="task-item-title">{{ task.job_name }}</p></span
          >
          <span class="item-text"
            ><p class="task-item-status">{{ task.job_status }}</p></span
          >

          <span class="item-text"
            ><button class="item-render">Remove</button></span
          >
        </div>
      </div>

      <div class="task-container" v-if="filter === 'failed'">
        <div
          class="task-item"
          v-for="(task, index) in taskStore.failedTasks"
          :key="index">
          <span class="item-text"
            ><p class="task-item-title">{{ task.job_name }}</p></span
          >
          <span class="item-text"
            ><p class="task-item-status">{{ task.job_status }}</p></span
          >

          <span class="item-text"
            ><button class="item-render">Remove</button></span
          >
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import { onClickOutside } from '@vueuse/core';
import { useTaskStore } from '@/stores/TaskStore';
useTaskStore().fetchTasks();

const filter = ref('all');
const taskStore = useTaskStore();
const addJob = ref(false);
const showModal = ref(false);
const deleteModalRef = ref(null);

const renderEngines = taskStore.renderEngines;
const jobTypes = taskStore.jobTypes;
// const fileFormat = taskStore.fileFormat;
const jobStatus = taskStore.jobStatus;
const renderSamples = ref('');
const jobFilePath = ref('');
const priority = ref('');
const jobName = ref('');
let resolutionX = ref(1920);
let resolutionY = ref(1080);
let frameRangeX = ref('');
let frameRangeY = ref('');
const selectedPreset = ref('');
const resolutionPercentage = ref('');

const presetKeys = computed(() => Object.keys(taskStore.preset));

const toggleAddJob = () => {
  addJob.value = !addJob.value;
  showModal.value = false; // Close the modal when toggling addJob
  toggleModalBlur(addJob.value); // Add blur effect when modal is open
};

onClickOutside(deleteModalRef, toggleAddJob);

const toggleModalBlur = (isBlurred) => {
  document.body.classList.toggle('blur-body', isBlurred);
};

const selectedPresetData = computed(() => {
  return taskStore.preset.find((preset) => preset.id === selectedPreset.value);
});

const handleSubmit = async () => {
  try {
    const newTask = {
      name: jobName.value,
      file_path: jobFilePath.value,
      preset: job_type_name,
      render_engine: taskStore.renderEngines,
      status: taskStore.jobStatus,
      render_type: taskStore.jobTypes,
      // file_format: taskStore.fileFormat,
      priority: taskStore.priority.value,
      render_samples: renderSamples.value,
      resolution_x: resolutionX.value,
      resolution_y: resolutionY.value,
      frame_start: frameRangeX.value,
      frame_end: frameRangeY.value,
      resolution_percentage: resolutionPercentage.value,
    };

    const response = await axios.post(
      'http://127.0.0.1:82/data/jobs',
      newTask,
      {
        headers: {
          'Content-Type': 'application/json',
        },
      }
    );

    console.log('Task submitted successfully:', response.data); // Log success message

    addJob.value = false;
    toggleModalBlur(false); // Remove blur effect when modal is closed
  } catch (error) {
    console.error('Error adding task:', error);
    // Handle potential errors (e.g., display a message to the user)
  }
};

const removeTask = async (taskId) => {
  try {
    // Send a request to your backend API to remove the task
    const response = await fetch(`http://127.0.0.1:5000/deletjob/${taskId}`, {
      method: 'DELETE',
    });

    // Check if the request was successful
    if (!response.ok) {
      throw new Error('Failed to remove task');
    }

    // Update the task list by fetching tasks again
    taskStore.fetchTasks();
  } catch (error) {
    console.error('Error removing task:', error);
  }
};

const updateFields = () => {
  if (selectedPreset.value) {
    const preset = taskStore.preset[selectedPreset.value];
    // Update input field values with preset values
    resolutionX.value = preset.resolutionX;
    resolutionY.value = preset.resolutionY;
    taskStore.renderEngines = preset.renderEngines;
    taskStore.jobTypes = preset.jobTypes;
    taskStore.fileFormat = preset.fileFormat;
    renderSamples.value = preset.renderSamples;
  }
};
onMounted(() => {
  taskStore.fetchPresets(); // Fetch presets when the component is mounted
});

onMounted(() => {
  taskStore.fetchTasks();
});
// Call toggleModalBlur(true) when modal is opened
// Call toggleModalBlur(false) when modal is closed
</script>

<style scoped>
.container {
  height: 50px;
}
.grids {
  margin-top: 50px;
  display: flex;
  justify-content: center;
  gap: 10px;
}
.input-modal {
  border-radius: 10px;
  background: rgb(174, 174, 228);
  width: 320px;
  height: 130px;
}
.input-modal h4 {
  font-family: 'poppins', sans-serif;
  display: absolute;
  left: 20px;
  font-size: 20px;
  justify-content: center;
  align-items: center;
  text-align: center;
}
.render-input input {
  position: relative;
  width: 250px;
  border-radius: 10px;
  height: 40px;
  border: none;
  margin-left: 30px;
}
.input-modal .input-wrapper {
  justify-content: center;
  display: flex;
  gap: 10px;
}
.input-wrapper input {
  position: relative;
  width: 130px;
  border-radius: 10px;
  height: 40px;
  border: none;
}
.input-modal select {
  display: flex;
  color: brown;
  justify-content: center;
  align-items: center;
  border: none;
  outline: none !important;
  padding: 10px 20px;
  border-radius: 5px;
  width: 90%;
  margin-left: 10px;
}
.inputs {
  display: flex;
  gap: 200px;
  justify-content: center;
  align-items: center;
}
.field-group {
  width: 600px;
  position: relative;
  padding-top: 15px;
}
.input-field {
  margin-top: 40px;
  width: 500px;
  outline: 0;
  border: 0;
  border-bottom: 2px solid #999999;
  padding: 0px 0;
  background: transparent;
  font-size: 28px;
  color: black;
  transition: 0.3s ease;
}
.input-label {
  display: block;
  position: relative;
  top: 20px;
  font-size: 12pt;
  font-family: 'poppins', sans-serif;
  color: #999999;
  text-transform: uppercase;
  transition: 0.3s ease;
}
.input-field::placeholder {
  color: transparent;
  font-size: 18pt;
}
.input-field:placeholder-shown ~ .input-label {
  font-size: 15pt;
  top: 35px;
}
.input-field:focus {
  border-image: linear-gradient(to right, #ef5a03, #d1d357);
  border-image-slice: 1;
}
.input-field:focus ~ input-label {
  display: block;
  position: absolute;
  top: 0;
  font-size: 10pt;
  color: #ef5a03;
}
.toggle-buttons button.active {
  background-color: #41435f;
  /* Add any other styles you want for active buttons */
}

.main-container {
  font-family: 'poppins', sans-serif;
  background-color: #292a39;
  position: relative;
  height: 88vh;
  border-radius: 15px;
  overflow-y: auto;
  overflow-x: hidden;
}
.main-container::-webkit-scrollbar {
  width: 15px; /* Adjust width as needed */
}

.main-container::-webkit-scrollbar-track {
  background-color: #292a39; /* Color for the scrollbar track */
}

.main-container::-webkit-scrollbar-thumb {
  background-color: #666;
  border-radius: 10px; /* Color for the scrollbar thumb */
}
.container::-webkit-scrollbar {
  display: none; /* This hides the scrollbar */
}

.search-option {
  display: flex;
  justify-content: space-between;
  margin-right: 10px;
  margin-bottom: 5px;
}

.toggle-buttons {
  display: flex;
  margin-top: 10px;
  margin-bottom: 10px;
  gap: 10px;
}
.toggle-buttons button {
  /*padding: 13px 25px;*/
  width: 210px;
  height: 50px;
  background-color: #292a39;
  border: none;
  outline: none !important;
  color: white;
  border-radius: 10px;
  font-size: 15pt;
}
.search input {
  background-color: #292a39;
  border: none;
  outline: none !important;
  color: white;
  padding: 26px 355px;
  margin-right: 290px;
  border-radius: 10px;
}
.search input::placeholder {
  justify-content: start;
  font-size: 16pt;
  color: #fff;
}
.input-task {
  display: flex;
  gap: 10px;
}
.input-task input {
  background-color: #292a39;
  border: none;
  outline: none !important;
  color: white;
  padding: 26px 170px;
  border-radius: 10px;
}
.input-task input::placeholder {
  justify-content: start;
  font-size: 16pt;
  color: #fff;
}
.input-task button {
  background-color: #3c8b35;
  border: none;
  outline: none !important;
  color: white;
  padding: 26px 40px;
  border-radius: 10px;
}

.add-tasks button {
  margin-top: 10px;
  position: relative;
  background-color: #3c8b35;
  color: #fff;
  width: 210px;
  height: 50px;
  border-radius: 10px;
  font-size: 15pt;
}
.header-wrapper-container {
  /*background-color: blue;*/
  border-radius: 10px;
  margin-top: 10px;
  align-items: center;
  justify-content: space-between;
  height: 7%;
}
.header-wrapper {
  display: flex;
  flex-direction: row;
  align-items: center;
  padding-top: 10px;
  padding-bottom: 10px;
  background-color: #1a1b33;

  border-radius: 10px;
  margin: 10px 0px;
  align-items: center;
  justify-content: space-between;
  cursor: pointer;
}
.texts {
  flex: 1;
  display: flex;
  flex-direction: row;
  align-items: center;
  /* gap: 10px; */
  border-radius: 10px;
  /* background: #a93bc5; */
  cursor: pointer;
  justify-content: space-between;
}
.text {
  color: #fff;
  padding: 20px 20px;
  font-size: 25px;
  font-style: normal;
  font-weight: 600;
  line-height: 20px;
  letter-spacing: -0.36px;
}
.task-container {
  border-radius: 10px;
  margin-top: 20px;
  width: 1885px;
  align-items: center;
  overflow-y: auto;
}
.task-container .task-title {
  position: fixed; /* Make the title sticky */
  top: 159px; /* Stick it at the top */
  background-color: #292a39; /* Background color */
  z-index: 1; /* Ensure it's above other elements */
  padding: 10px 0; /* Add some padding for spacing */
  margin-bottom: 80px;
}
.task-item {
  /* flex: 2;
  padding: auto 5px;
  display: flex;
  flex-direction: row;
  align-items: center;
  /* gap: 10px;
  border-radius: 10px;
  cursor: pointer;
  justify-content: space-between;
  margin-top: 5px;
  margin-right: 25px;
  margin-left: 10px;
   display: flex;
  flex-direction: row;
  align-items: center;
  /* gap: 10px; */
  background: #3b3d75;
  display: flex;
  justify-content: space-between;
  border-radius: 10px;
  margin-bottom: 10px;
  margin-left: 10px;
  align-items: center;
  cursor: pointer;
}
.item-text {
  margin-left: 20px;
  margin-right: 110px;
  color: white;
  width: 60px;
  top: 10px;
  text-align: center;
  font-size: 16pt;
}
.task-item-status {
  color: black;
  text-align: center;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 150px;
  height: 15px;
  padding: 7px;
  font-size: 16px;
  background-color: #fff;
  border-radius: 30px;
  margin-right: 0px;
}
.task-item-priority {
  margin-right: 40px;
}

.task-item-modal button {
  padding: 5px 10px;
  border-radius: 10px;
}

.item-render {
  padding: 10px 50px;
  border-radius: 10px;
  border: none;
  font-size: 16px;
  background-color: rgb(240, 81, 81);
}

.modals {
  border-radius: 10px;
  width: 70%;
  padding: 10px 10px;
  position: absolute;
  top: 180px;
  bottom: 200px;
  left: 350px;
  justify-content: center;
  align-items: center;
}
.modal-input {
  position: relative;
  padding: 0 20px;
  gap: 10px;
  margin-top: 10px;
  margin-right: 10px;
  margin-left: 10px;
  width: 200px;
  height: 150px;
  border-radius: 20px;
  background: rgb(174, 174, 228);
}
.modal-input h4 {
  font-size: 20px;
  text-align: center;
  margin-right: 47px;
}
.modal-input input {
  display: block;
  width: 80%;
  font-size: 16px;
  /*background-color: #292a39;*/
  border: none;
  outline: none !important;
  color: white;
  padding: 15px 20px;
  border-radius: 10px;
}

.modals > div {
  background-color: #2c2e4e;
  border-radius: 10px;
  width: 1360px;
  height: 700px;
}
.grid {
  top: 20;
  display: grid;
  grid-template-columns: 1fr 1fr 1fr 1fr 1fr;
}
.buttons {
  top: 520px;
  left: 400px;
  position: absolute;
  align-items: center;
  display: grid;
  gap: 230px;
  grid-template-columns: 1fr 1fr;
}
.buttons button[type='submit'] {
  background-color: #4caf50;
  color: white;
  font-size: 23px;
}

.buttons button:first-child {
  background-color: #f44336;
  color: white;
  font-size: 23px;
}
.buttons button {
  width: 250px;
  height: 60px;
  border: none;
  outline: none !important;
  border-radius: 10px;
}
/*input,
select {
  display: block;
  background-color: #292a39;
  padding: 15px 50px;
  border-radius: 10px;
  box-sizing: border-box;
  border: none;
  border-radius: 10px;
  font-size: 1em;
} */

.flex {
  display: flex;
  gap: 10px;
}
.first {
  gap: 10px;
  display: flex;
  margin-left: 20px;
}
.task-item-status.pending {
  background-color: #f9d043 !important; /* Light yellow for pending */
}

.task-item-status.rendering {
  background-color: #4caf50 !important; /* Light green for running */
}

.item-text.completed {
  background-color: #673ab7 !important; /* Light purple for completed */
}

.task-item-status.failed {
  background-color: #f44336 !important; /* Light red for failed */
}
</style>

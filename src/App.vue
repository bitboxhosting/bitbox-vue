<template>
  <div
    class="flex flex-col h-full min-h-[100vh] justify-center items-center bg-gradient-to-t from-base-300 to-base-100"
  >
    <div class="w-lg mx-auto space-y-4" style="max-width: 100vw">
      <Card
        v-for="(upload, index) in recentUploads"
        :key="index"
        style="
          background-image: url('/icon-gray-lg.png');
          background-size: cover;
        "
      >
        <p class="text-xl break-all mt-24">{{ upload.filename }}</p>
        <div class="flex space-x-2">
          <button
            class="btn btn-square text-xl"
            @click="
              copy(
                `${server}/uploads/${upload.id}/${encodeURIComponent(
                  upload.filename
                )}`
              )
            "
          >
            <i class="fa-solid fa-clone"></i>
          </button>
          <input :value="upload.path" class="input font-mono w-full" readonly />
        </div>
      </Card>

      <Card>
        <input
          type="file"
          @change="onFileSelected"
          class="file-input max-w-sm"
        />
        <button
          @click="uploadFile"
          class="btn btn-primary"
          v-if="server && !invalidServerErr"
        >
          Upload
        </button>
        <label for="server-err" class="btn btn-primary" v-else>Upload</label>
      </Card>
    </div>
  </div>

  <div class="fixed bottom-4 left-4">
    <label
      for="settings"
      class="btn btn-circle text-xl btn-outline btn-secondary"
      ><i class="fa-solid fa-gear"></i
    ></label>
  </div>
  <div class="fixed bottom-20 left-4">
    <label for="info" class="btn btn-circle text-xl btn-outline btn-accent"
      ><i class="fa-solid fa-bars"></i
    ></label>
  </div>
  <div class="fixed bottom-36 left-4">
    <label for="uploads" class="btn btn-circle text-xl btn-outline btn-primary"
      ><i class="fa-solid fa-file"></i
    ></label>
  </div>

  <Modal id="server-err">
    <h3 class="font-bold text-lg">Server validation error</h3>
    <p class="py-4">
      Either no server has been selected or the server selection is invalid. Try
      selecting one from the server list in
      <span class="text-secondary font-bold"
        >Settings <i class="fa-solid fa-gear"></i
      ></span>
    </p>
    <div class="modal-action">
      <label for="server-err" class="btn">Close</label>
    </div>
  </Modal>

  <Modal id="uploads" v-if="serverInfo">
    <div class="flex justify-between">
      <span class="text-xl font-bold">Uploads</span>
      <label for="uploads" class="btn btn-circle btn-outline btn-sm">
        <i class="fa-solid fa-xmark"></i>
      </label>
    </div>
    <div class="mt-8">
      <a
        v-for="upload in uploads.slice().reverse()"
        :href="upload.path"
        class="link block"
        >{{ upload.filename }}</a
      >
    </div>
  </Modal>

  <Modal id="info" v-if="serverInfo">
    <div class="flex justify-between">
      <span class="text-xl font-bold">Info</span>
      <label for="info" class="btn btn-circle btn-outline btn-sm">
        <i class="fa-solid fa-xmark"></i>
      </label>
    </div>
    <div class="mt-8">
      <div class="mb-8">
        <h2>
          <b>{{ serverInfo.server_name }}</b> run by
          <b>{{ serverInfo.owner_name }}</b>
        </h2>
      </div>
      <h2 class="font-semibold text-lg">Message from server</h2>
      <p>{{ serverInfo.message }}</p>
      <h2 class="font-semibold text-lg mt-8">Server rules</h2>
      <ul v-if="serverInfo.rules" class="list-disc">
        <li v-for="rule in serverInfo.rules">{{ rule }}</li>
      </ul>
    </div>
    <div v-if="serverInfo.donate" class="mt-8">
      <a
        :href="serverInfo.donate"
        class="btn btn-primary btn-circle btn-outline text-xl"
        ><i class="fa-solid fa-money-bill-wave"></i
      ></a>
      Donate to this server
    </div>
  </Modal>

  <Modal id="settings">
    <div class="flex justify-between">
      <span class="text-xl font-bold">Settings</span>
      <label for="settings" class="btn btn-circle btn-outline btn-sm">
        <i class="fa-solid fa-xmark"></i>
      </label>
    </div>
    <div class="grid grid-cols-3 mt-8">
      <span class="font-bold block my-auto">Server</span>
      <input v-model="server" class="input bg-base-200 w-full col-span-2" />
      <div class="alert alert-error col-span-3 mt-4" v-if="invalidServerErr">
        The selected server is invalid
      </div>

      <span class="text-xl font-bold mt-4 col-span-3">Server List</span>

      <div class="card bg-base-200 col-span-3 mt-4 p-4 space-y-3">
        <div class="flex justify-between">
          <span class="font-semibold text-sm">Server URL</span>
          <span class="font-semibold text-sm">Country</span>
        </div>
        <div class="flex justify-between" v-for="listedServer in serverList">
          <button @click="server = listedServer.url">
            {{ listedServer.url }}
          </button>
          <span>{{ countryToFlag(listedServer.country) }}</span>
        </div>
      </div>
    </div>
  </Modal>
</template>

<script setup>
import { ref, watch, onMounted, computed } from 'vue'
import Card from './components/Card.vue'
import Modal from './components/Modal.vue'
import axios from 'axios'

const serverInfo = ref({})
const serverList = ref([])
const selectedFile = ref(null)
const server = ref('')
const invalidServerErr = ref(false)

const uploads = ref([])
const recentUploads = computed(() => uploads.value.slice(-3))

onMounted(() => {
  getServerInfo()
  getServerList()
  getUploads()
})

if (!server.value && localStorage.server) {
  server.value = localStorage.server
}

watch(server, async () => {
  localStorage.server = server.value
  getServerInfo()
})

watch(
  uploads,
  async () => {
    localStorage.uploads = JSON.stringify(uploads.value)
  },
  { deep: true }
)

const getServerInfo = () => {
  axios
    .get(`${server.value}/info`)
    .then((response) => {
      serverInfo.value = response.data
      invalidServerErr.value = false
    })
    .catch((error) => {
      console.error(error)
      invalidServerErr.value = true
      serverInfo.value = null
    })
}

const onFileSelected = (event) => {
  selectedFile.value = event.target.files[0]
}

const uploadFile = () => {
  const endpointUrl = `${server.value}/upload`
  const formData = new FormData()
  formData.append('files', selectedFile.value)

  axios
    .post(endpointUrl, formData, {
      headers: {
        'Content-Type': 'multipart/form-data'
      }
    })
    .then((response) => {
      console.log(response.data), saveFileProperties(response.data)
    })
    .catch((error) => {
      console.error(error)
    })
}

const saveFileProperties = (response) => {
  let content = {
    path: `${server.value}${response.file}`,
    id: response.id,
    filename: response.file.replace(/^.*\//, '')
  }

  uploads.value.push(content)
}

const copy = (str) => {
  navigator.clipboard.writeText(str)
}

const getServerList = () => {
  axios
    .get(
      'https://raw.githubusercontent.com/bitboxhosting/extras/master/server-list.json'
    )
    .then((response) => {
      response.data.forEach((item, index) => {
        axios
          .get(`${item}/info`)
          .then((response) => {
            let list = response.data
            list.url = item
            serverList.value.push(list)
          })
          .catch((error) => {
            console.error(error)
          })
      })
    })
    .catch((error) => {
      console.error(error)
    })
}

const getUploads = () => {
  if (localStorage.uploads) {
    uploads.value = JSON.parse(localStorage.uploads)
  }
}

const countryToFlag = (code) => {
  const codePoints = code
    .toUpperCase()
    .split('')
    .map((char) => 127397 + char.charCodeAt())

  return String.fromCodePoint(...codePoints)
}
</script>

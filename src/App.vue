<template>
  <div
    class="flex flex-col h-full min-h-[100vh] justify-center items-center bg-gradient-to-t from-base-300 to-base-100"
  >
    <div class="w-lg mx-auto space-y-4" style="max-width: 100vw">
      <Card
        v-for="(upload, index) in uploads"
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

  <Modal id="server-err">
    <h3 class="font-bold text-lg">Server validation error</h3>
    <p class="py-4">
      Either no server has been selected or the server selection is invalid
    </p>
    <div class="modal-action">
      <label for="server-err" class="btn">Close</label>
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
      <div class="alert alert-info col-span-3 mt-4" v-if="invalidServerErr">
        <div>
          You can try visiting it in your browser to accept the certificate, if
          the selected server uses self-signed certs
        </div>
      </div>
    </div>
  </Modal>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue'
import Card from './components/Card.vue'
import Modal from './components/Modal.vue'
import axios from 'axios'

const serverInfo = ref({})

const selectedFile = ref(null)

const server = ref('')

const invalidServerErr = ref(false)

onMounted(() => {
  getServerInfo()
})

if (!server.value && localStorage.server) {
  server.value = localStorage.server
}

watch(server, async () => {
  localStorage.server = server.value
  getServerInfo()
})

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

const uploads = ref([])

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
    path: response.file,
    id: response.id,
    filename: response.file.replace(/^.*\//, '')
  }

  if (uploads.value.length > 2) {
    uploads.value.splice(0, 1)
  }

  uploads.value.push(content)
}

const copy = (str) => {
  navigator.clipboard.writeText(str)
}
</script>

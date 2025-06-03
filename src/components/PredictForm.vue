<script setup>
import { ref } from 'vue'

const form = ref({
  sks: null,
  ipk: null,
  angkatan: '',
  jenis_kelamin: '',
  pekerjaan_ayah: '',
  pekerjaan_ibu: '',
})

const result = ref('')
const error = ref('')
const loading = ref(false)

async function submitForm() {
  error.value = ''
  result.value = ''
  loading.value = true

  try {
    const response = await fetch('https://fast-api-production-b8ec.up.railway.app/predict', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        NIM: form.value.angkatan,
        JenisKelamin: form.value.jenis_kelamin,
        SKS: form.value.sks,
        IPK: form.value.ipk,
        PekerjaanAyah: form.value.pekerjaan_ayah,
        PekerjaanIbu: form.value.pekerjaan_ibu,
      }),
    })

    if (!response.ok) {
      throw new Error('Server error: ' + response.statusText)
    }

    const data = await response.json()

    result.value = data.prediction || 'Tidak ada hasil prediksi.'
  } catch (err) {
    error.value = 'Terjadi kesalahan saat memproses: ' + err.message
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <div class="max-w-3xl mx-auto bg-white p-8 rounded-lg shadow mt-8">
    <h1 class="text-4xl font-bold mb-6 text-center">DECISION TREE C4.5</h1>

    <form @submit.prevent="submitForm" class="space-y-6">
      <div>
        <label for="sks" class="block font-semibold mb-1">SKS</label>
        <input
          id="sks"
          type="number"
          v-model.number="form.sks"
          min="0"
          required
          class="w-full border rounded p-2"
        />
      </div>

      <div>
        <label for="ipk" class="block font-semibold mb-1">IPK</label>
        <input
          id="ipk"
          type="number"
          step="0.01"
          v-model.number="form.ipk"
          min="0"
          max="4"
          required
          class="w-full border rounded p-2"
        />
      </div>

      <div>
        <label for="angkatan" class="block font-semibold mb-1">Angkatan</label>
        <select id="angkatan" v-model="form.angkatan" required class="w-full border rounded p-2">
          <option value="" disabled>Pilih Angkatan</option>
          <option value="14">2014</option>
          <option value="15">2015</option>
          <option value="16">2016</option>
          <option value="17">2017</option>
          <option value="18">2018</option>
        </select>
      </div>

      <div>
        <label for="jenis_kelamin" class="block font-semibold mb-1">Jenis Kelamin</label>
        <select
          id="jenis_kelamin"
          v-model="form.jenis_kelamin"
          required
          class="w-full border rounded p-2"
        >
          <option value="" disabled>Pilih Jenis Kelamin</option>
          <option value="L">Laki-laki</option>
          <option value="P">Perempuan</option>
        </select>
      </div>

      <div>
        <label for="pekerjaan_ayah" class="block font-semibold mb-1">Pekerjaan Ayah</label>
        <select
          id="pekerjaan_ayah"
          v-model="form.pekerjaan_ayah"
          required
          class="w-full border rounded p-2"
        >
          <option value="" disabled>Pilih Pekerjaan Ayah</option>
          <option value="PNS">PNS</option>
          <option value="Swasta">Swasta</option>
          <option value="Nelayan">Nelayan</option>
          <option value="Wiraswasta">Wiraswasta</option>
          <option value="TNI/POLRI">TNI/Polri</option>
          <option value="Tani">Petani</option>
          <option value="Lain-lain">Lainnya</option>
        </select>
      </div>

      <div>
        <label for="pekerjaan_ibu" class="block font-semibold mb-1">Pekerjaan Ibu</label>
        <select
          id="pekerjaan_ibu"
          v-model="form.pekerjaan_ibu"
          required
          class="w-full border rounded p-2"
        >
          <option value="" disabled>Pilih Pekerjaan Ibu</option>
          <option value="PNS">PNS</option>
          <option value="Swasta">Swasta</option>
          <option value="Nelayan">Nelayan</option>
          <option value="Wiraswasta">Wiraswasta</option>
          <option value="TNI/POLRI">TNI/Polri</option>
          <option value="Tani">Petani</option>
          <option value="Ibu Rumah Tangga">Ibu Rumah Tangga</option>
          <option value="Lain-lain">Lainnya</option>
        </select>
      </div>

      <button
        type="submit"
        class="w-full bg-blue-600 text-white p-3 rounded hover:bg-blue-700 transition duration-300"
        :disabled="loading"
      >
        Periksa
      </button>
    </form>

    <div v-if="loading" class="text-center mt-6 font-semibold">Memproses...</div>
    <div v-if="error" class="text-center mt-6 text-red-600 font-semibold">{{ error }}</div>
    <div v-if="result" class="mt-8 p-6 bg-gray-200 rounded-lg text-center text-3xl font-bold">
      {{ result }}
    </div>
  </div>
</template>

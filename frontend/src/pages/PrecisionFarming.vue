<template>
  <div class="bg-background min-h-screen">
    <div class="max-w-3xl mx-auto py-12 px-4 sm:px-6 lg:px-8">
      <div class="text-center">
        <h1 class="text-4xl font-bold text-menubg">Precision Farming</h1>
        <p class="text-lg text-gray-600 mt-4">
          Upload an image for disease and pest diagnosis
        </p>
      </div>

      <div v-if="!imagePreview" class="flex justify-center items-center mt-8">
        <div
          class="w-full md:w-2/3 lg:w-1/2 border-2 border-dashed border-gray-400 rounded-lg p-8 cursor-pointer"
          @dragover.prevent
          @drop.prevent="handleDrop"
          @click="openFileInput"
        >
          <input
            type="file"
            @change="handleFileUpload"
            class="hidden"
            ref="fileInput"
          />
          <div class="w-full h-full flex flex-col justify-center items-center">
            <i
              class="fas fa-upload text-gray-500 text-4xl mb-4"
              aria-hidden="true"
            ></i>
            <p class="text-lg text-gray-600 mb-4">
              Drag and drop or click to upload an image
            </p>
            <button
              class="bg-secondary hover:bg-tertiary text-textLighter py-2 px-4 rounded-md"
            >
              Upload
            </button>
          </div>
        </div>
      </div>
      <div v-if="imagePreview" class="mt-8">
        <img :src="imagePreview" alt="Image Preview" width="500" height="500" class="mx-auto w-full md:w-2/3 lg:w-1/2 h-[350px] object-cover rounded-lg" />
        <div class="flex justify-center items-center mt-4 space-x-7">
          
          <button @click="processImage" class="mt-4 bg-green-500 hover:bg-green-600 text-white py-2 px-4 rounded-md flex items-center">
            <span v-if="loading" class="loader"></span> <!-- Spinner Loader -->
            <span v-else>
              <i class="fas fa-robot"></i> Scan with AI
            </span>
          </button>
          
                    <button @click="removeImage" class="mt-4 border-2 border-red-500 text-red-500 hover:bg-red-600 hover:text-white py-2 px-4 rounded-md">
                      Remove Image
                    </button>
      </div>
      </div>


      <div v-if="results" class="mt-12 bg-white rounded-lg shadow-lg p-6">
        <h2 class="text-2xl font-bold text-menubg mb-4">Diagnosis Results</h2>
        <p class="text-text font-medium">{{ results }}</p>
        <p class="text-text mt-4 font-semibold">Remedy Suggestions:</p>
        <ul class="list-disc pl-6 text-text space-y-2">
          <li class="flex items-center">
            <i class="fas fa-check-circle text-green-500 mr-2"></i> Remedy 1
          </li>
          <li class="flex items-center">
            <i class="fas fa-check-circle text-green-500 mr-2"></i> Remedy 2
          </li>
          <li class="flex items-center">
            <i class="fas fa-check-circle text-green-500 mr-2"></i> Remedy 3
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      results: null,
      remedySuggestions: [],
      imagePreview: null,
      loading: false,
      file: null,
    };
  },
  methods: {
    openFileInput() {
      this.$refs.fileInput.click();
    },
    async handleFileUpload(event) {
      const file = event.target.files[0];
      this.file = file;
      this.imagePreview = URL.createObjectURL(file); // Set the image preview
      // await this.processImage(file);
    },
    async handleDrop(event) {
      const file = event.dataTransfer.files[0];
      this.imagePreview = URL.createObjectURL(file); // Set the image preview
      this.file = file;
      // await this.processImage(file);
    },
    removeImage() {
      this.imagePreview = null; // Clear the image preview
      this.results = null; // Optionally clear results
      this.remedySuggestions = []; // Optionally clear remedy suggestions
      this.file = null;
    },
    async processImage() {
      this.loading = true;
      try {
        // Convert the image to a base64-encoded string
        const imageData = await this.convertImageToBase64(this.file);
        
        // Send the image data to the server for prediction
        const formData = new FormData();
        formData.append('file', this.file); // Append the file to the FormData object

      // Send the image data to the server for prediction
        const response = await axios.post('http://127.0.0.1:5000/predict', formData, {
        headers: {
          'Content-Type': 'multipart/form-data', // Set the content type to multipart/form-data
        },
      });

      this.results = response.data; // Assuming the response is the predicted label

        // Fetch remedy suggestions based on the prediction
        this.remedySuggestions = await this.fetchRemedySuggestions(this.results);
      } catch (error) {
        console.error("response error:", error);
        this.results = 'An error occurred during the diagnosis.';
        this.remedySuggestions = [];
      } finally {
        this.loading = false;
      }
    },
    async convertImageToBase64(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onloadend = () => {
          const base64data = reader.result;
          resolve(base64data.split(',')[1]); // Return only the base64 part
        };
        reader.onerror = reject;
        reader.readAsDataURL(file);
      });
    },
    async fetchRemedySuggestions(prediction) {
      // Implement your logic to fetch remedy suggestions based on the prediction
      return ['Remedy 1', 'Remedy 2', 'Remedy 3'];
    },
  },
};
</script>

<style scoped>
.fas {
  @apply text-gray-500;
}

.loader {
  border: 2px solid transparent;
  border-top: 2px solid white;
  border-radius: 50%;
  width: 16px;
  height: 16px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

</style>
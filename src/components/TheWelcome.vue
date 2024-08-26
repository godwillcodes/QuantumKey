<template>
  <div class="h-screen flex items-center justify-center bg-gray-900">
    <div
      class="bg-gray-800 bg-opacity-90 backdrop-blur-md border border-gray-600 rounded-lg p-10 max-w-3xl w-full shadow-lg"
    >
      <h1 class="text-3xl font-extrabold text-center text-white mb-8">
        QuantumKey Password Generator
      </h1>

      <form @submit.prevent="generatePassword" class="space-y-8">
        <!-- Length Slider -->
        <div>
          <label for="length" class="block text-lg font-bold text-gray-300 mb-2"
            >Password Length: {{ length }}</label
          >
          <input
            type="range"
            id="length"
            v-model="length"
            min="8"
            max="64"
            step="1"
            aria-describedby="length-helper"
            class="w-full h-2 bg-gray-700 border border-gray-600 rounded-md cursor-pointer"
          />
          <p id="length-helper" class="mt-1 text-gray-400 text-sm">
            Specify the length of the password (between 8 and 64 characters).
          </p>
        </div>

        <!-- Character Types -->
        <div>
          <label class="block text-lg font-bold text-gray-300 mb-3">Character Types:</label>
          <div class="grid grid-cols-2 gap-4">
            <label class="flex items-center text-gray-400">
              <input
                type="checkbox"
                v-model="includeUppercase"
                class="mr-2"
                aria-label="Include uppercase letters"
              />
              <span class="text-sm font-light">Uppercase</span>
            </label>
            <label class="flex items-center text-gray-400">
              <input
                type="checkbox"
                v-model="includeLowercase"
                class="mr-2"
                aria-label="Include lowercase letters"
              />
              <span class="text-sm font-light">Lowercase</span>
            </label>
            <label class="flex items-center text-gray-400">
              <input
                type="checkbox"
                v-model="includeNumbers"
                class="mr-2"
                aria-label="Include numbers"
              />
              <span class="text-sm font-light">Numbers</span>
            </label>
            <label class="flex items-center text-gray-400">
              <input
                type="checkbox"
                v-model="includeSpecialChars"
                class="mr-2"
                aria-label="Include special characters"
              />
              <span class="text-sm font-light">Special Characters</span>
            </label>
          </div>
          <p v-if="errorMessage" class="mt-2 text-red-500 text-sm">{{ errorMessage }}</p>
        </div>

        <!-- Generated Password -->
        <div>
          <label class="block text-lg font-bold text-gray-300 mb-2">Generated Password:</label>
          <input
            type="text"
            :value="generatedPassword"
            readonly
            aria-describedby="password-helper"
            class="w-full px-4 py-3 border border-gray-600 rounded-md shadow-sm bg-gray-700 text-white focus:outline-none"
          />
          <p id="password-helper" class="mt-1 text-gray-400 text-sm">
            The generated password will appear here.
          </p>
        </div>

        <!-- Buttons -->
        <div class="flex items-center justify-between space-x-2">
          <button
            type="button"
            @click="copyToClipboard"
            class="bg-blue-600 text-white py-3 px-5 rounded-md hover:bg-blue-700 transition-transform transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-blue-500"
            aria-label="Copy password to clipboard"
          >
            Copy
          </button>
          <button
            type="button"
            @click="generatePassword"
            class="bg-green-600 text-white py-3 px-5 rounded-md hover:bg-green-700 transition-transform transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-green-500"
            aria-label="Regenerate password"
          >
            Regenerate
          </button>
        </div>
      </form>

      <!-- Strength Indicator -->
      <div v-if="generatedPassword" class="mt-6 text-center">
        <div :class="strengthClass" class="text-lg font-semibold">{{ strengthText }}</div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref } from 'vue'
import axios from 'axios'

// Reactive state
const length = ref(12)
const includeUppercase = ref(true)
const includeLowercase = ref(true)
const includeNumbers = ref(true)
const includeSpecialChars = ref(true)
const generatedPassword = ref('')
const strengthText = ref('')
const strengthClass = ref('')
const errorMessage = ref('')

// Password generation logic
const generatePassword = () => {
  const charSets: { [key: string]: string } = {
    uppercase: 'ABCDEFGHIJKLMNOPQRSTUVWXYZ',
    lowercase: 'abcdefghijklmnopqrstuvwxyz',
    numbers: '0123456789',
    special: '!@#$%^&*()_+-=[]{}|;:,.<>?/'
  }

  let charset = ''
  if (includeUppercase.value) charset += charSets.uppercase
  if (includeLowercase.value) charset += charSets.lowercase
  if (includeNumbers.value) charset += charSets.numbers
  if (includeSpecialChars.value) charset += charSets.special

  if (charset.length === 0) {
    errorMessage.value = 'Please select at least one character type for the password.'
    return
  } else {
    errorMessage.value = ''
  }

  generatedPassword.value = Array.from({ length: length.value }, () =>
    charset.charAt(Math.floor(Math.random() * charset.length))
  ).join('')

  updateStrength()
}

// Update strength indicator
const updateStrength = async () => {
  const password = generatedPassword.value

  let strength = 'Weak'
  let className = 'text-red-500'

  if (password.length >= 12) {
    strength = 'Medium'
    className = 'text-yellow-500'
  }

  if (
    password.length >= 16 &&
    /[A-Z]/.test(password) &&
    /[a-z]/.test(password) &&
    /\d/.test(password) &&
    /[!@#$%^&*()_+-=[]{}|;:,.<>?]/.test(password)
  ) {
    strength = 'Strong'
    className = 'text-green-500'
  }

  // Call external API for real-world strength assessment
  try {
    const response = await axios.post('https://api.example.com/password-strength', { password })
    const data = response.data
    strengthText.value = `Strength: ${strength} - ${data.message || ''}`
    className = data.class || className
  } catch (error) {
    console.error('Failed to fetch password strength from external API:', error)
    strengthText.value = `Strength: ${strength}`
  }

  strengthClass.value = className
}

// Copy password to clipboard
const copyToClipboard = () => {
  navigator.clipboard
    .writeText(generatedPassword.value)
    .then(() => alert('Password copied to clipboard!'))
    .catch((err) => console.error('Failed to copy password: ', err))
}
</script>

<style scoped>
/* Ensure backdrop-filter is supported by the browser */
@supports (backdrop-filter: blur(10px)) {
  .backdrop-blur-md {
    backdrop-filter: blur(8px);
  }
}

/* Style the range input */
input[type='range'] {
  -webkit-appearance: none;
  appearance: none;
}

input[type='range']::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: #4f46e5;
  cursor: pointer;
}

input[type='range']::-webkit-slider-runnable-track {
  width: 100%;
  height: 4px;
  background: #6b7280;
  border-radius: 2px;
}

input[type='range']::-moz-range-thumb {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: #4f46e5;
  cursor: pointer;
}

input[type='range']::-moz-range-track {
  width: 100%;
  height: 4px;
  background: #6b7280;
  border-radius: 2px;
}

input[type='range']::-ms-thumb {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: #4f46e5;
  cursor: pointer;
}

input[type='range']::-ms-track {
  width: 100%;
  height: 4px;
  background: transparent;
  border-color: transparent;
  color: transparent;
}
</style>

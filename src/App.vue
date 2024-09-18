<script setup>
import chat from "./assets/chat.svg";
import { ref } from "vue";
import axios from "axios";
import bot from "../src/assets/bot.png";
import wave from "./assets/wave.vue";
import darkblueWave from "./assets/darkbluewave.png";
import lightblueWave from "./assets/lightbluewave.png";
import greenWave from "./assets/greenwave.png";
import yellowWave from "./assets/yellowwave.png";
import defaultWave from "./assets/wave.png";
import blueWave from "./assets/bluewave.png";

function handleChat() {
  isChatOpen.value = !isChatOpen.value;
}

const isChatOpen = ref(false);
const newMessage = ref("");
const messages = ref([]);
const selectedFile = ref(null); // For file upload

// The function to send an AI message using OpenAI API
// const getAIResponse = async (userMessage) => {
//   try {
//     const response = await axios.post(
//       "https://api.openai.com/v1/chat/completions",
//       {
//         model: "gpt-4o-mini", // Choose the model
//         prompt: userMessage, // The user's message as the prompt
//         max_tokens: 150, // Maximum number of tokens in the response
//         temperature: 0.7, // Creativity level of the response
//       },
//       {
//         headers: {
//           Authorization: `Bearer ${import.meta.env.VITE_OPENAI_API_KEY}`,
//           'Content-Type': 'application/json', // Replace with your actual OpenAI API key
//         },
//       }
//     );

//     return response.data.choices[0].text.trim(); // The generated AI response

//   } catch (error) {
//     console.error("Error with OpenAI API:", error);
//     return "I'm sorry, I couldn't process your request.";
//   }
// };
const getAIResponse = async (userMessage) => {
  try {
    // Using JSONPlaceholder API to simulate interaction
    const response = await axios.post(
      "https://jsonplaceholder.typicode.com/posts",
      {
        title: userMessage, // Using userMessage as the title
        body: "This is a placeholder response", // Placeholder text
        userId: 1, // Dummy user ID
      }
    );

    // Extracting the response from JSONPlaceholder
    return `Received response: ${response.data.title} - ${response.data.body}`;
  } catch (error) {
    console.error("Error with API:", error);
    return "I'm sorry, I couldn't process your request.";
  }
};

const sendMessage = async () => {
  if (newMessage.value.trim() === "" && !selectedFile.value) return; // Don't proceed if no message or file

  // Prepare the formData if file is selected
  const formData = new FormData();
  let fileUploadResponse = null;

  if (selectedFile.value) {
    formData.append("file", selectedFile.value);
    formData.append("message", newMessage.value); // Append message with file if both exist
  }

  // Add the user's message to the chat
  messages.value.push({
    id: Date.now(),
    text: newMessage.value,
    isAI: false,
  });

  try {
    // POST file and message to the backend if a file is selected
    if (selectedFile.value) {
      fileUploadResponse = await axios.post("https://file.io", formData, {
        headers: {
          "Content-Type": "multipart/form-data",
        },
      });

      console.log("File uploaded:", fileUploadResponse.data);
    }

    // If only a message is sent or if both file and message are sent, get AI response
    const aiResponse = await getAIResponse(newMessage.value);
    messages.value.push({
      id: Date.now(),
      text: aiResponse,
      isAI: true,
    });

    // Clear the input and reset file after sending
    selectedFile.value = null;
    newMessage.value = "";

  } catch (error) {
    console.error("Error posting data:", error);
    messages.value.push({
      id: Date.now(),
      text: "Error processing your request.",
      isAI: true,
    });
  }
};
function getWaveStyle(waveNumber) {
  // if (waveNumber === 4) {
  //   return {
  //     backgroundImage: `url(${blueWave})`,
  //   };
  // }
  // if (waveNumber === 2) {
  //   return {
  //     backgroundImage: `url(${lightblueWave})`,
  //   };
  // }
  // if (waveNumber === 3) {
  //   return {
  //     backgroundImage: `url(${greenWave})`,
  //   };
  // }
  return {
    backgroundImage: `url(${lightblueWave})`,
  };
}


// Handle file input change
const handleFileUpload = (event) => {
  const file = event.target.files[0];
  if (file) {
    selectedFile.value = file; // Store the selected file
    newMessage.value = file.name; // Show the file name in the input box
  }
};
</script>

<template>
  <div class="relative h-screen bg-red-200">
    <!-- Main Content -->

    <div
      v-if="isChatOpen"
      class="fixed bottom-28 right-12 overflow-x-hidden overflow-y-hidden bg-dark-blue h-600px rounded-xl w-96 text-white"
    >
      <section>
        <div class="wave" id="wave1" :style="getWaveStyle(1)" style="--i: 1"></div>
        <div class="wave" id="wave2" :style="getWaveStyle(2)" style="--i: 2"></div>
        <div class="wave" id="wave3" :style="getWaveStyle(3)" style="--i: 3"></div>
        <div class="wave" id="wave4" :style="getWaveStyle(4)" style="--i: 4"></div>
      </section>
      <div class="mt-10 flex items-center gap-2 px-6">
        <div><img :src="bot" class="h-10 w-auto" /></div>
        <div>Bop</div>
      </div>
      <div class="px-6 py-4 h-96 overflow-y-auto scrollable" style="height: calc(100% - 150px)">
        <div v-for="message in messages" :key="message.id" class="mb-4 flex">
          <!-- User Messages -->
          <div v-if="!message.isAI" class="message-bubble user">
            <p class="text-left text-sm">{{ message.text }}</p>
            <div v-if="message.fileName" class="text-xs mt-1">
              File: {{ message.fileName }}
            </div>
          </div>

          <!-- AI Messages -->
          <div v-if="message.isAI" class="message-bubble ai">
            <p class="text-left text-sm">{{ message.text }}</p>
          </div>
        </div>
      </div>

      <!-- Input Section -->
      <div class="absolute bottom-0 left-0 w-full p-4 bg-gray-800 rounded-b-xl flex items-center">
    <!-- File Upload -->
    <div class="relative">
      <label class="flex items-center justify-center w-10 h-10 bg-gray-700 text-white rounded-l cursor-pointer hover:bg-gray-600">
        <!-- File Upload Icon (FontAwesome or any other icon library) -->
        <svg
          xmlns="http://www.w3.org/2000/svg"
          fill="none"
          viewBox="0 0 24 24"
          stroke="currentColor"
          class="w-6 h-6"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M3 16l4-4m0 0l4 4m-4-4v12m0 0H7m4 0h10M9 8h3m0 0h3m0 0h3"
          />
        </svg>
        <input
          type="file"
          @change="handleFileUpload"
          class="absolute inset-0 w-full h-full opacity-0 cursor-pointer"
        />
      </label>
    </div>

    <!-- Message Input -->
    <form @submit.prevent="sendMessage" class="w-full">
      <input
        v-model="newMessage"
        type="text"
        placeholder="Type your message or select a file..."
        class="w-full p-2 rounded-r bg-gray-700 text-white outline-none"
      />
    </form>
  </div>
    </div>

    <button
      @click="handleChat"
      class="fixed bottom-8 right-12 rounded-full p-0 bg-transparent border-none"
    >
      <img :src="chat" alt="chat" class="w-16 h-auto bg-transparent" />
    </button>
  </div>
</template>

<style scoped>
section {
  position: relative;
  width: 100%;

  height: 8px;
  transform: scaleY(-1); /* Invert the entire section (waves and animation) */
}

.wave {
  position: absolute;
  bottom: -43px;
  left: 0;
  width: 100%;
  height: 80px;
  background: url(../src/assets/wave.png);
  background-size: 1000px 100px;
  animation: animate 2s ease-in-out infinite;
  animation-delay: calc(var(--i) * 0.25s);
}

.wave#wave1 {
  z-index: 1000;
  opacity: 1;
  background-position-x: 300px;
 
}

.wave#wave2 {
  z-index: 999;
  opacity: 0.5;
  /* opacity:1; */
  background-position-x: 200px;
  
}

.wave#wave3 {
  z-index: 998;
  opacity: 0.2;
  /* opacity:1; */
  background-position-x: 100px;
}

.wave#wave4 {
  z-index: 997;
  opacity: 0.1;
  /* opacity:1; */
  background-position-x: 0px;
}

@keyframes animate {
  0%,
  100% {
    transform: translateY(0px);
  }
  50% {
    transform: translateY(-10px); /* Invert the animation movement */
  }
}

.mb-4 {
  display: flex;
  flex-direction: column; /* Ensure vertical stacking of elements */
  width: 100%;
}

/* Message bubbles */
.message-bubble {
  max-width: 60%; /* Adjust based on your design */
  padding: 8px 12px;
  border-radius: 12px; /* Rounded edges */
  overflow-wrap: break-word; /* Break long words */
  word-wrap: break-word; /* Ensure text wraps */
  white-space: pre-wrap; /* Preserve whitespace */
  margin-bottom: 8px;
  display: inline-block;
}

/* User messages alignment */
.message-bubble.user {
  background-color: #4a4a4a; /* User message color */
  align-self: flex-start; /* Align to the left */
}

/* AI messages alignment */
.message-bubble.ai {
  background-color: #007bff; /* AI message color */
  align-self: flex-end; /* Align to the right */
}

/* Ensure the message text is not overflowing */
.message-bubble p {
  margin: 0;
  overflow-wrap: break-word; /* Break long words */
  word-wrap: break-word; /* Ensure text wraps */
}
::-webkit-scrollbar {
  width: 12px; /* Width of the scrollbar */
}

::-webkit-scrollbar-thumb {
  background: #888; /* Color of the scrollbar thumb */
  border-radius: 6px; /* Round the corners */
}

::-webkit-scrollbar-thumb:hover {
  background: #555; /* Color of the scrollbar thumb when hovered */
}

::-webkit-scrollbar-track {
  background: #f1f1f1; /* Background of the track */
  border-radius: 6px; /* Round the corners */
}

/* Custom scrollbar for Firefox */
.scrollbar-container {
  scrollbar-width: thin; /* Thin scrollbar */
  scrollbar-color: #888 #f1f1f1; /* Thumb and track colors */
}

/* Add this class to your scrollable container */
.scrollable {
  max-height: 500px; /* Adjust as needed */
  overflow-y: auto; /* Enable vertical scrolling */
}
</style>

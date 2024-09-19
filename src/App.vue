<script setup>

import { ref, watch, nextTick } from "vue";
import axios from "axios";
import MessageIcon from "@mui/icons-material/Message";
import bot from "../src/assets/bot.png";
import wave from "./assets/wave.vue";
import darkblueWave from "./assets/darkbluewave.png";
import lightblueWave from "./assets/lightbluewave.png";
import greenWave from "./assets/greenwave.png";
import yellowWave from "./assets/yellowwave.png";
import defaultWave from "./assets/wave.png";
import blueWave from "./assets/bluewave.png";
import { useMessagesStore } from "./store/messages"; 

const { messages, addMessage, updateMessage } = useMessagesStore();

function handleChat() {
  isChatOpen.value = !isChatOpen.value;
}

const isChatOpen = ref(false);
const newMessage = ref("");

const selectedFile = ref(null); // For file upload
const isLoading = ref(false);
const chatContainer = ref(null);
const lastMessage = ref(null);
const scrollToBottom = () => {
  // Check if the lastMessage exists in the DOM before scrolling
  if (lastMessage.value && lastMessage.value.scrollIntoView) {
    lastMessage.value.scrollIntoView({ behavior: "smooth" });
  }
};

// Watch for changes in messages and scroll when they update
watch(messages, async () => {
  // Wait for the DOM to update before scrolling
  await nextTick();
  scrollToBottom();
});

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
  if (newMessage.value.trim() === "" && !selectedFile.value) return;

  const formData = new FormData();
  let fileUploadResponse = null;

  if (selectedFile.value) {
    formData.append("file", selectedFile.value);
    formData.append("message", newMessage.value);
  }

  addMessage({
    id: Date.now(),
    text: newMessage.value,
    isAI: false,
  });

  try {
    if (selectedFile.value) {
      fileUploadResponse = await axios.post("https://file.io", formData, {
        headers: {
          "Content-Type": "multipart/form-data",
        },
      });

      console.log("File uploaded:", fileUploadResponse.data);
    }

    const loaderMessageId = Date.now() + 1;
    addMessage({
      id: loaderMessageId,
      text: "",
      isAI: true,
      isLoading: true,
    });

    const aiResponse = await getAIResponse(newMessage.value);

    updateMessage(loaderMessageId, {
      text: aiResponse,
      isLoading: false,
    });
  } catch (error) {
    console.error("Error posting data:", error);
    updateMessage(loaderMessageId, {
      text: "Error generating response.",
      isLoading: false,
    });
  } finally {
    newMessage.value = "";
    selectedFile.value = null;
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
  <div class="relative h-screen">
   
    <transition name="fade-slide">
      <div
        v-if="isChatOpen"
        class="fixed mobile-size md:bottom-28 md:right-12 inset-off md:w-96 laptop-height md:rounded-xl bg-dark-blue text-white md:overflow-hidden"
      >
        <section>
          <div
            class="wave"
            id="wave1"
            :style="getWaveStyle(1)"
            style="--i: 1"
          ></div>
          <div
            class="wave"
            id="wave2"
            :style="getWaveStyle(2)"
            style="--i: 2"
          ></div>
          <div
            class="wave"
            id="wave3"
            :style="getWaveStyle(3)"
            style="--i: 3"
          ></div>
          <div
            class="wave"
            id="wave4"
            :style="getWaveStyle(4)"
            style="--i: 4"
          ></div>
        </section>
        <div class="mt-10 flex justify-between items-center gap-2 px-6">
          <div class="flex gap-2">
            <img :src="bot" class="h-10 w-auto" />
            <div class="self-center">Bop</div>
          </div>

          <button
            @click="handleChat"
            class="rounded-full p-0 bg-transparent border-none"
          >
            <div
              class="xl:p-1 w-6 h-6 md:w-12 md:h-12 xl:w-14 xl:h-14 rounded-full flex items-center justify-center bg-blue-500"
            >
              <svg
                v-if="!isChatOpen"
                class="w-4 h-4 md:w-7 md:h-7 xl:w-8 xl:h-8 text-white"
                fill="none"
                stroke="currentColor"
                viewBox="0 0 24 24"
                xmlns="http://www.w3.org/2000/svg"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M4 6h16M4 12h16m-7 6h7"
                />
              </svg>
              <svg
                v-else
                class="w-4 h-4 md:w-7 md:h-7 xl:w-8 xl:h-8 text-white"
                fill="none"
                stroke="currentColor"
                viewBox="0 0 24 24"
                xmlns="http://www.w3.org/2000/svg"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M6 18L18 6M6 6l12 12"
                />
              </svg>
            </div>
          </button>
        </div>
        <div
          class="px-6 py-4 h-96 overflow-y-auto scrollable"
          ref="chatContainer"
          style="height: calc(100% - 150px)"
        >
          <div v-for="message in messages" :key="message.id" class="mb-4 flex">
          
            <div v-if="!message.isAI" class="message-bubble user">
              <p class="text-left text-sm">{{ message.text }}</p>
              <div v-if="message.fileName" class="text-xs mt-1">
                File: {{ message.fileName }}
              </div>
            </div>

         
            <div v-if="message.isAI" class="message-bubble ai">
              <div v-if="message.isLoading" class="loader"></div>
              <p v-else class="text-left text-sm">{{ message.text }}</p>
            </div>
          </div>
          <div ref="lastMessage"></div>
        </div>

        <!-- Input Section -->
        <div
          class="absolute bottom-0 left-0 w-full p-4 bg-gray-800 rounded-b-xl flex items-center"
        >
          <!-- File Upload -->
          <div class="relative">
            <label
              class="flex items-center justify-center gap-2 w-10 h-10 bg-gray-700 text-white rounded-l cursor-pointer hover:bg-gray-600"
            >
              
              <i class="fas fa-upload text-lg text-blue-500"></i>

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
    </transition>
    <button
      @click="handleChat"
      :class="`${
        isChatOpen ? 'hidden' : ''
      } md:block fixed bottom-6 right-6 md:bottom-8 md:right-12 rounded-full p-0 bg-transparent border-none`"
    >
      <div
        class="xl:p-1 w-9 h-9 md:w-12 md:h-12 xl:w-14 xl:h-14 rounded-full flex items-center justify-center bg-blue-500"
      >
        <svg
          v-if="!isChatOpen"
          class="w-6 h-6 md:w-7 md:h-7 xl:w-8 xl:h-8 text-white"
          fill="none"
          stroke="currentColor"
          viewBox="0 0 24 24"
          xmlns="http://www.w3.org/2000/svg"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M4 6h16M4 12h16m-7 6h7"
          />
        </svg>
        <svg
          v-else
          class="w-6 h-6 md:w-7 md:h-7 xl:w-8 xl:h-8 text-white"
          fill="none"
          stroke="currentColor"
          viewBox="0 0 24 24"
          xmlns="http://www.w3.org/2000/svg"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M6 18L18 6M6 6l12 12"
          />
        </svg>
      </div>
    </button>
  </div>
</template>

<style scoped>

.mobile-size {
  height: 100vh;
  width: 100%;
  inset: 0;
}
@media (max-width: 768px) {
  .message-bubble {
    max-width: 80%; 
  }
  .wave {
    height: 80px; 
  }
}

@media (min-width: 768px) {
  .laptop-height {
    height: 75vh;
  }
  .inset-off {
    left: auto;
    top: auto;
  }

  .md\:bottom-28 {
    bottom: 7rem; /* Original bottom distance for laptops */
  }
  .md\:right-12 {
    right: 3rem; /* Position from the right for laptops */
  }

  .md\:w-96 {
    width: 24rem; /* Width for laptops */
  }
  .md\:rounded-xl {
    border-radius: 0.75rem; /* Rounded corners for desktops */
  }
}
.fade-slide-enter-active,
.fade-slide-leave-active {
  transition: opacity 0.5s ease, transform 0.5s ease;
}

.fade-slide-enter, 
.fade-slide-leave-to /* .fade-slide-leave-active in <2.1.8 */ {
  opacity: 0;
  transform: translateY(20px);
}
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
  flex-direction: column; 
  width: 100%;
}


.message-bubble {
  max-width: 60%; 
  padding: 8px 12px;
  border-radius: 12px; 
  overflow-wrap: break-word; 
  word-wrap: break-word; 
  white-space: pre-wrap; 
  margin-bottom: 8px;
  display: inline-block;
}


.message-bubble.user {
  background-color: #4a4a4a; 
  align-self: flex-start; 
}


.message-bubble.ai {
  background-color: #007bff; 
  align-self: flex-end; 
}


.message-bubble p {
  margin: 0;
  overflow-wrap: break-word;
  word-wrap: break-word; 
}
::-webkit-scrollbar {
  width: 12px; 
}

::-webkit-scrollbar-thumb {
  background: #374151; 
  border-radius: 6px; 
}

::-webkit-scrollbar-thumb:hover {
  background: #555; 
}

::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 6px; 
}


.scrollbar-container {
  scrollbar-width: thin; 
  scrollbar-color: #888 #f1f1f1; 
}


.scrollable {
  max-height: 500px;
  overflow-y: auto; 
}


.loader {
  width: 5px;
  aspect-ratio: 1;
  border-radius: 50%;
  animation: l5 1s infinite linear alternate;
}

@keyframes l5 {
  0% {
    box-shadow: 8px 0 #000, -8px 0 #0002;
    background: #000;
  }
  33% {
    box-shadow: 8px 0 #000, -8px 0 #0002;
    background: #0002;
  }
  66% {
    box-shadow: 8px 0 #0002, -8px 0 #000;
    background: #0002;
  }
  100% {
    box-shadow: 8px 0 #0002, -8px 0 #000;
    background: #000;
  }
}
</style>

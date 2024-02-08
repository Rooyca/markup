<template>
  <nav >
    <div class="title">MarkUP</div>
    <div class="options">
      <label>
        <input type="checkbox" v-model="showMarkdown" />
        Markdown
      </label>
      <button @click="input = ''">Clear</button>
      <button @click="openModal()">Share</button>
      <a href="https://github.com/rooyca/markup" target="_blank" class="heart">❤️</a>
    </div>
  </nav>

  <div class="editor">
    <textarea v-if="showMarkdown" class="input" v-model="input" @input="update"></textarea>
    <div class="output">
      <div v-html="output"></div>
    </div>
  </div>


  <div class="modal-overlay" v-if="showModal">
    <div class="modal-content">
      <div class="modal-header">
        <h2 >Share this!</h2>
        <p>Copy the url below to share your markdown.</p>
      </div>

      <div class="modal-body">
        <input type="text" readonly :value="urlShort" />
        <button @click="copy">Copy</button>
        <button @click="showModal = false" style="background-color:#424242;">Close</button>
      </div>

    </div>
  </div>

</template>

<script setup>
import { marked } from 'marked';
import { debounce } from 'lodash-es';
import { ref, computed, onMounted } from 'vue';
import {useToast} from 'vue-toast-notification';
import 'vue-toast-notification/dist/theme-sugar.css';

const $toast = useToast();
const input = ref('# MarkUP');
const showModal = ref(false);
const showMarkdown = ref(!window.location.href.includes('showmarkdown=false'));
const urlShort = ref('Loading...');

const openModal = () => {
  createShortUrl();
  showModal.value = true;
};

const url = computed(() => {
  const url = new URL(window.location.href);
  url.searchParams.set('showmarkdown', false);
  return url;
});

const output = computed(() => marked(input.value));

const update = debounce((e) => {
  input.value = e.target.value;
  // update the url with the new text
  const url = new URL(window.location.href);
  // base64 encoding
  const textBase64 = btoa(e.target.value);
  url.searchParams.set('text', textBase64);
  window.history.replaceState({}, '', url);
}, 100);

const copy = () => {
  navigator.clipboard.writeText(urlShort.value).then(() => {
    $toast.success('Copied to clipboard!', {'position': 'top', 'duration': '1200'})
  });
  showModal.value = false;
};

const createShortUrl = async () => {
    fetch("https://cool-river-1072.fly.dev/new", {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        url: 'https://markup-phi.vercel.app/?showmarkdown=false&text=' + btoa(input.value)
      })
    })
    .then(response => response.json())
    .then(data => {
      console.log(data);
      urlShort.value = data;
    })
    .catch((error) => {
      console.error('Error:', error);
    });

};

onMounted(() => {
  const url = new URL(window.location.href);
  const text = url.searchParams.get('text');

  if (text) {
    // decode base64
    input.value = atob(text);
  }
});
</script>

<style>
nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 20px;
  height: 60px;
  background-color: #242424;
  border-bottom: 1px solid #f6f6f6;
}
.options button, .heart{
  margin-left: 10px;
}

button, .heart {
  background-color: #242424;
  color: #f6f6f6;
  border: 1px solid #f6f6f6;
  padding: 5px 10px;
  border-radius: 5px;
  cursor: pointer;
}

.heart {
  font-size: 14px;
  text-decoration: none;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  backdrop-filter: blur(2px);
}

.modal-content {
  background-color: rgb(46, 46, 46);
  color: rgba(255, 255, 255, 0.87);
  padding: 20px;
  width: 400px;
  border-radius: 7px;
}

.modal-body input {
  width: 100%;
}

.modal-body button {
  margin-top: 20px;
  margin-right: 10px;
}
body {
  margin: 0;
  color-scheme: light dark;
  color: rgba(255, 255, 255, 0.87);
  background-color: #242424;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  -webkit-text-size-adjust: 100%;
}

.editor {
  height: 100vh;
  display: flex;
}

.input,
.output {
  overflow: auto;
  width: 100%;
  height: 100%;
  box-sizing: border-box;
  padding: 0 20px;
}

.input {
  border: none;
  border-right: 1px solid #f6f6f6;
  resize: none;
  outline: none;
  font-size: 14px;
  font-family: 'Monaco', courier, monospace;
  padding: 20px;
}

.title {
  font-size: 20px;
  font-weight: bold;
  color: #f6f6f6;
  font-family: 'Monaco', courier, monospace;
  cursor: pointer;
}
code {
  display: inline-block;
  padding: 0.2em 0.4em;
  margin: 0;
  font-size: 85%;
  background-color: #f8f8f8;
  border-radius: 3px;
  color: #333;
  font-family: "Courier New", Courier, monospace;
}

@media (max-width: 400px) {
  .editor {
    flex-direction: column;
  }
  .input {
    border-right: none;
    border-bottom: 1px solid #f6f6f6;
  }
  nav {
    flex-direction: column;
    height: auto;
    padding-bottom: 20px;
  }
  .title {
    margin-bottom: 10px;
    margin-top: 10px;
  }
}
</style>

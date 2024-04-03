<script setup lang="ts">
import { ref } from 'vue'

// state
const token = ref('')
const loading = ref(false)
const markedRead = ref(0)
const isCompleted = ref(false)
const urlString = window.location.href
const fragmentIndex = urlString.indexOf('#')
const fragment = urlString.substring(fragmentIndex + 1)
const params = new URLSearchParams(fragment)
const accessToken = params.get('access_token')
// get user id by request to google api
if (accessToken) token.value = `${accessToken}`

// authorize google api oauth2
function handler() {
  const url = `https://accounts.google.com/o/oauth2/v2/auth?scope=https://www.googleapis.com/auth/gmail.modify&include_granted_scopes=true&response_type=token&state=getToken&redirect_uri=${window.location.origin}/&client_id=${'149475774472-b0il9chm8dms4qmq0cv9gh6oi8e2v07u.apps.googleusercontent.com'}`
  window.location.href = url
}

function loopMarkRead(token: string, pageToken = '') {
  fetch(
    `https://www.googleapis.com/gmail/v1/users/me/messages?maxResults=500&pageToken=${pageToken}&q=is:unread`,
    {
      method: 'GET',
      headers: {
        Authorization: `Bearer ${token}`
      }
    }
  )
    .then((res) => res.json())
    .then((data) => {
      if (data.messages) {
        fetch('https://gmail.googleapis.com/gmail/v1/users/me/messages/batchModify', {
          method: 'POST',
          headers: {
            Authorization: `Bearer ${token}`,
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({
            ids: data.messages.map((message: any) => message.id),
            addLabelIds: [],
            removeLabelIds: ['UNREAD']
          })
        }).then(() => {
          markedRead.value += data.messages.length
        })
      }

      if (data.nextPageToken) {
        loopMarkRead(token, data.nextPageToken)
      } else {
        loading.value = false
        isCompleted.value = true
        alert('All messages marked as read')
      }
    })
}

function markread() {
  loading.value = true
  loopMarkRead(`${token.value}`, '')
}
</script>

<template>
  <div v-show="!loading" class="flex-center">
    <button class="button-85" @click="handler" v-show="!token">Authorize</button>
    <button class="button-85" @click="handler" v-show="isCompleted">
      Authorize another account
    </button>
    <br />
    <button class="button-85" @click="markread" v-show="token">Mark as read</button>
    <br />
  </div>
  <div v-show="loading" class="flex-center">
    <div>Email Mark Read: {{ markedRead }}</div>
    <br />
    <div class="loader"></div>
  </div>
</template>
<style scoped>
/* CSS */
.flex-center {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.button-85 {
  padding: 0.6em 2em;
  border: none;
  outline: none;
  color: rgb(255, 255, 255);
  background: #111;
  cursor: pointer;
  position: relative;
  z-index: 0;
  border-radius: 10px;
  user-select: none;
  -webkit-user-select: none;
  touch-action: manipulation;
}

.button-85:before {
  content: '';
  background: linear-gradient(
    45deg,
    #ff0000,
    #ff7300,
    #fffb00,
    #48ff00,
    #00ffd5,
    #002bff,
    #7a00ff,
    #ff00c8,
    #ff0000
  );
  position: absolute;
  top: -2px;
  left: -2px;
  background-size: 400%;
  z-index: -1;
  filter: blur(5px);
  -webkit-filter: blur(5px);
  width: calc(100% + 4px);
  height: calc(100% + 4px);
  animation: glowing-button-85 20s linear infinite;
  transition: opacity 0.3s ease-in-out;
  border-radius: 10px;
}

@keyframes glowing-button-85 {
  0% {
    background-position: 0 0;
  }
  50% {
    background-position: 400% 0;
  }
  100% {
    background-position: 0 0;
  }
}

.button-85:after {
  z-index: -1;
  content: '';
  position: absolute;
  width: 100%;
  height: 100%;
  background: #222;
  left: 0;
  top: 0;
  border-radius: 10px;
}

.loader {
  --c1: #673b14;
  --c2: #f8b13b;
  width: 40px;
  height: 80px;
  border-top: 4px solid var(--c1);
  border-bottom: 4px solid var(--c1);
  background: linear-gradient(90deg, var(--c1) 2px, var(--c2) 0 5px, var(--c1) 0) 50%/7px 8px
    no-repeat;
  display: grid;
  overflow: hidden;
  animation: l5-0 2s infinite linear;
}
.loader::before,
.loader::after {
  content: '';
  grid-area: 1/1;
  width: 75%;
  height: calc(50% - 4px);
  margin: 0 auto;
  border: 2px solid var(--c1);
  border-top: 0;
  box-sizing: content-box;
  border-radius: 0 0 40% 40%;
  -webkit-mask:
    linear-gradient(#000 0 0) bottom/4px 2px no-repeat,
    linear-gradient(#000 0 0);
  -webkit-mask-composite: destination-out;
  mask-composite: exclude;
  background:
    linear-gradient(var(--d, 0deg), var(--c2) 50%, #0000 0) bottom / 100% 205%,
    linear-gradient(var(--c2) 0 0) center/0 100%;
  background-repeat: no-repeat;
  animation: inherit;
  animation-name: l5-1;
}
.loader::after {
  transform-origin: 50% calc(100% + 2px);
  transform: scaleY(-1);
  --s: 3px;
  --d: 180deg;
}
@keyframes l5-0 {
  80% {
    transform: rotate(0);
  }
  100% {
    transform: rotate(0.5turn);
  }
}
@keyframes l5-1 {
  10%,
  70% {
    background-size:
      100% 205%,
      var(--s, 0) 100%;
  }
  70%,
  100% {
    background-position: top, center;
  }
}
</style>

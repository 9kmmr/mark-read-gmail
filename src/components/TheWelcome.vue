<script setup lang="ts">
import { ref } from 'vue'

// state
const token = ref('')
const urlString = window.location.href
const fragmentIndex = urlString.indexOf('#')
const fragment = urlString.substring(fragmentIndex + 1)
const params = new URLSearchParams(fragment)
const accessToken = params.get('access_token')
// get user id by request to google api
if (accessToken) token.value = `${accessToken}`

// authorize google api oauth2
function handler() {
  const url = `https://accounts.google.com/o/oauth2/v2/auth?scope=https://www.googleapis.com/auth/gmail.modify&include_granted_scopes=true&response_type=token&state=getToken&redirect_uri=${window.location.href}&client_id=${'149475774472-b0il9chm8dms4qmq0cv9gh6oi8e2v07u.apps.googleusercontent.com'}`
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
      console.log(data)

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
      })

      if (data.nextPageToken) {
        loopMarkRead(token, data.nextPageToken)
      } else {
        alert('All messages marked as read')
      }
    })
}

function markread() {
  loopMarkRead(`${token.value}`, '')
}
</script>

<template>
  <button class="button-85" @click="handler" v-show="!token">Authorize</button>
  <br />
  <button class="button-85" @click="markread" v-show="token">Mark as read</button>
</template>
<style scoped>
/* CSS */
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
</style>

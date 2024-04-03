<script setup lang="ts">
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
        .then((res) => res.json())
        .then((response) => {
          console.log('response: ', response)
        })
      if (data.nextPageToken) {
        loopMarkRead(token, data.nextPageToken)
      } else {
        alert('All messages marked as read')
      }
    })
}

function markread() {
  const urlString = window.location.href
  const fragmentIndex = urlString.indexOf('#')
  const fragment = urlString.substring(fragmentIndex + 1)
  const params = new URLSearchParams(fragment)
  const accessToken = params.get('access_token')
  // get user id by request to google api
  console.log('accessToken', accessToken)
  loopMarkRead(`${accessToken}`, '')
}
</script>

<template>
  <button @click="handler">Authorized</button>
  <br />
  <button @click="markread">Mark as read</button>
</template>

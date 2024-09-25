<template>
  <div>
    <h1>RFID Tag Reader</h1>
    <p>Listening for RFID tag reads...</p>
    <p>RFID Tag: {{ rfidRead }}</p>

  </div>
</template>
<script setup>
import Echo from 'laravel-echo';
import { onMounted, ref } from 'vue';
import Axios from 'axios';

const rfidRead = ref("");

const axios = Axios.create({
  baseURL: 'http://localhost:8000',
  headers: {
    'Content-Type': 'application/json',
    'Accept': 'application/json',
  },
});

onMounted(() => {
  const echo = new Echo({
    broadcaster: 'reverb',
        key: 'mhp4yxekltahowumdb9h',
        wsHost: "localhost",
        wsPort: 8080,
        forceTLS: false,
        encrypted: true,
        disableStats: true,
        authorizer: (channel, options) => {
            return {
                authorize: (socketId, callback) => {
                    axios.post('broadcasting/auth', {
                        socket_id: socketId,
                        channel_name: channel.name
                    })
                    .then(response => {
                        callback(false, response.data);
                    })
                    .catch(error => {
                        callback(true, error);
                    });
                }
            };
        },
  });

  echo.channel('rfid-tag-read')
    .listen('RFIDTagReadEvent', (e) => {
      rfidRead.value = e.tagId;
    });
})
 </script>
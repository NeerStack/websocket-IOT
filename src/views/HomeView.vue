<template>
  <main>
    <!-- <header>
      <h1>Home</h1>
    </header> -->
    <div class="form">
      <h2>Subscribe</h2>
      <form @submit.prevent>
        <label for="topic">Topic: </label>
        <input type="text" v-model="topic" id="topic" />
        <label for="qos">QoS: </label>
        <input type="number" v-model="qos" id="qos" />
        <button class="submit" type="submit" @click="doSubscribe()" >Subscribe</button>
      </form>
    </div>
    <div class="form">
      <h2>Publish</h2>
      <form  @submit.prevent>
        <label for="message">Message: </label>
        <input type="text" v-model="message" id="message" />
        <button class="submit" type="submit" @click="sendMesage()" >Send</button>
      </form>
    </div>
    <div>
      <h2>Response</h2>
      <textarea name="response" v-model="response" cols="30" rows="10">
      </textarea>
    </div>
  </main>
</template>

<script setup>
import { ref } from 'vue'
import Swal from 'sweetalert2'
import mqtt from "mqtt";
import { host, username, password } from '../utils/index.js'
// import WebSocket from 'ws'

const toast = Swal.mixin({
  toast: true,
  position: 'top-end',
  showConfirmButton: false,
  timer: 3000,
  timerProgressBar: true,
  didOpen: (toast) => {
    toast.addEventListener('mouseenter', Swal.stopTimer)
    toast.addEventListener('mouseleave', Swal.resumeTimer)
  }
})

const response = ref()
const message = ref()

const clientId = 'mqttjs_' + Math.random().toString(16).substr(2, 8)

const client = mqtt.connect(host, {
  clientId: clientId,
  username: username,
  password: password,
})

client.on('connect', () => {
  toast.fire({
    icon: 'success',
    title: 'MQTT connection established!'
  })
})
const topic = ref("topic/mqttx")
const qos = ref(0)
const subscription = ref({
  topic: topic.value,
  qos: qos.value,
});

const doSubscribe = () => {
  const { topic, qos } = subscription.value;
  client.subscribe(
    topic,
    { qos },
    (error, granted) => {
      if (error) {
        toast.fire({
          icon: 'error',
          title: 'MQTT subscription error!'
        })
        return;
      }
      toast.fire({
        icon: 'success',
        title: 'MQTT subscription established!'
      })
      console.log("Subscribed", granted);
    }
  );
};



const sendMesage = () =>{
  // console.log(message.value)
  if(message.value === ''){
    toast.fire({
      icon: 'error',
      title: 'Please enter a message!'
    })
    return
  }
  const publish = ref({
  topic: topic.value,
  payload: `{"received_text": "${message.value}" }`,
  qos: qos.value,
  retain: false,
});

  client.publish(publish.value.topic, publish.value.payload, {qos: publish.value.qos}, (error) => {
    if (error) {
      toast.fire({
        icon: 'error',
        title: 'MQTT publish error!'
      })
      return;
    }
    toast.fire({
      icon: 'success',
      title: 'MQTT publish success!'
    })
  });
}

client.on('message', (topic, message) => {
  // message is Buffer
  console.log(message.toString())
  response.value = `Topic: ${topic},  Response: ${message.toString()}`
})

</script>
<style scoped>
h1 {
  font-size: 2rem;
  margin-bottom: 1rem;
}

h2{
  font-size: 1.5rem;
  margin-bottom: 1rem;
}

.form {
  margin-bottom: 2rem;
  border: 1px solid var(--color-border);
  padding: 20px 20px 20px 20px;
  border-radius: 2rem;
}

.form form {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

label {
  display: block;
  margin-bottom: 0.5rem;
}

input{
  width: 100%;
  padding: 0.5rem;
  border-radius: 0.25rem;
  border: 1px solid var(--color-border);
}

.submit {
  background-color: var(--color-text);
  color: #000;
  border: 0;
  border-radius: 0.25rem;
  padding: 0.5rem 1rem;
  cursor: pointer;
}

textarea {
  width: 100%;
  padding: 0.5rem;
  border-radius: 0.25rem;
  border: 1px solid var(--color-border);
}

</style>

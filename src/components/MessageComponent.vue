<script setup>
import { ref, watch } from "vue";

const content = ref("")
const messages = ref(null)
const messageObject = ref( null )
const orderTypes = ref(["Latest", "Oldest", "Upvotes", "Downvotes"])
const orderType = ref("Latest")

messageObject.value = {
  content: ""
}

async function fetchData() {
  const requestUrl = "http://127.0.0.1:8080/messages"
  var res;
  switch (orderType.value) {
    case "Latest":
      res = await fetch(requestUrl + "/search/findAllByOrderByCreatedAtDesc")
      break
    case "Oldest":
      res = await fetch(requestUrl + "/search/findAllByOrderByCreatedAtAsc")
      break
    case "Upvotes":
      res = await fetch(requestUrl + "/search/findAllByOrderByUpVoteDesc")
      break
    case "Downvotes":
      res = await fetch(requestUrl + "/search/findAllByOrderByDownVoteDesc")
      break
    default:
      res = await fetch(requestUrl + "/search/findAllByOrderByCreatedAtDesc")
  }

  const messageObject = await res.json()
  messages.value = await messageObject._embedded.messages
  console.log(messages.value)
}

async function postData() {
  messageObject.value.content = content
  const res = (await fetch("http://127.0.0.1:8080/messages", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify(messageObject.value)
  }))
  content.value = ""
  await fetchData()
}

async function incrementUpVote(link) {
  await fetch(link + "/upvote")
  await fetchData()
}

async function incrementDownVote(link) {
  await fetch(link + "/downvote")
  await fetchData()
}

function dateFormatter(dateString) {
  const date = new Date(dateString)

  return date.getFullYear() + "/" + date.getMonth() + "/" + date.getDate() + " " + date.getHours() + ":" + date.getMinutes() + ":" + date.getSeconds()
}

fetchData()
watch(fetchData)

</script>

<template>
  <v-text-field @keyup.enter="postData" v-model="content" class="messageInput" label="Message" variant="solo-filled">
    <template v-slot:append-inner>
      <v-btn class="bg-lime-darken-3" @click="postData">send</v-btn>
    </template>
  </v-text-field>

  <v-sheet class="pr-3 d-flex flex-row-reverse">
    <v-btn @click="fetchData" class="bg-lime-darken-4">refresh</v-btn>
  </v-sheet>
  <v-card class="messageWrapper bg-lime-lighten-4">
    <v-select
        class="bg-lime-lighten-4 w-25 ml-auto"
        v-model="orderType"
        base-color="lime-darken-4"
        item-color="lime-darken-4"
        density="compact"
        hide-details
        :items="orderTypes"
    >
    </v-select>
    <v-list class="messageList bg-lime-lighten-4" v-for="message in messages" min-width="400">
      <v-list-item-title class="messageTitle">{{ message.content }}</v-list-item-title>
      <v-list-item-subtitle class="messageSubtitle d-flex">
        <v-sheet class="bg-lime-lighten-4 w-25 me-auto">
          {{ dateFormatter(message.createdAt) }}
        </v-sheet>
        <v-sheet style="cursor: pointer" @click="incrementUpVote(message._links.self.href)" class="bg-lime-lighten-4 text-light-blue font-weight-bold align-end">
          {{ message.upVote }}
          &nbsp
        </v-sheet>
        <v-sheet style="cursor: pointer" @click="incrementDownVote(message._links.self.href)" class="bg-lime-lighten-4 text-red font-weight-bold align-content-end">
          {{ message.downVote }}
        </v-sheet>
      </v-list-item-subtitle>
      <v-divider></v-divider>
    </v-list>
  </v-card>
</template>

<style scoped>
.messageInput {
  margin: 10px;
}
.messageWrapper {
  margin: 10px;
}
.messageList{
  padding: 10px;
}
.messageTitle {
  margin: 10px;
}
.messageSubtitle {
  margin: 10px;
}
</style>
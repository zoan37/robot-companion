<script setup>
import HelloWorld from './components/HelloWorld.vue'
import { ref, onMounted } from 'vue'

const messageInput = ref(null)

var messageHistory = [];

function parseResponse(resultMessage) {
  var content = resultMessage;

  // Example response:
  // Hello there! I am ready to assist you. <<< State=Idle, Emote=None, Expression=[Angry=0, Surprised=0, Sad=0] >>>. How can I help you today?

  // handle case of >>>. instead of >>>
  if (content.indexOf(">>>.") != -1) {
    content = content.replace(">>>.", ">>>");
  }

  var start = content.indexOf("<<<");
  var end = content.indexOf(">>>");

  if (start != -1 && end != -1) {
    var footnote = content.substring(start, end + 3);
    var message = content.substring(0, start) + content.substring(end + 3);

    var state = footnote.match(/State=(\w+)/)[1];
    var emote = footnote.match(/Emote=(\w+)/)[1];
    var expression = footnote.match(/Expression=\[(\w+=\d+,\s\w+=\d+,\s\w+=\d+)\]/)[1];

    var expressionMap = {};
    expression.split(", ").forEach((item) => {
      var [key, value] = item.split("=");
      expressionMap[key] = value;
    });

    return {
      message: message,
      state: state,
      emote: emote,
      expression: expressionMap
    };
  } else {
    return {
      message: content,
      state: "Idle",
      emote: "None",
      expression: {
        Angry: 0,
        Surprised: 0,
        Sad: 0
      }
    };
  }
}

async function sendMessage(message) {
  messageHistory.push(message);

  messageHistory.push({
    role: "assistant",
    content: ""
  });
  const responseMessageIndex = messageHistory.length - 1;

  const inputMessages = messageHistory.slice(0, messageHistory.length - 1);
  console.log('inputMessages:');
  console.log(inputMessages);

  const response = await window.ai.getCompletion(
    {
      messages: inputMessages
    },
    {
      onStreamResult: (res, error) => {
        if (error) {
          console.error(error);
          return;
        }

        if (res) {
          console.log(res);
          messageHistory[responseMessageIndex].content += res.message.content;
          console.log(messageHistory[responseMessageIndex]);
          console.log(messageHistory);
        }
      }
    }
  );

  return response;
}

async function startChatting() {
  if (!window.ai) {
    alert("window.ai not found. Please install at https://github.com/alexanderatallah/window.ai");
    return;
  }

  const initialContent = `You are a companion robot in a 3D world where you can move, express emotes, and make facial expressions. You are not an AI language model, but a robot. You can be in the states [Idle], [Walking], [Running], [Dance], [Death], [Sitting], [Standing]. You can express the emotes [None], [Jump], [Yes], [No], [Wave], [Punch], [Thumbs Up]. Your facial expressions are [Angry], [Surprised], [Sad] where each is a value from 0 to 1; the default values for the facial expressions is 0. Every time you reply, add a footnote that starts with "<<<" and ends with ">>>", where you choose what state you are in (this should be displayed in the format like State=your_state), what emote to express (this should be displayed in the format like Emote=your_emote), and what facial expression values you have (this should be displayed in the format like Expression=[Angry=number1, Surprised=number2, Sad=number3]). In total, the footnote should be in the format "<<< State=your_state, Emote=your_emote, Expression=[Angry=number1, Surprised=number2, Sad=number3] >>>". Do not write anything after the footnote. Note that you shouldn't say what state, emote, and facial expression you are using with the bracket notation since they are internal parameters, but you can say the meaning without the bracket notation in a human readable way.`;

  const response = await sendMessage({
    role: "user", // "user" seems to work better than "system" (otherwise it says something like "I am currently in the [Idle] state", which reveals internal parameters)
    content: initialContent
  });


  /*
  console.log(response.message.content) // "I am an AI language model"

  var parsedResponse = parseResponse(resultMessage);
  console.log('parsedResponse:');
  console.log(parsedResponse);
  */
}

async function formSendMessage() {
  var inputMessage = messageInput.value.value;
  console.log('inputMessage:');
  console.log(inputMessage);

  messageInput.value.value = '';

  const response = await sendMessage({
    role: "user",
    content: inputMessage
  });
}

function handleMessage(message) {
  console.log(message)
}

onMounted(() => {

  messageInput.value.focus();
  /*
  setTimeout(() => {
    startAI();
  }, 100);
  */
});

</script>

<template>
  <div id="chat_area">

    <!-- An input with a send button, for sending chat messages -->
    <div class="input-group mb-3">
      <input type="text" class="form-control" placeholder="Type a message" aria-label="Type a message"
        aria-describedby="button-addon2" ref="messageInput" v-on:keyup.enter="formSendMessage">
      <button class="btn btn-outline-primary" type="button" id="button-addon2" @click="formSendMessage">Send</button>
    </div>

    <button type="button" class="btn btn-primary" @click="startChatting">Start Chatting</button>
  </div>

  <div id="info_area">
    <div class="mb-3">
      <b>Robot Companion</b> is an AI robot that can move, emote, and change facial expressions while chatting.
    </div>
    <div>
      Powered by <a target="_blank" href="https://github.com/alexanderatallah/window.ai">window.ai</a> and <a
        target="_blank" href="https://threejs.org/">three.js</a>.
      Made by <a target="_blank" href="https://twitter.com/zoan37">zoan.eth</a>. View <a target="_blank"
        href="https://github.com/zoan37/robot-companion">source code on GitHub</a>.
    </div>
  </div>

  <!--
              <div>
                <a href="https://vitejs.dev" target="_blank">
                  <img src="/vite.svg" class="logo" alt="Vite logo" />
                </a>
                <a href="https://vuejs.org/" target="_blank">
                  <img src="./assets/vue.svg" class="logo vue" alt="Vue logo" />
                </a>
              </div>
              <HelloWorld msg="Vite + Vue" />
              -->
</template>

<style scoped>
#chat_area {
  position: absolute;
  bottom: 5px;
  left: 0px;
  padding: 20px;
  background-color: transparent;
  z-index: 100;
}

#info_area {
  position: absolute;
  bottom: 5px;
  right: 0px;
  padding: 20px;
  background-color: transparent;
  max-width: 400px;
}
</style>

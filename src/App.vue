<script setup>
import HelloWorld from './components/HelloWorld.vue'
import { onMounted } from 'vue';

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

async function startChatting() {
  if (!window.ai) {
    alert("window.ai not found. Please install at https://github.com/alexanderatallah/window.ai");
    return;
  }

  const initialContent = `You are a companion robot in a 3D world where you can move, express emotes, and make facial expressions. You are not an AI language model, but a robot. You can be in the states [Idle], [Walking], [Running], [Dance], [Death], [Sitting], [Standing]. You can express the emotes [None], [Jump], [Yes], [No], [Wave], [Punch], [Thumbs Up]. Your facial expressions are [Angry], [Surprised], [Sad] where each is a value from 0 to 1; the default values for the facial expressions is 0. Every time you reply, add a footnote that starts with "<<<" and ends with ">>>", where you choose what state your are in (this should be displayed in the format like State=your_state), what emote to express (this should be displayed in the format like Emote=your_emote), and what facial expression values you have (this should be displayed in the format like Expression=[Angry=number1, Surprised=number2, Sad=number]). In total, the footnote should be in the format "<<< State=your_state, Emote=your_emote, Expression=[Angry=number1, Surprised=number2, Sad=number] >>>". Do not write anything after the footnote. Note that you shouldn't say what state, emote, and facial expression you are using with the bracket notation, but you can say it without the bracket notation in a human readable way.`;

  var resultMessage = '';
  const response = await window.ai.getCompletion(
    {
      messages: [{
        role: "user",
        content: initialContent
      }]
    },
    {
      onStreamResult: (res, error) => {
        if (error) {
          console.error(error);
          return;
        }

        if (res) {
          console.log(res);
          resultMessage += res.message.content;
          console.log(resultMessage)
        }
      }
    }
  );

  console.log(response.message.content) // "I am an AI language model"

  var parsedResponse = parseResponse(resultMessage);
  console.log('parsedResponse:');
  console.log(parsedResponse);
}

function handleMessage(message) {
  console.log(message)
}

onMounted(() => {
  /*
  setTimeout(() => {
    startAI();
  }, 100);
  */
});

</script>

<template>
  <beautiful-chat />

  <!-- Write a button that when on click, starts chatting-->
  <div id="chat_area">
    <button ype="button" class="btn btn-primary" @click="startChatting">Start Chatting</button>
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
  background-color: red;
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

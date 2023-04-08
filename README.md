# robot-companion

<img width="500" alt="screenshot" src="https://user-images.githubusercontent.com/104385984/230722441-3f5834b2-da21-4ff1-af3e-be4d6a3f361b.png">

**Demo:** https://robot-companion.vercel.app/

Robot Companion is an AI robot that can move, emote, and change facial expressions while chatting. For example, you can say "Let's party!", and the robot will start dancing without you explictly telling it to dance.

It is powered by [window.ai](https://github.com/alexanderatallah/window.ai) and three.js. The intention was to play around with the window.ai API, which allows you to plug in your own AI model (e.g. GPT-3.5) into an app. The 3D assets and animation are from the [Three.js skinning and morphing example](https://threejs.org/examples/webgl_animation_skinning_morph.html).

Possible animations:
* States: [Idle], [Walking], [Running], [Dance], [Death]
* Emotes: [Jump], [Yes], [No], [Wave], [Punch], [Thumbs Up]
* Facial Expressions: [Angry], [Surprised], [Sad] (each value between 0 and 1)

### How it works

The AI model is given an initial prompt that says it's an AI robot in a 3D world and lists the various states, emotes, and facial expressions possible, and that it should add a footnote at the end of a reply in the format `<<< State=your_state, Emote=your_emote, Expression=[Angry=number1, Surprised=number2, Sad=number3] >>>`.

After receiving a reply from the AI model, the code parses the footnote and calls the corresponding animations, and the footnote is removed from the message before displaying it in the chat box.

---
layout: laborfolio
title: Ondoj kaj frekvencoj
js:
  - folio-0c
---

<!--
https://pressbooks.pub/sound/chapter/frequency-domain-graphs-2/

# pri apliko en JS:
https://developer.mozilla.org/en-US/docs/Web/API/OscillatorNode/setPeriodicWave
https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API/Advanced_techniques
https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API/Simple_synth

https://marcgg.com/blog/2016/11/01/javascript-audio/#
https://tonejs.github.io/
https://www.npmjs.com/package/fft-js

https://ib-lenhardt.com/kb/glossary/signal-modulation
https://www.gaussianwaves.com/2015/11/interpreting-fft-results-obtaining-magnitude-and-phase-information/
-->


<div class="container">
  <div class="keyboard"></div>
</div>
<div class="settingsBar">
  <div class="left">
    <span>Amplitudo: </span>
    <input
      type="range"
      min="0.0"
      max="1.0"
      step="0.01"
      value="0.5"
      list="volumes"
      name="volume" />
    <datalist id="volumes">
      <option value="0.0" label="Mute"></option>
      <option value="1.0" label="100%"></option>
    </datalist>
  </div>
  <div class="right">
    <span>Ondospeco: </span>
    <select name="waveform">
      <option value="custom">Propra</option>
      <option value="sine">Sinuso</option>
      <!--
      <option value="square" selected>Square</option>
      <option value="sawtooth">Sawtooth</option>
      <option value="triangle">Triangle</option>
      -->
    </select>
  </div>
</div>

<canvas width="512" height="300" id="osciloskopo"></canvas>
<canvas width="512" height="300" id="frekvencoj"></canvas>


<style>
.container {
  /*
  overflow-x: scroll;
  overflow-y: hidden;
  */
  width: 660px;
  height: 110px;
  white-space: nowrap;
  margin: 10px;
}

.keyboard {
  width: auto;
  padding: 0;
  margin: 0;
}

.key {
  cursor: pointer;
  font:
    16px "Open Sans",
    "Lucida Grande",
    "Arial",
    sans-serif;
  border: 1px solid black;
  border-radius: 5px;
  width: 20px;
  height: 80px;
  text-align: center;
  box-shadow: 2px 2px darkgray;
  display: inline-block;
  position: relative;
  margin-right: 3px;
  user-select: none;
  -moz-user-select: none;
  -webkit-user-select: none;
  -ms-user-select: none;
}

.key div {
  position: absolute;
  bottom: 0;
  text-align: center;
  width: 100%;
  pointer-events: none;
}

.key div sub {
  font-size: 10px;
  pointer-events: none;
}

.key:hover {
  background-color: #eeeeff;
}

.key:active,
.active {
  background-color: black;
  color: white;
}

.octave {
  display: inline-block;
  padding-right: 6px;
}

.settingsBar {
  padding-top: 8px;
  font:
    14px "Open Sans",
    "Lucida Grande",
    "Arial",
    sans-serif;
  position: relative;
  vertical-align: middle;
  width: 100%;
  height: 30px;
}

.left {
  width: 50%;
  position: absolute;
  left: 0;
  display: table-cell;
  vertical-align: middle;
}

.left span,
.left input {
  vertical-align: middle;
}

.right {
  width: 50%;
  position: absolute;
  right: 0;
  display: table-cell;
  vertical-align: middle;
}

.right span {
  vertical-align: middle;
}

.right input {
  vertical-align: baseline;
}

</style>

<script>


const audioContext = new AudioContext();
const oscList = [];
let mainGainNode = null;
const keyboard = document.querySelector(".keyboard");
const wavePicker = document.querySelector("select[name='waveform']");
const volumeControl = document.querySelector("input[name='volume']");
let customWaveform = null;
let sineTerms = null;
let cosineTerms = null;

let analyser = null;
let dataArray = [];
let bufferLength = null;

const canvas = [document.getElementById("osciloskopo"),document.getElementById("frekvencoj")];
const canvasCtx = canvas.map((c) => c.getContext("2d"));

function notoj(okt=4) {
  // https://en.wikipedia.org/wiki/Pitch_(music)
  const fq4 = {
    c: 261.63,
    "c#": 277.18,
    d: 293.67,
    "d#": 311.125,
    e: 329.625,
    f: 349.23,
    "f#": 370,
    g: 392,
    "g#": 415.3,
    a: 440,
    "a#": 466.16,
    h: 493.875
  }
  return Object.entries(fq4).map(([noto,frekv]) => [
    noto,
    frekv * Math.pow(2,okt-4)
  ])
}

/*
function createNoteTable() {
  const noteFreq = [
    { A: 27.5, "A#": 29.13523509488062, B: 30.867706328507754 },
    {
      C: 32.70319566257483,
      "C#": 34.64782887210901,
      D: 36.70809598967595,
      "D#": 38.89087296526011,
      E: 41.20344461410874,
      F: 43.65352892912549,
      "F#": 46.2493028389543,
      G: 48.99942949771866,
      "G#": 51.91308719749314,
      A: 55,
      "A#": 58.27047018976124,
      B: 61.73541265701551,
    },
  ];
  for (let octave = 2; octave <= 7; octave++) {
    noteFreq.push(
      Object.fromEntries(
        Object.entries(noteFreq[octave - 1]).map(([key, freq]) => [
          key,
          freq * 2,
        ]),
      ),
    );
  }
  noteFreq.push({ C: 4186.009044809578 });
  return noteFreq;
}
*/

function setup() {
  //const noteFreq = createNoteTable();

  volumeControl.addEventListener("change", changeVolume, false);

  mainGainNode = audioContext.createGain();

  analyser = audioContext.createAnalyser();
  analyser.fftSize = 512;

  bufferLength = analyser.frequencyBinCount;
  dataArray = [new Uint8Array(bufferLength),new Uint8Array(bufferLength)];
  analyser.getByteTimeDomainData(dataArray[0]);
  analyser.getByteFrequencyData(dataArray[1]);

  //mainGainNode.connect(audioContext.destination);
  mainGainNode.connect(analyser);
  analyser.connect(audioContext.destination);

  mainGainNode.gain.value = volumeControl.value;

  // Create the keys; skip any that are sharp or flat; for
  // our purposes we don't need them. Each octave is inserted
  // into a <div> of class "octave".

  // 4-a oktavo
  notoj(4).forEach(([noto,frekv]) => {
    if (noto.length === 1) {
      keyboard.appendChild(createKey(noto, 4, frekv));
    }
  });

  const c5 = notoj(5)[0];
  keyboard.appendChild(createKey(c5[0], 5, c5[1]));

/*
  noteFreq.forEach((keys, idx) => {
    const keyList = Object.entries(keys);
    const octaveElem = document.createElement("div");
    octaveElem.className = "octave";

    keyList.forEach((key) => {
      if (key[0].length === 1) {
        octaveElem.appendChild(createKey(key[0], idx, key[1]));
      }
    });

    keyboard.appendChild(octaveElem);
  });

  document
    .querySelector("div[data-note='B'][data-octave='5']")
    .scrollIntoView(false);
*/

  //sineTerms = new Float32Array([0, 0, 1, 0, 1]);
  //sineTerms = new Float32Array([0, 1, 0, 0.5, 0, 0.25]);
  //sineTerms = new Float32Array([0, 1,0.5, 0.3, 0.1]);  
  // sineTerms = new Float32Array([0, 1, 0, 0.5, 0, 0.2, 0, 0.1]);


  sineTerms = new Float32Array([0, 1.0, 0, 0.4, 0, 0.1, 0, 0.05]);
  cosineTerms = new Float32Array(sineTerms.length); // ĉio 0 - neniu fazo
  //cosineTerms = sineTerms; //

  customWaveform = audioContext.createPeriodicWave(cosineTerms, sineTerms);

  for (let i = 0; i < 9; i++) {
    oscList[i] = {};
  }

}

setup();

// draw an oscilloscope of the current audio source

function draw(c) {
  function nulejo(data,nul=128) {
    for (let i = 0; i < bufferLength-1; i++) {
      if (data[i]<nul && data[i+1] >= nul)
        return i;
    }
    return 0;
  }

  // mendu la sekvan desegnon de la diagramo
  requestAnimationFrame(() => draw(c));
  let offset = 0;

  if (c == 0) {
    analyser.getByteTimeDomainData(dataArray[c]);
    offset = nulejo(dataArray[c]);
    console.log("offset: "+offset);
    //if (!offset) console.log(dataArray[c]);
  } else {
    analyser.getByteFrequencyData(dataArray[c]);
  }

  const cnv = canvas[c];
  const ctx = canvasCtx[c];

  ctx.fillStyle = "rgb(200 200 200)";
  ctx.fillRect(0, 0, cnv.width, cnv.height);

  ctx.lineWidth = 2;
  ctx.strokeStyle = "rgb(0 0 0)";

  ctx.beginPath();

  const sliceWidth = (cnv.width * 1.0) / bufferLength;
  let x = 0;

  //for (let i = offset; i < bufferLength+offset; i++) {
  for (let i = offset; i < bufferLength; i++) {
    const v = dataArray[c][i%bufferLength] / 128.0;
    const y = (v * cnv.height) / 2;

    if (i === offset) {
      ctx.moveTo(x, y);
    } else {
      ctx.lineTo(x, y);
    }

    x += sliceWidth;
  }

  //ctx.lineTo(cnv.width, cnv.height / 2);
  ctx.stroke();
}

draw(0);
draw(1);


function createKey(note, octave, freq) {
  const keyElement = document.createElement("div");
  const labelElement = document.createElement("div");

  keyElement.className = "key";
  keyElement.dataset["octave"] = octave;
  keyElement.dataset["note"] = note;
  keyElement.dataset["frequency"] = freq;
  labelElement.appendChild(document.createTextNode(note));
  labelElement.appendChild(document.createElement("sub")).textContent = octave;
  keyElement.appendChild(labelElement);

  keyElement.addEventListener("mousedown", notePressed, false);
  keyElement.addEventListener("mouseup", noteReleased, false);
  keyElement.addEventListener("mouseover", notePressed, false);
  keyElement.addEventListener("mouseleave", noteReleased, false);

  return keyElement;
}
function playTone(freq) {
  const osc = audioContext.createOscillator();
  osc.connect(mainGainNode);

  const type = wavePicker.options[wavePicker.selectedIndex].value;

  if (type === "custom") {
    osc.setPeriodicWave(customWaveform);
  } else {
    osc.type = type;
  }

  osc.frequency.value = freq;
  osc.start();

  return osc;
}
function notePressed(event) {
  if (event.buttons & 1) {
    const dataset = event.target.dataset;

    if (!dataset["pressed"] && dataset["octave"]) {
      const octave = Number(dataset["octave"]);
      oscList[octave][dataset["note"]] = playTone(dataset["frequency"]);
      dataset["pressed"] = "yes";
    }
  }
}
function noteReleased(event) {
  const dataset = event.target.dataset;

  if (dataset && dataset["pressed"]) {
    const octave = Number(dataset["octave"]);

    if (oscList[octave] && oscList[octave][dataset["note"]]) {
      oscList[octave][dataset["note"]].stop();
      delete oscList[octave][dataset["note"]];
      delete dataset["pressed"];
    }
  }
}
function changeVolume(event) {
  mainGainNode.gain.value = volumeControl.value;
}
const synthKeys = document.querySelectorAll(".key");
// prettier-ignore
const keyCodes = [
  "Space",
  "ShiftLeft", "KeyZ", "KeyX", "KeyC", "KeyV", "KeyB", "KeyN", "KeyM", "Comma", "Period", "Slash", "ShiftRight",
  "KeyA", "KeyS", "KeyD", "KeyF", "KeyG", "KeyH", "KeyJ", "KeyK", "KeyL", "Semicolon", "Quote", "Enter",
  "Tab", "KeyQ", "KeyW", "KeyE", "KeyR", "KeyT", "KeyY", "KeyU", "KeyI", "KeyO", "KeyP", "BracketLeft", "BracketRight",
  "Digit1", "Digit2", "Digit3", "Digit4", "Digit5", "Digit6", "Digit7", "Digit8", "Digit9", "Digit0", "Minus", "Equal", "Backspace",
  "Escape",
];
function keyNote(event) {
  const elKey = synthKeys[keyCodes.indexOf(event.code)];
  if (elKey) {
    if (event.type === "keydown") {
      elKey.tabIndex = -1;
      elKey.focus();
      elKey.classList.add("active");
      notePressed({ buttons: 1, target: elKey });
    } else {
      elKey.classList.remove("active");
      noteReleased({ buttons: 1, target: elKey });
    }
    event.preventDefault();
  }
}
addEventListener("keydown", keyNote);
addEventListener("keyup", keyNote);
</script>



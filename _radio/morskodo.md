---
layout: page
title: "Morsa kodo"
---

<!--

https: "//eo.wikipedia.org/wiki/Morsa_kodo


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

<!--canvas width="512" height="300" id="osciloskopo"></canvas-->
<canvas width="200" height="200" id="magnitudoj"></canvas>
<br/>
<canvas width="200" height="600" id="akvofalo"></canvas>


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

const morsfrekvenco = 700;
const audioContext = new AudioContext();

  // longoj: punkto = 1, streko = 3, inter literoj = 3, inter vortoj = 7

mkodo = {  
  "A": "•-",
  "B": "-•••",
  "C": "-•-•",
  "Ĉ": "-•-••",
  "D": "-••",
  "E": "•",
  "F": "••-•",
  "G": "--•",
  "Ĝ": "--•-•",
  "H": "••••",
  "Ĥ": "-•--•", //aŭ ": "----",
  "I": "••",
  "J": "•---",
  "Ĵ": "•---•",
  "K": "-•-",
  "L": "•-••",
  "M": "--",
  "N": "-•",
  "O": "---",
  "P": "•--•",
  "R": "•-•",
  "S": "•••",
  "Ŝ": "•••-•",
  "T": "-",
  "U": "••-",
  "Ŭ": "••--",
  "V": "•••-",
  "Z": "--••",
  
  "Q": "--•-",
  "W": "•--",
  "X": "-••-",
  "Y": "-•--",
  
  "0": "-----",
  "1": "•----",
  "2": "••---",
  "3": "•••--",
  "4": "••••-",
  "5": "•••••",
  "6": "-••••",
  "7": "--•••",
  "8": "---••",
  "9": "----•",
  
  ".": "•-•-•-",
  ",": "--••--",
  ":": "---•••",
  ";": "-•-•-•",
  "?": "••--••",
  "'": "•----•",
  "-": "-••••-",
  "/": "-••-•",
  "(": "-•--•",
  ")": "-•--•-",
  "\"": "•-••-•",
  
  "=": "-•••-",
  "+": "•-•-•",
  "@": "•--•-•"
}

// Q-kodoj, por demando aldonu demandsignon, por respondo uzu sen tiu
// https://eo.wikipedia.org/wiki/Q-kodo
// https://www.eventoj.hu/steb/vortaroj/hams-interpreter-DL1CU.pdf
// en praktiko nur parto de la listigitaj kodoj estas uzata kaj devas esti lernita:
// QRG, QRL, QRM, QRN, QRO, QRP, QRQ, QRS, QRT, QRV, QRX, QRZ, QSB, QSL, QSO, QSP, QSY kaj QTH. 

// respondoj povas esti ciferoj 1..5 laŭ
// https://eo.wikipedia.org/wiki/RST-kodo
//    R legeblo/komprenebleco // "readability
//    S signalforteco // strength
//    T tono // tone, la tono de morsa signalo
//
//    1 Nekomprenebla
//    2 Apenaŭ komprenebla, kompreneblas iuj vortoj
//    3 Komprenebla ege malfacile
//    4 Komprenebla facile
//    5 Perfekte komprenebla

vortareto = {
  "CQ": "ĝenerala alvoko!", // seek you
  "73!": "ĝis!",
  "YL": "fraŭlino", // young lady = fraŭlino, aĝo ne vere gravas!
  "XYL": "eksfraŭlino",
  "88": "kisoj",
  "SKEDO": "rendevuo", // el angla "schedule", aranĝon laŭ frekvenco kaj tempo
  "PSE": "bonvolu",
  "K": "venu",
  "QRG": "frekvenco",
  "QRM": "intermikso", // homaj signalĝenoj
  "QRN": "bruo", // naturaj signalĝenoj
  "QRV": "sendpreta",
  "QRX": "momenton!",
  "QRZ": "kiu vokas?",
  "QSB": "malfortiĝo",
  "QSL": "konfirmkarto",
  "QSO": "kontakto",
  "QTH": "loko"
}

mallongigoj = {
  "QRA": ["Kiu estas adreso de via stacio?", "Adreso de mia stacio estas ... ."],
  "QRB": ["Kiu estas distanco inter vi kaj mi?", "Distanco inter vi kaj mi estas ... ."],
  "QRG": ["Diru precizan frekvencon", "Frekvenco estas ... .", "frekvenco"],
  "QRH": ["Ĉu ŝanĝiĝas mia frekvenco?", "Via frekvenco ŝanĝiĝas."],
  "QRI": ["Ĉu tono de mia elsendo ŝanĝiĝas?", "Tono de via elsendo ŝanĝiĝas."],
  "QRJ": ["Ĉu mia signalo estas nesufiĉa por ricevado?", "Via signalo estas malforta, la ricevado malfacilas."],
  "QRK": ["Kiu estas komprenebleco de mia elsendo", "Komprenebleco de via elsendo estas ... "], // RST
  "QRL": ["Ĉu vi estas okupita?", "Mi estas okupita."],
  "QRM": ["Ĉu vi havas malhelpajn signalojn de la aliaj stacioj?", "Mi havas malhelpajn signalojn de la aliaj stacioj."],
  "QRN": ["Ĉu vi havas atmosferajn malhelpaĵojn?", "Mi havas atmosferajn malhelpaĵojn.", "bruo"],
  "QRO": ["Ĉu mi devas pligrandigi povumon de mia elsendo?", "Pligrandigu povumon de via elsendo."],
  "QRP": ["Ĉu mi devas malpligrandigi povumon de mia elsendo?", "Malpligrandigu povumon de via elsendo."], 
  "QRQ": ["Ĉu mi devas pligrandigi rapidon de mia elsendo?", "Pligrandigu rapidon de via elsendo."],
  "QRS": ["Ĉu mi devas malpligrandigi rapidon de mia elsendo?", "Malpligrandigu rapidon de via elsendo."],
  "QRT": ["Ĉu mi devas ĉesi mian elsendon?", "Ĉesu vian elsendon. Aŭ: mi ĉesas mian elsendon."],
  "QRU": ["Ĉu vi havas ion por mi?", "Mi ne (plu) havas ion ajn por vi."],
  "QRV": ["Ĉu vi pretas ricevadi?", "Mi pretas ricevadi. Aŭ: mi estas sendpreta."],
  "QRW": ["Ĉu mi diru al ... (la tria stacio), ke vi vokas ĝin (je frekvenco … kHz (MHz))?", "Diru al ... (la tria stacio), ke mi vokas ĝin (je frekvenco … kHz (MHz))"],
  "QRX": ["Kiam vi denove vokos min? Aŭ simple: momenton bv!", "Mi denove vokos vin je ... (tempo)."], // +tempo UTC, frekvenco kHz
  "QRY": ["Kiu estas mia vico?", "Via vico estas ... ."],
  "QRZ": ["Kiu vokas min? Aŭ: Bv. diri vian voksignon.", "Vin vokas ... ."], // demando kun propra voksigno: QRZ (de) ...
  "QSA": ["Kiu estas forto de mia elsendo? (1 ... 5)", "Forto de via elsendo estas ... (1 ... 5)."], // RST
  "QSB": ["Ĉu mia signalo havas okazajn malfortiĝojn?", "Via signalo havas okazajn malfortiĝojn."],
  "QSD": ["Ĉu mia manipulo (telegrafa modulo) havas difektojn?", "Via manipulo (telegrafa modulo) havas difektojn."],
  "QSL": ["Ĉu vi konfirmos ricevon?", "Mi konfirmos ricevon."], // ankaŭ nomo de konfirmkarto"
  "QSO": ["Ĉu vi povas interkomunikiĝi senpere kun ... ?", "Mi povas interkomunikiĝi senpere kun ... ."],
  "QSP": ["Ĉu vi povas transsendi (al)... ?", "Mi transsendos (al)... ."],
  "QSQ": ["Ĉu elsendi vortojn pounufoje?", "Elsendu vortojn pounufoje."],
  "QSV": ["Ĉu vi povas elsendi literojn 'V' por agordo?", "Mi elsendas literojn 'V' por agordo."],
  "QSW": ["Ĉu vi povas elsendi je frekvenco … kHz (MHz)?", "Mi elsendos je frekvenco … kHz (MHz)."],
  "QSX": ["Ĉu vi aŭskultas je frekvenco … kHz (MHz)?", "Mi aŭskultas je frekvenco … kHz (MHz)."],
  "QSY": ["Ĉu mi devas ŝanĝi frekvencon al … kHz (MHz)?", "Ŝanĝu frekvencon al … kHz (MHz). Aŭ: mi ŝanĝas frekvencon al"],
  "QSZ": ["Ĉu mi devas sendi ĉiun vorton dufoje?", "Sendu ĉiun vorton dufoje."],
  "QTC": ["Ĉu vi havas mesaĝojn?", "Mi havas mesaĝojn por vi."],
  "QTH": ["Diru viajn koordinatojn.", "Miaj koordinatoj estas.", "loko"],
  "QTR": ["Diru tempon.", "Tempo estas ... ."],
  "QTU": ["Kiam funkcias via stacio?", "Mia stacio funkcias ekde ... ĝis ... ."],
  "QUA": ["Ĉu vi havas mesaĝojn de ... ?", "Mesaĝoj de ... estas: ..."],
}

function bitmasko(signoj) {
  function bm(signo) {
    const kodo = mkodo[signo];
    if (kodo) {
      let b = 0b0; // la bitoj de la morsokodo: 0 paŭzo, 1 punkto, 111 streko
      let m = 0b1; // pozicio en b (masko)
      for (const k of kodo) {
        switch (k) {
          case "-": b |=m; m<<=1; b |=m; m<<=1; // aldonu unuan kaj duan biton por streko
          case "•": b |= m; m<<=1; // unu bito por punkto, resp. tria bito por streko
          default: m<<=1; // post ĉiu pepo aldonu paŭzeton
        }
      }
      return b;
    }
  }

  let kodo = [];
  for (s of signoj.toUpperCase()) {
    const b = bm(s)
    kodo.push(b);
    console.log(s+": "+b.toString(2));
  };
  return kodo;
}

//let morsaktiva = false; // sono jes/ne
let masko = 0b1; // pozicio en b (masko)
let tempilo;
let tempunuo = 100; //ms
let morsilo;

// enpakas "coroutine" fun
function coroutine(fun) {
  let cr = fun(); // perparu fun
  cr.next(); // rulu ĝis yield
  return function(x) {
    cr.next(x);
  }
}

function sendu(teksto) {
  const kodoj = bitmasko(teksto);
  sendu_kodojn(kodoj);
}

function sendu_kodojn(kodoj) {

  // preparo de la oscililo
  const osc = audioContext.createOscillator();
  osc.connect(mainGainNode);

  const type = wavePicker.options[wavePicker.selectedIndex].value;

  if (type === "custom") {
    osc.setPeriodicWave(customWaveform);
  } else {
    osc.type = type;
  }

  osc.frequency.value = morsfrekvenco;

  // sendo de la signalo laŭ kodo
  morsilo = coroutine(function*() {
    console.log("osc.start");
    osc.start();
    for (const k of kodoj) {
      console.log("sendota: "+k);
      masko = 0b1;
      tempilo = setInterval(sendu_biton,tempunuo,k,osc);
      mainGainNode.gain.value = 0; // mallaŭte
      yield;
    };
    console.log("osc.stop");
    osc.stop();
  });
}

function sendu_biton(k,osc) {
  const b = k & masko;
  masko <<= 1;

  if (b) { //} && !morsaktiva) {
    //morsaktiva = true;
    mainGainNode.gain.value = volumeControl.value;
  } else { //if (!b && morsaktiva) {
    //morsaktiva = false;
    mainGainNode.gain.value = 0;
  }

  if (masko > (0b1 << 20)) {
    console.log("sendita: "+k);
    //osc.stop();
    clearInterval(tempilo);
    // sekva kodero
    morsilo();
  }
}


const oscList = [];
let mainGainNode = null;
const keyboard = document.querySelector(".keyboard");
const wavePicker = document.querySelector("select[name='waveform']");
const volumeControl = document.querySelector("input[name='volume']");
let customWaveform = null;
let sineTerms = null;
let cosineTerms = null;

let analyser = null;
let dataArray;
let bufferLength = null;

const canvas = {
  magn: document.getElementById("magnitudoj"),
  falo: document.getElementById("akvofalo")
};
const ctxMagn = canvas["magn"].getContext("2d");
const ctxFalo = canvas["falo"].getContext("2d");

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
  analyser.fftSize = 128; // 256; // 512;
  analyser.smoothingTimeConstant = 0.2;

  bufferLength = analyser.frequencyBinCount;
  dataArray = new Uint8Array(bufferLength);
  analyser.getByteFrequencyData(dataArray);

  //mainGainNode.connect(audioContext.destination);
  mainGainNode.connect(analyser);
  analyser.connect(audioContext.destination);

  mainGainNode.gain.value = volumeControl.value;

  // Create the keys; skip any that are sharp or flat; for
  // our purposes we don't need them. Each octave is inserted
  // into a <div> of class "octave".

/*
  // 4-a oktavo
  notoj(4).forEach(([noto,frekv]) => {
    if (noto.length === 1) {
      keyboard.appendChild(createKey(noto, 4, frekv));
    }
  });
*/

  const f5 = notoj(5)[5];
  keyboard.appendChild(createKey(f5[0], 5, f5[1]));


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

function magnitudoj() {
  function nulejo(data,nul=128) {
    for (let i = 0; i < bufferLength-1; i++) {
      if (data[i]<nul && data[i+1] >= nul)
        return i;
    }
    return 0;
  }

  const cnv = canvas["magn"];
  const ctx = ctxMagn;

  // mendu la sekvan desegnon de la diagramo
  requestAnimationFrame(() => magnitudoj());
  let offset = 0;

  analyser.getByteFrequencyData(dataArray);

  ctx.fillStyle = "rgb(200 200 200)";
  ctx.fillRect(0, 0, cnv.width, cnv.height);

  ctx.lineWidth = 2;
  ctx.strokeStyle = "rgb(0 0 0)";

  ctx.beginPath();

  const sliceWidth = (cnv.width * 1.0) / bufferLength;
  let x = 0;

  //for (let i = offset; i < bufferLength+offset; i++) {
  for (let i = 0; i < bufferLength; i++) {
    const v = dataArray[i] / 128.0;
    const y = (v * cnv.height) / 2 - 5;

    ctx.lineTo(x, cnv.height-y);

    x += sliceWidth;
  }

  //ctx.lineTo(cnv.width, cnv.height / 2);
  ctx.stroke();
}

let frameCounter = 0;
const frameRatio = 1; // 1..5.. ( 1 = ĉiufoje, ~60/s)

function akvofalo() {
  requestAnimationFrame(() => akvofalo());

  frameCounter++;

  if (frameCounter > frameRatio) {
      
    analyser.getByteFrequencyData(dataArray);

    const cnv = canvas["falo"];
    const ctx = ctxFalo;
    const lineHeight = 1;

    function getColor(value) {
        // 1. Normalize the 0-255 value to a Hue (0-360) for a full rainbow
        // We use a range, for instance, 240 (blue) to 0 (red) for a nice spectrum.
        // To get a full spectrum, map 0-255 to 0-360: (value / 255) * 360
        const hue = 180 + Math.round((value / 128.0) * 360);
        const hel = 5 + Math.round((value / 128.0) * 60);

        // 2. Return the HSL color string (using 50% saturation and 50% lightness for vibrant colors)
        return `hsl(${hue}, 100%, ${hel}%)`;
    }  

    // ŝovu ĉion malsupren
    ctx.drawImage(
        cnv, // Source: The cnv itself
        0, 0, // Source X, Y (Start at the top-left)
        cnv.width, cnv.height - lineHeight, // Source Width, Height (Exclude the bottom strip that will move off-screen)
        0, lineHeight, // Destination X, Y (Draw it starting 'lineHeight' pixels from the top)
        cnv.width, cnv.height - lineHeight // Destination Width, Height
    );

    ctx.fillStyle = "rgb(0 0 0)";
    ctx.fillRect(0, 0, cnv.width, lineHeight);

  ////

    const sliceWidth = (cnv.width * 1.0) / bufferLength;
    let x = 0;

    for (let i = 0; i < bufferLength; i++) {
      //const v = dataArray[i] / 128.0;
      const color = getColor(dataArray[i]);

        ctx.beginPath();
        // Use the color for the fill style
        ctx.fillStyle = color;

        // Draw the dot (a circle)
        ctx.fillRect(
            i, 
            0, 
            sliceWidth,
            sliceWidth 
        );
        ctx.closePath();
    }

    frameCounter = 0;
  }
}

magnitudoj();
akvofalo();

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



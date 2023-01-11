<script>
  import { onMount } from "svelte";
  import * as Tone from "tone";

  class SampleObject {
    constructor(url, name, imageUrl, alt) {
      // Constructor
      this.url = url;
      this.name = name;
      this.imageUrl = imageUrl;
      this.alt = alt;
    }
  }
  let sounds = [
    new SampleObject(
      "https://cdn.pixabay.com/download/audio/2022/03/10/audio_9fe79df036.mp3?filename=safari-kick-2-37314.mp3",
      "Bass Drum",
      "https://cdn-icons-png.flaticon.com/128/1639/1639499.png"
    ),
    new SampleObject(
      "https://cdn.pixabay.com/download/audio/2022/03/15/audio_5245fcde09.mp3?filename=kick-bahadi-89434.mp3",
      "Kick Drum",
      "https://cdn-icons-png.flaticon.com/128/4268/4268740.png"
    ),
    new SampleObject(
      "https://cdn.pixabay.com/download/audio/2022/03/19/audio_942d475509.mp3?filename=clap-2-95736.mp3",
      "Clap",
      "https://cdn-icons-png.flaticon.com/128/1629/1629860.png"
    ),
    new SampleObject(
      "https://cdn.pixabay.com/download/audio/2022/06/05/audio_97ed83a845.mp3?filename=snare-112754.mp3",
      "Snare",
      "https://cdn-icons-png.flaticon.com/128/5482/5482949.png"
    ),
    new SampleObject(
      "https://cdn.pixabay.com/download/audio/2022/03/15/audio_37b9b2d333.mp3?filename=spurc_clhat-80825.mp3",
      "Hi Hat",
      "https://cdn-icons-png.flaticon.com/128/2768/2768561.png"
    ),
    new SampleObject(
      "https://cdn.pixabay.com/download/audio/2022/01/18/audio_fb6e6bf2f6.mp3?filename=left-crashwav-14567.mp3",
      "Cymbal Crash",
      "https://cdn-icons-png.flaticon.com/128/7662/7662976.png"
    ),
    new SampleObject(
      "https://tonejs.github.io/audio/berklee/gong_1.mp3",
      "Gong",
      "https://cdn-icons-png.flaticon.com/128/451/451990.png"
    ),
    new SampleObject(
      "https://cdn.pixabay.com/download/audio/2022/03/26/audio_6f6f7af49d.mp3?filename=fx_wark-107953.mp3",
      "Fx",
      "https://cdn-icons-png.flaticon.com/512/5229/5229543.png"
    ),
    new SampleObject(
      "https://cdn.pixabay.com/download/audio/2022/12/06/audio_16a5327172.mp3?filename=burst-128424.mp3",
      "Burst",
      "https://cdn-icons-png.flaticon.com/512/959/959711.png"
    ),
    new SampleObject(
      "https://cdn.pixabay.com/download/audio/2021/08/04/audio_19faff7fe5.mp3?filename=elephant-trumpets-growls-6047.mp3",
      "Elephant",
      "https://cdn-icons-png.flaticon.com/512/47/47101.png"
    ),
    new SampleObject(
      "https://cdn.pixabay.com/download/audio/2022/10/23/audio_b3ca30d553.mp3?filename=horse-123780.mp3",
      "Horse",
      "https://cdn-icons-png.flaticon.com/512/33/33751.png"
    ),
    new SampleObject(
      "https://cdn.pixabay.com/download/audio/2022/06/08/audio_98351be2c8.mp3?filename=duck-quack-112941.mp3",
      "Duck",
      "https://cdn-icons-png.flaticon.com/512/6959/6959777.png"
    )
  ];
  let MIDI_NUM_NAMES = [
    "C_1",
    "C#_1",
    "D_1",
    "D#_1",
    "E_1",
    "F_1",
    "F#_1",
    "G_1",
    "G#_1",
    "A_1",
    "A#_1",
    "B_1",
    "C0",
    "C#0",
    "D0",
    "D#0",
    "E0",
    "F0",
    "F#0",
    "G0",
    "G#0",
    "A0",
    "A#0",
    "B0",
    "C1",
    "C#1",
    "D1",
    "D#1",
    "E1",
    "F1",
    "F#1",
    "G1",
    "G#1",
    "A1",
    "A#1",
    "B1",
    "C2",
    "C#2",
    "D2",
    "D#2",
    "E2",
    "F2",
    "F#2",
    "G2",
    "G#2",
    "A2",
    "A#2",
    "B2",
    "C3",
    "C#3",
    "D3",
    "D#3",
    "E3",
    "F3",
    "F#3",
    "G3",
    "G#3",
    "A3",
    "A#3",
    "B3",
    "C4",
    "C#4",
    "D4",
    "D#4",
    "E4",
    "F4",
    "F#4",
    "G4",
    "G#4",
    "A4",
    "A#4",
    "B4",
    "C5",
    "C#5",
    "D5",
    "D#5",
    "E5",
    "F5",
    "F#5",
    "G5",
    "G#5",
    "A5",
    "A#5",
    "B5",
    "C6",
    "C#6",
    "D6",
    "D#6",
    "E6",
    "F6",
    "F#6",
    "G6",
    "G#6",
    "A6",
    "A#6",
    "B6",
    "C7",
    "C#7",
    "D7",
    "D#7",
    "E7",
    "F7",
    "F#7",
    "G7",
    "G#7",
    "A7",
    "A#7",
    "B7",
    "C8",
    "C#8",
    "D8",
    "D#8",
    "E8",
    "F8",
    "F#8",
    "G8",
    "G#8",
    "A8",
    "A#8",
    "B8",
    "C9",
    "C#9",
    "D9",
    "D#9",
    "E9",
    "F9",
    "F#9",
    "G9"
  ];
  let colors = [
    "blue",
    "red",
    "green",
    "teal",
    "pink",
    "white",
    "purple",
    "orange",
    "yellow",
    "brown",
    "blue",
    "green",
    "teal",
    "white",
    "pink",
    "purple",
    "yellow",
    "brown",
    "pink",
    "blue"
  ];
  let timings = ["1m", "2n", "2t", "4n", "4t", "8n", "8t", "16n", "16t"];
  let effects = ["Reverb", "Big Reverb"];
  let currentTiming;
  let players;
  let analysers = [];
  let recorder;
  let downloadContainer;
  let status = "";
  let currentSoundIndex;
  let mediaRecorder;
  let chunks = [];
  let tempo;
  let currentSoundID;
  let currentSoundEffect;
  let piano;

  onMount(() => {
    // microphone recorder
    navigator.mediaDevices.getUserMedia({ audio: true }).then((stream) => {
      mediaRecorder = new MediaRecorder(stream);
      // mediaRecorder.start();
    });

    // Tone.js
    tempo = 120;
    Tone.Transport.bpm.value = tempo;

    recorder = new Tone.Recorder();
    players = sounds.map((sound) => {
      // Create an analyser node for each sound
      let analyser = new Tone.Analyser("waveform", 128);

      // Create player for each sound
      let player = new Tone.Player(sound.url)
        .connect(recorder)
        .connect(analyser)
        .toDestination();
      analysers.push(analyser);

      return player;
    });
    //	PIANO
    piano = new Tone.PolySynth(Tone.Synth, {
      volume: -8,
      oscillator: {
        partials: [1, 2, 5]
      },
      portamento: 0.005
    }).toDestination();
  });

  function playSound(index, loopPoint) {
    let player = players[index];
    let analyser = analysers[index];

    if (loopPoint) {
      const loop = new Tone.Loop((time) => {
        Tone.Transport.scheduleRepeat(() => {
          player.start(Tone.Transport.nextSubdivision(loopPoint));
        }, loopPoint);
      }, "1m").start(Tone.Transport.nextSubdivision("1m"));

      Tone.Transport.start();
      Tone.start();
    } else player.start();
    console.log(analyser.getValue());
    const values = analyser.getValue();

    let soundIcon = document.getElementById(loopPoint);

    for (let i = 0; i < values.length; i++) {
      const amplitude = values[i];
      // const x = map(i, 0, values.length - 1, 0, width);
      // const y = height / 2 + amplitude * height;
      console.log(amplitude);
    }
  }
  let stopPlay = async () => {
    // the recorded audio is returned as a blob
    const recording = await recorder.stop();

    // download the recording by creating an anchor element and blob url
    const url = URL.createObjectURL(recording);
    const anchor = document.createElement("a");
    anchor.download = url;
    anchor.href = url;
    anchor.textContent = "DOWNLOAD";
    downloadContainer.appendChild(anchor);

    // Show recording
    const audio = document.getElementById("sample-recording");
    audio.hidden = false;
    audio.src = url;
    console.log(url);
  };

  function handleDragEnter(e) {
    status =
      "You are dragging over the " +
      (e.target.getAttribute("id") || e.target.getAttribute("alt"));
    currentSoundIndex = e.target.getAttribute("name");
    currentSoundID =
      e.target.getAttribute("id") || e.target.getAttribute("alt");
  }

  function handleDragStart(e) {
    status = "Dragging the element " + e.target.getAttribute("id");

    e.dataTransfer.dropEffect = "move";
    e.dataTransfer.setData("text", e.target.getAttribute("id"));

    if (timings.includes(e.target.innerText)) {
      currentTiming = e.target.innerText;
      console.log("TIMING", currentTiming);
    }

    if (effects.includes(e.target.innerText)) {
      currentSoundEffect = e.target.innerText;
      console.log("currentSoundEffect", currentSoundEffect);
    }
  }

  function handleDragDrop(e) {
    e.preventDefault();
    console.log("DROPPED", e.target.id);

    if (currentSoundEffect) {
      let player = players[currentSoundIndex];
      const reverb = new Tone.Reverb().toDestination();
      const bigReverb = new Tone.Reverb().toDestination();
      bigReverb.decay = 6;
      // bigReverb.wet = .2
      bigReverb.preDelay = 0.09;
      currentSoundEffect == "Reverb" && player.connect(reverb);
      currentSoundEffect == "Big Reverb" && player.connect(bigReverb);

      let soundDiv = document.getElementById(currentSoundID + "-holder");
      const divCopy = document.createElement("div");
      divCopy.textContent = currentSoundEffect;
      divCopy.className = "effect col";

      soundDiv.appendChild(divCopy);
    }

    if (currentTiming) {
      playSound(currentSoundIndex, currentTiming);
      // What was dragged
      let measureIcon = document.getElementById(e.target.id);

      let soundDiv = document.getElementById(currentSoundID + "-holder");
      const divCopy = document.createElement("div");
      divCopy.textContent = measureIcon.textContent;
      divCopy.className = "timing col";

      soundDiv.appendChild(divCopy);
    }
    currentSoundEffect = "";
    currentTiming = "";
    currentSoundID = "";
    currentSoundIndex = "";
  }

  async function startAudioRecording() {
    mediaRecorder.start();

    mediaRecorder.ondataavailable = (e) => {
      chunks.push(e.data);
    };
    const time = Tone.Time("1m").toSeconds();
    await new Promise((r) => setTimeout(r, time * 1000));
    stopAudioRecording();
  }
  function stopAudioRecording() {
    mediaRecorder.stop();
    mediaRecorder.onstop = (e) => {
      const audio = document.getElementById("audio-recording");
      const blob = new Blob(chunks, { type: "audio/ogg; codecs=opus" });
      chunks = [];
      const audioURL = window.URL.createObjectURL(blob);
      audio.src = audioURL;
      audio.hidden = false;
      const newSoundDiv = document.getElementsByClassName("sound-clip")[0];
      newSoundDiv.hidden = false;
      newSoundDiv.id = audioURL;
      // TODO: Add logic to create new player
    };
  }

  var familyOfTriads = [
    [0, 4, 7],
    [0, 3, 7],
    [0, 3, 6],
    [0, 4, 8]
  ];
  // https://www.guitarland.com/MusicTheoryWithToneJS/PlayChords.html
  function makeChordArray(root, chordFormula, timeInterval) {
    var indexMIDI;
    var aChord = [];
    var timeAndChord = [];
    var timeSum = 0;
    var toneTime = "0";
    var chordArray = [];
    for (let i = 0; i < chordFormula.length; i++) {
      for (let j = 0; j < chordFormula[i].length; j++) {
        // add the root to each chord tone
        indexMIDI = chordFormula[i][j] + Number(root);
        // tranlate to a pitch/octave name
        aChord.push(MIDI_NUM_NAMES[indexMIDI]);
      }
      let j = 0;
      // create add time and chord together
      timeAndChord.push(toneTime);
      timeAndChord.push(aChord);
      chordArray.push(timeAndChord);
      // now calc the time value for next time
      timeSum += parseInt(timeInterval);
      toneTime = "0:" + timeSum;
      console.log(Tone.Time(toneTime));
      // clear the arrays;
      aChord = [];
      timeAndChord = [];
    }
    return chordArray;
  }
  var root; // let user choose
  var familyOfTriads = [
    [0, 4, 7],
    [0, 3, 7],
    [0, 3, 6],
    [0, 4, 8]
  ];
  var MAJOR_SCALE = [0, 2, 4, 5, 7, 9, 11, 12];
  var NATURAL_MINOR_SCALE = [0, 2, 3, 5, 7, 8, 10, 12];
  var HARMONIC_MINOR_SCALE = [0, 2, 3, 5, 7, 8, 11, 12];
  var MELODIC_MINOR_SCALE = [0, 2, 3, 5, 7, 9, 11, 12];
  function getScaleFormula() {
    console.log("getScaleFormula()");
    var scaleTypeMenu = document.getElementById("scaleType");
    var scaleType = scaleTypeMenu.options[scaleTypeMenu.selectedIndex].value;
    if (scaleType === "Major") {
      return MAJOR_SCALE;
    } else if (scaleType === "NaturalMinor") {
      return NATURAL_MINOR_SCALE;
    } else if (scaleType === "HarmonicMinor") {
      return HARMONIC_MINOR_SCALE;
    } else if (scaleType === "MelodicMinor") {
      return MELODIC_MINOR_SCALE;
    } else {
      return NATURAL_MINOR_SCALE;
    }
  }

  var romanNumeralToIndex = {
    I: 0,
    II: 1,
    III: 2,
    IV: 3,
    V: 4,
    VI: 5,
    VII: 6
  };
  function createDiatonicTriadFormula_OneOctave(scaleDegreeRoot, scale) {
    var useMajorV = document.myForm2.V_Major.checked;
    var oneChord = [];
    oneChord.push(scale[scaleDegreeRoot % 7]);
    if (useMajorV && scaleDegreeRoot == 4 && scale == NATURAL_MINOR_SCALE) {
      oneChord.push(scale[(scaleDegreeRoot + 2) % 7] + 1);
    } else {
      oneChord.push(scale[(scaleDegreeRoot + 2) % 7]);
    }
    oneChord.push(scale[(scaleDegreeRoot + 4) % 7]);
    return oneChord;
  }
  function playChordProgression() {
    //    console.log("playChordProgression");
    var rootMenu = document.getElementById("root2");
    root = rootMenu.options[rootMenu.selectedIndex].value;

    var chordMenu = document.getElementById("chordProgressions");
    var chordProgStr = chordMenu.options[chordMenu.selectedIndex].value;
    var chordProgArr = chordProgStr.split("-");
    var oneChord = [];
    var chordArray = [];
    var scale = getScaleFormula();

    for (let i = 0; i < chordProgArr.length; i++) {
      var index = romanNumeralToIndex[chordProgArr[i]];
      oneChord = createDiatonicTriadFormula_OneOctave(index, scale);
      chordArray.push(oneChord);
    }
    let myChords = makeChordArray(root, chordArray, "2");
    if (!piano) {
      piano = new Tone.PolySynth(Tone.Synth, {
        volume: -8,
        oscillator: {
          partials: [1, 2, 5]
        },
        portamento: 0.005
      }).toDestination();
    }

    console.log(myChords);
    var chordPart = new Tone.Part(function (time, chord) {
      piano.triggerAttackRelease(chord, "2n", time);
    }, myChords).start(Tone.Transport.nextSubdivision("1m"));

    chordPart.loop = true;
    chordPart.loopEnd = "2m";

    Tone.Transport.start(Tone.Transport.nextSubdivision("1m"));
    Tone.start(Tone.Transport.nextSubdivision("1m"));
  }
  // var button1 = document.getElementById("playChordsButton");
  // button1.onclick = playFamilyOfTriads;
</script>

<svelte:head>
  <link h rel="stylesheet" href="/index.css" />
  <!-- <link rel="icon" type="image/svg" href='favicon.png' /> -->
</svelte:head>

<main style="width: 50vw; margin: 100px auto">
  <div class="row">
    <div class="col">
      <h1 class="mb-4">Sampler</h1>
    </div>
    <div class="col right-col">
      <button on:click={Tone.Transport.stop()} class="btn btn-danger"
        >STOP ALL SOUNDS</button
      >
    </div>
  </div>
  <h3>{status}</h3>

  <div bind:this={downloadContainer} />
  <div class="row mb-3">
    <div class="col">
      <audio id="sample-recording" hidden controls />
      <button on:click={recorder.start()}>RECORD</button>
      <button on:click={stopPlay}>STOP</button>
      <div class="border w-50 my-3 p-2">
        <div id="tempo">
          Tempo: {Tone.Transport?.bpm?.value || tempo}
        </div>
        <input
          on:input={(e) =>
            Tone.Transport.bpm
              ? (Tone.Transport.bpm.value = e.target.value)
              : ""}
          id="tempo-slider"
          type="range"
          min="40"
          max="180"
          value="120"
        />
      </div>
    </div>
    <div class="col right-col">
      <div
        style="background-color: black; color: white"
        hidden
        class="sound-clip sound right-col"
        draggable="true"
        on:dragover={handleDragEnter}
        on:dragstart={handleDragStart}
        on:dragend={handleDragDrop}
        on:click={() => playSound()}
        on:keydown={() => playSound()}
      >
        CUSTOM SOUND
      </div>

      <audio id="audio-recording" hidden controls />
      <button on:click={startAudioRecording}> RECORD NEW SAMPLE </button>
    </div>
  </div>
  <div class="row mb-3 ">
    {#each timings as timing}
      <div
        draggable="true"
        on:dragover={handleDragEnter}
        on:dragstart={handleDragStart}
        on:dragend={handleDragDrop}
        style="cursor: grab;"
        class="col"
        id={timing}
      >
        <div class="timing">{timing}</div>
      </div>
    {/each}
  </div>
  <div class="row mb-3 ">
    {#each effects as effect}
      <div
        draggable="true"
        on:dragover={handleDragEnter}
        on:dragstart={handleDragStart}
        on:dragend={handleDragDrop}
        style="cursor: grab;"
        class="col"
        id={effect}
      >
        <div class="effect">{effect}</div>
      </div>
    {/each}
  </div>

  <div class="container-fluid">
    <div class="row mb-3">
      {#each sounds as sound, index}
        <div
          style="background-color: {colors[index]};"
          class="col-3 text-align-center"
        >
          <input
            style="float: left; height: 50%; margin-top:20% "
            orient="vertical"
            on:input={(e) => (players[index].volume.value = e.target.value)}
            type="range"
            min="-30"
            max="0"
            value="-12"
          />
          <div
            data-sound-url={sound.url}
            id={sound.name}
            name={index}
            on:dragover={handleDragEnter}
            on:click={() => playSound(index)}
            on:keydown={null}
            class="sound"
          >
            <img
              name={index}
              width="100%"
              alt={sound.name}
              src={sound.imageUrl}
            />
          </div>
          <div id="{sound.name}-holder" class="row" />
        </div>
      {/each}
    </div>
    <div class="row">
      <form name="myForm2">
        chord progression | <select
          id="chordProgressions"
          name="chordProgressions"
        >
          <option value="I-IV-V-I">I-IV-V-I</option>
          <option value="I-VII-VI-V">I-VII-VI-V</option>
          <option value="I-V-IV-V">I-V-IV-V</option>
          <option value="I-V-VI-IV">I-V-VI-IV</option>
          <option value="II-V-I-VI">II-V-I-VI</option>
          <option value="I-II-III-IV">I-II-III-IV</option>
          <option value="I-III-IV-I">I-III-IV-I</option>
          <option value="IV-I-V-VI">IV-I-V-VI</option>
          <option value="IV-V-I-IV">IV-V-I-IV</option>
        </select>

        <select name="root2" id="root2">
          <option value="57">A </option><option value="58">Bb </option><option
            value="59"
            >B
          </option><option value="60">C </option><option value="61"
            >C#
          </option><option value="61">Db </option><option value="62"
            >D
          </option><option value="63">Eb </option><option value="64"
            >E
          </option><option value="65">F </option><option value="66"
            >F#
          </option><option value="66">Gb </option><option value="67"
            >G
          </option><option value="68">Ab </option><option value="69"
            >A
          </option></select
        >
        <input
          type="button"
          class="playButton"
          VALUE="Play Chord Progression"
          id="playChordProgressionButton"
          on:click={playChordProgression}
        />
        <input type="button" class="stopButton" VALUE="Stop" />

        | tempo:<select
          name="tempo2"
          value={Tone.Transport?.bpm?.value?.toString()}
          on:change={(e) => (Tone.Transport.bpm.value = e.target.value)}
        >
          <option value="60">60</option>
          <option value="70">70</option>
          <option value="80">80</option>
          <option value="90">90</option>
          <option value="100">100</option>
          <option value="110">110</option>
          <option value="120">120</option>
          <option value="130">130</option>
          <option value="140">140</option>
          <option value="160">160</option>
          <option value="180">180</option>
          <option value="200">200</option>
        </select>

        Scale type:<select name="scaleType" id="scaleType">
          <option value="Major">Major</option>
          <option value="NaturalMinor">Natural Minor</option>
          <option value="HarmonicMinor">Harmonic Minor</option>
          <option value="MelodicMinor">Melodic Minor</option>
        </select>
        <label
          ><input type="checkbox" name="V_Major" id="V_Major" value="V_Major" />
          use major V on natural minor
        </label>
      </form>
    </div>
  </div>
</main>

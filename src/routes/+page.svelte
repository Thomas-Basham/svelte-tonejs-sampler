<script>
  import { onMount } from "svelte";
  import * as Tone from "tone";
  import {
    sounds,
    MIDI_NUM_NAMES,
    colors,
    timings,
    effects
  } from "../components/data";
  import Chords from "../components/Chords.svelte";
  export let currentTiming;
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
    <Chords {MIDI_NUM_NAMES} {Tone} {piano} />
  </div>
</main>

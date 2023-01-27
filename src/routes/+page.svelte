<script>
  // TODO: add chords and melodies? All of this code is working,
  //       it just doesn't really pair with the sampler well
  // import Chords from "../components/Chords.svelte";
  import { onMount } from "svelte";
  import About from "../components/About.svelte";
  import * as Tone from "tone";
  import {
    sounds,
    MIDI_NUM_NAMES,
    colors,
    timings,
    effects,
    SampleObject,
  } from "../components/data";
  export let currentTiming;
  let players;
  let analyzers = [];
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
  let customSounds = [];
  let recording = false;

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
      analyzers.push(analyser);

      return player;
    });
    //	PIANO
    piano = new Tone.PolySynth(Tone.Synth, {
      volume: -8,
      oscillator: {
        partials: [1, 2, 5],
      },
      portamento: 0.005,
    }).toDestination();
  });

  function playSound(index, loopPoint) {
    let player = players[index];
    let analyser = analyzers[index];

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
    const record = await recorder.stop();
    recording = false;
    // download the recording by creating an anchor element and blob url
    const url = URL.createObjectURL(record);
    const anchor = document.createElement("a");
    anchor.download = url;
    anchor.href = url;
    anchor.textContent = "DOWNLOAD";
    anchor.className = "download-link";
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
    e.target.className += " grabbing";
    console.log(e.target.className);
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
      // create blob from recording
      const blob = new Blob(chunks, { type: "audio/ogg; codecs=opus" });

      // reset chunks
      chunks = [];
      const audioURL = window.URL.createObjectURL(blob);

      let sampleObj = new SampleObject(
        audioURL,
        "CUSTOM SOUND",
        "https://cdn-icons-png.flaticon.com/128/865/865548.png"
      );
      sounds.push(sampleObj);

      let analyser = new Tone.Analyser("waveform", 128);
      let player = new Tone.Player(audioURL)
        .connect(recorder)
        .connect(analyser)
        .unsync()
        .toDestination();
      analyzers.push(analyser);
      players.push(player);
      customSounds = sounds.slice(12);
    };
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
      <button on:click={Tone.Transport.stop()} class="btn  btn-outline-danger"
        >STOP ALL SOUNDS</button
      >
    </div>
  </div>
  <h3>{status}</h3>

  <div class="download-container" bind:this={downloadContainer} />
  <div class="row mb-3">
    <div class="col">
      <audio id="sample-recording" hidden controls />
      <br />
      {#if !recording}
        <button
          class="record-btn"
          on:click={() => {
            recording = true;
            recorder.start();
          }}>RECORD</button
        >
      {/if}
      {#if recording}
        <button class="stop-btn" on:click={stopPlay}>STOP</button>
      {/if}
      <div class="tempo border my-3 p-2">
        <div id="tempo">
          Tempo: {(Tone.Transport?.bpm?.value &&
            Math.floor(Tone.Transport?.bpm?.value)) ||
            tempo}
        </div>
        <input
          on:input={(e) =>
            Tone.Transport.bpm
              ? (Tone.Transport.bpm.value = Math.floor(e.target.value))
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
      <div class="row mb-3">
        {#each customSounds as sound, index}
          <div
            style="background-color: {colors[index]};"
            class="col-3 sound-col"
            id="sound-col-{sound}"
            
            >
            <input
            style="float: left; width:100%"
            on:input={(e) =>
                (players[index + 12].volume.value = e.target.value)}
              type="range"
              min="-30"
              max="0"
              value="-12"
              />
              <div
              id={sound.name}
              data-sound-url={sound.url}
              name={index + 12}
              on:dragover={handleDragEnter}
              on:click={() => playSound(index + 12)}
              on:keydown={null}
              class="custom-sound"
            >
              <p style="text-align: center; margin: 0 auto" class="text-align-center">CUSTOM SOUND</p>
            </div>
            <div id="{sound.name}-holder" class="row" />
          </div>
        {/each}
      </div>

      <button class="record-btn" on:click={startAudioRecording}>
        RECORD NEW SAMPLE
      </button>
    </div>
  </div>
  <div class="row mb-3 ">
    {#each timings as timing}
      <div
        draggable="true"
        on:dragover={handleDragEnter}
        on:dragstart={handleDragStart}
        on:dragend={handleDragDrop}
        class="col grab"
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
          id="sound-col-{sound}"
          style="border:solid {colors[index]} 2px ;text-align:center;  "
          class="col-3 sound-col"
        >
          <input
            style="margin: 3px auto; "
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
              style="fill: {colors[index]}"
              name={index}
              width="40%"
              alt={sound.name}
              src={sound.imageUrl}
            />
          </div>
          <div id="{sound.name}-holder" class="row" />
        </div>
      {/each}
    </div>
    <About />
    <!-- <Chords {MIDI_NUM_NAMES} {Tone} {piano} /> -->
  </div>
</main>

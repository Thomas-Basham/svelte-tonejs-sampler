<script>
  import { onMount } from "svelte";
  import * as Tone from "tone";

  let sounds = [
    "https://cdn.pixabay.com/download/audio/2022/03/10/audio_e9a93ed562.mp3?filename=bassdrum-10-45967.mp3",
    "https://cdn.pixabay.com/download/audio/2021/08/04/audio_19faff7fe5.mp3?filename=elephant-trumpets-growls-6047.mp3",
    "https://cdn.pixabay.com/download/audio/2022/10/23/audio_b3ca30d553.mp3?filename=horse-123780.mp3",
    "https://cdn.pixabay.com/download/audio/2022/06/08/audio_98351be2c8.mp3?filename=duck-quack-112941.mp3",
    "https://cdn.pixabay.com/download/audio/2021/08/09/audio_ec2624b77c.mp3?filename=failure-drum-sound-effect-2-7184.mp3",
    "https://cdn.pixabay.com/download/audio/2022/01/18/audio_374cfeefe6.mp3?filename=punch-boxing-02wav-14897.mp3",
    "https://cdn.pixabay.com/download/audio/2022/11/02/audio_c266b117e9.mp3?filename=hit-drum-superclasher-cinematic-trailer-sound-effects-124759.mp3",
    "https://cdn.pixabay.com/download/audio/2022/01/18/audio_fb6e6bf2f6.mp3?filename=left-crashwav-14567.mp3",
    "https://cdn.pixabay.com/download/audio/2021/10/23/audio_6de1a03d51.mp3?filename=casio-sk-1-keyboard-synth-drum-pulse-001-9758.mp3",
    "https://cdn.pixabay.com/download/audio/2022/01/18/audio_c760f516ad.mp3?filename=hit-of-orchestral-cymbals-and-bass-drum-14471.mp3",
    "https://cdn.pixabay.com/download/audio/2022/03/19/audio_942d475509.mp3?filename=clap-2-95736.mp3",
    "https://tonejs.github.io/audio/berklee/gong_1.mp3"
  ];
  let colors = [
    "blue",
    "red",
    "green",
    "orange",
    "pink",
    "white",
    "purple",
    "yellow",
    "brown",
    "blue",
    "green",
    "orange",
    "white",
    "pink",
    "purple",
    "yellow",
    "brown",
    "pink",
    "blue"
  ];
  let timings = ["1m", "2n", "2t", "4n", "4t", "8n", "8t"];
  let currentTiming;
  let players;
  let recorder;
  let downloadContainer;
  let status = "";
  let currentSoundIndex;
  let mediaRecorder;
  let chunks = [];

  onMount(() => {
    // microphone recorder
    navigator.mediaDevices.getUserMedia({ audio: true }).then((stream) => {
      mediaRecorder = new MediaRecorder(stream);
      // mediaRecorder.start();
    });

    // Tone.js
    recorder = new Tone.Recorder();
    players = sounds.map((sound) => {
      let player = new Tone.Player(sound).connect(recorder).toDestination();
      return player;
    });
  });

  function playSound(index, loopPoint) {
    console.log(index);
    let player = players[index];
    if (loopPoint) {
      player.setLoopPoints(loopPoint);
      player.loop = true;
    }
    player.start();
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
    status = "You are dragging over the " + e.target.getAttribute("name");
    currentSoundIndex = e.target.getAttribute("name");
  }

  function handleDragStart(e) {
    status = "Dragging the element " + e.target.getAttribute("id");
    e.dataTransfer.dropEffect = "move";
    e.dataTransfer.setData("text", e.target.getAttribute("id"));
    if (timings.includes(e.target.textContent)) {
      currentTiming = e.target.textContent;
      console.log("TIMING", currentTiming);
    }
    // e.dataTransfer.setData("text", e.target.textContent);
    // console.log("DRAG START", e.dataTransfer.getData("text"))
  }

  function handleDragDrop(e) {
    e.preventDefault();
    console.log("DROPPED");
    // let element_id = e.dataTransfer.getData("text");
    console.log("current timing ", currentTiming);
    // sounds[currentSoundIndex] = element_id;
    // status = "You dropped " + element_id + " INTO " + currentSoundIndex;
    // players = sounds.map((sound) => {
    //   let player = new Tone.Player(sound).connect(recorder).toDestination();

    //   return player;
    // });
  }
  function startAudioRecording() {
    mediaRecorder.start();

    mediaRecorder.ondataavailable = (e) => {
      chunks.push(e.data);
    };
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
    };
  }
</script>

<svelte:head>
  <link h rel="stylesheet" href="/index.css" />
</svelte:head>

<main style="width: 50vw; margin: 100px auto">
  <h1 class="mb-4">Sampler</h1>
  <h3>{status}</h3>

  <div bind:this={downloadContainer} />
  <div class="row mb-3">
    <div class="col">
      <audio id="sample-recording" hidden controls />
      <button on:click={recorder.start()}>RECORD</button>
      <button on:click={stopPlay}>STOP</button>
    </div>
    <div class="col ">
      <div
        style="background-color: black; color: white"
        hidden
        class="sound-clip sound"
        draggable="true"
        on:dragover={handleDragEnter}
        on:dragstart={handleDragStart}
        on:dragend={handleDragDrop}
        on:click={() => playSound(index)}
      >
        CUSTOM SOUND
      </div>

      <audio id="audio-recording" hidden controls />
      <button on:click={startAudioRecording}> RECORD NEW SAMPLE </button>
      <button on:click={stopAudioRecording}> STOP RECORDING </button>
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
      >
        <div class="timing">{timing}</div>
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
          <div
            id={sound}
            name={index}
            on:dragover={handleDragEnter}
            on:click={() => playSound(index)}
            class="sound"
          >
            PLAY SOUND
          </div>
        </div>
      {/each}
    </div>
  </div>
</main>

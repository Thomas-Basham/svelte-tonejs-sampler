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
  let players;
  let recorder;
  let downloadContainer;
  let status = "";
  let currentID;

  onMount(() => {
    recorder = new Tone.Recorder();
    players = sounds.map((sound) => {
      return new Tone.Player(sound).connect(recorder).toDestination();
    });
  });

  function playSound(index) {
    console.log(index);
    players[index].start();
  }
  let stopPlay = async () => {
    // the recorded audio is returned as a blob
    const recording = await recorder.stop();

    // download the recording by creating an anchor element and blob url
    const url = URL.createObjectURL(recording);
    const anchor = document.createElement("a");
    anchor.download = "recording.mp3";
    anchor.href = url;
    anchor.textContent = "DOWNLOAD";
    downloadContainer.appendChild(anchor);

    // Show recording
    const audio = document.querySelector("audio");
    audio.hidden = false;
    audio.src = url;
    console.log(url);
  };

  function handleDragEnter(e) {
    status = "You are dragging over the " + e.target.getAttribute("name");
    currentID = e.target.getAttribute("name");
  }

  function handleDragStart(e) {
    status = "Dragging the element " + e.target.getAttribute("id");
    e.dataTransfer.dropEffect = "move";
    e.dataTransfer.setData("text", e.target.getAttribute("id"));
  }

  function handleDragDrop(e) {
    e.preventDefault();
    var element_id = e.dataTransfer.getData("text");
    console.log(currentID);
    sounds[currentID] = element_id;
    sounds.splice(currentID, 1, element_id);
    status = "You dropped " + element_id + " INTO " + currentID;
    players = sounds.map((sound) => {
      return new Tone.Player(sound).connect(recorder).toDestination();
    });
  }
</script>

<svelte:head>
  <link h rel="stylesheet" href="/index.css" />
</svelte:head>

<div style="width: 50vw; margin: 100px auto">
  <h1 class="mb-4">Sampler</h1>
  <h3>{status}</h3>
  <div bind:this={downloadContainer} />
  <audio hidden controls />
  <button on:click={recorder.start()}>RECORD</button>
  <button on:click={stopPlay}>STOP</button>
  <div class="container-fluid">
    <div class="row mb-3">
      {#each sounds as sound, index}
        <div style="background-color: {colors[index]};" class="col-md-3">
          <div
            id={sound}
            name={index}
            draggable="true"
            on:dragover={handleDragEnter}
            on:dragstart={handleDragStart}
            on:dragend={handleDragDrop}
            on:click={() => playSound(index)}
            class="sound"
          >
            PLAY SOUND
          </div>
        </div>
      {/each}
    </div>
  </div>
</div>

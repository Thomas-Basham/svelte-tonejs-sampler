<script>
  export let MIDI_NUM_NAMES;
  export let piano;
  export let Tone;
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

<div class="row">
  <form name="myForm2">
    chord progression | <select id="chordProgressions" name="chordProgressions">
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

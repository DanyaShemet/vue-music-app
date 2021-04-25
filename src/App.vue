<template>
  <div id="app">
    <header>
      <h1>My music</h1>
    </header>

    <main>

      <input type="range" min="0" :max="current.duration" v-model="currentTime" ref="range" @change="changeHandler" @input="inputHandler">




      <input type="range" min="0" :max="current.duration" v-model="currentTime" ref="range" @change="changeHandler"
             @input="inputHandler">

      <section class="player">
        {{ currentTimeConverted }} / {{ current.durationConverted }}
        <h2 class="song-title">{{ current.title }} -
          <span>{{ current.artist }}</span>

          {{ current.durationConverted }}
        </h2>
        <div class="controls">
          <button class="random" @click="random_off" v-if="isRandom">Random off</button>
          <button class="random" @click="random_onn" v-else>Random on</button>

          <button class="prev" @click="prev">Prev</button>
          <button class="play" v-if="!isPlaying" @click="resume">Play</button>
          <button class="pause" v-else @click="pause">Pause</button>
          <button class="next" @click="next">Next</button>
          <button class="repeat" @click="isRepeat = !isRepeat">Repeat</button>
        </div>
      </section>
      <section class="playlist">
        <h3>The playlist</h3>
        <button v-for="(song) in songs" :key="song.src"
                :class="(song.src === current.src) ? 'song playing' : 'song'"
                @click="(song.src === current.src) ? '' :  play(song)"

        >
          {{ song.title }} - {{ song.artist }}
        </button>
      </section>
    </main>
  </div>
</template>

<script>
import moment from "moment";
import {parseTime} from "@/utils/time.plugin";

export default {
  name: 'App',
  data() {
    return {
      current: {},
      isRepeat: false,
      isRandom: false,
      index: 0,
      currentTimeConverted: '00:00',
      currentTime: 0,
      value: 0,
      isPlaying: false,
      unShuffledSongs: [],
      songs: [
        {
          title: 'Her Mannelig',
          artist: 'Garmarna',
          src: require('./assets/music/Garmarna-Herr Mannelig.mp3'),
          durationConverted: null,
          duration: null,
          active: false
        },
        {
          title: 'Voluspa',
          artist: 'Duivelaspack',
          src: require('./assets/music/Duivelspack -Voluspa-Die Weissagung aus der Lieder-Edda.mp3'),
          durationConverted: null,
          duration: null,
          active: false
        },
        {
          title: 'Viking fight music',
          artist: 'Viking fight music',
          src: require('./assets/music/viking.mp3'),
          durationConverted: null,
          duration: null,
          active: false
        },
      ],
      player: new Audio()
    }
  },

  methods: {
    inputHandler() {
      this.player.currentTime = this.currentTime
      this.currentTimeConverted = `${parseTime(this.currentTime)}`
    },
    changeHandler() {
      this.player.currentTime = this.currentTime
      this.currentTimeConverted = `${parseTime(this.currentTime)}`
    },
    updateTime(e) {
      this.currentTime = e.target.currentTime
      this.currentTimeConverted = `${parseTime(this.currentTime)}`
    },
    play(song) {
      this.songs.forEach(el => el.active = false)
      this.songs[this.index].active = true
      this.index = this.songs.indexOf(song)
      this.player.currentTime = this.currentTime
      this.isPlaying = true;
      if (typeof song.src != "undefined") {
        this.current = song;
        this.player.src = this.current.src
      }

      // Получение длительности песни
      if (typeof song.durationConverted !== "string") {
        this.player.onloadeddata = (e) => {
          let duration = moment.duration(e.target.duration, "seconds")
          let min = duration.minutes()
          let sec = duration.seconds() < 10 ? '0' + duration.seconds() : duration.seconds()
          this.current.durationConverted = min + ":" + sec
          this.current.duration = Math.floor(e.target.duration)
        }
      }

      this.player.play();

      // Изменение времени
      this.player.ontimeupdate = (e) => {
        this.updateTime(e)
      }

      // Окончание песни
      this.player.onended = () => this.isRepeat ? this.repeat : this.next

    },


    random_onn() {
      // Получаем песню - Garmarna
      // Узнаем какая по счету она стоит в новом массиве
      // Ставим в this.index ее порядковый номер в массиве

      this.isRandom = true
      this.unShuffledSongs = this.unShuffledSongs.concat(this.songs)
      this.shuffle(this.songs)
      this.index = this.songs.findIndex(s => s.active)
    },

    random_off() {
      this.isRandom = false
      this.songs = this.unShuffledSongs
      this.unShuffledSongs = []
    },


    shuffle(array) {
      for (let i = array.length - 1; i > 0; i--) {
        let j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }
    },

    // Повтор песни включен
    repeat() {
      this.play(this.songs[this.index])
    },


    pause() {
      this.player.pause()
      this.isPlaying = false;
    },
    resume() {
      this.player.currentTime = this.currentTime
      this.player.play();
      this.isPlaying = true;

    },
    next() {

      if (this.index === this.songs.length - 1) {
        this.index = 0
      } else {
        this.index++
      }

      this.currentTime = 0
      this.play(this.songs[this.index])
    },
    prev() {
      if (this.index === 0) {
        this.index = this.songs.length - 1
      } else {
        this.index--;
      }

      this.play(this.songs[this.index])
    }
  },

  created() {

    this.songs[this.index].active = true

    this.current = this.songs[this.index]
    this.player.src = this.current.src

    // дубляж который меня бесит, сделать так что бы при входе первая композиция не была выбрана
    this.player.onloadeddata = (e) => {
      let duration = moment.duration(e.target.duration, "seconds")
      let min = duration.minutes()
      let sec = duration.seconds() < 10 ? '0' + duration.seconds() : duration.seconds()
      this.current.durationConverted = min + ":" + sec
      this.current.duration = Math.floor(e.target.duration)
    }
    this.player.ontimeupdate = (e) => {
      this.updateTime(e)
    }
    this.player.addEventListener('ended', function () {
      this.isRepeat ? this.repeat() : this.next()
    }.bind(this))


  }
}

</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
}

header {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 15px;
  background-color: #212121;
  color: #FFF;
}

main {
  width: 100%;
  max-width: 768px;
  margin: 0 auto;
  padding: 25px;
}

.song-title {
  color: #53565A;
  font-size: 32px;
  font-weight: 700;
  text-transform: uppercase;
  text-align: center;
}

.song-title span {
  font-weight: 400;
  font-style: italic;
}

.controls {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 30px 15px;
}

button {
  appearance: none;
  background: none;
  border: none;
  outline: none;
  cursor: pointer;
}


.play, .pause {
  font-size: 20px;
  font-weight: 700;
  padding: 15px 25px;
  margin: 0px 15px;
  border-radius: 8px;
  color: #FFF;
  background-color: #CC2E5D;
}

.next, .prev {
  font-size: 16px;
  font-weight: 700;
  padding: 10px 20px;
  margin: 0px 15px;
  border-radius: 6px;
  color: #FFF;
  background-color: #FF5858;
}

.playlist {
  padding: 0px 30px;
}

.playlist h3 {
  color: #212121;
  font-size: 28px;
  font-weight: 400;
  margin-bottom: 30px;
  text-align: center;
}

.playlist .song {
  display: block;
  width: 100%;
  padding: 15px;
  font-size: 20px;
  font-weight: 700;
  cursor: pointer;
}

.playlist .song:hover {
  color: #FF5858;
}

.playlist .song.playing {
  color: #FFF;
  background-image: linear-gradient(to right, #CC2E5D, #FF5858);
}

</style>
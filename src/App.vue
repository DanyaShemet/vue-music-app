<template>
  <div id="app">
    <main>

      <section class="player">
        <ImageSong :src="current.image"/>
        <TitleSong :current="current"/>
        <div class="middle_block_range range">
          <input type="range" min="0" :max="current.duration" v-model="currentTime" ref="range" @change="changeHandler"
                 @input="inputHandler">
          <div class="time_range">
            <p>{{ currentTimeConverted }}</p>
            <p>{{ current.durationConverted }}</p>
          </div>
        </div>
        <div class="bottom__block_controls controls">
          <ButtonRandom @random="random" :isRandom="isRandom"/>
          <ButtonPrevious @prev="prev"/>
          <ButtonPlayAndResume @resume="resume" :isPlaying="isPlaying"/>
          <ButtonNext @next="next"/>
          <ButtonRepeat :isRepeat="isRepeat" @repeatStatus="repeatStatus"/>
        </div>
        <div class="link__playlist">
          <button>Далее</button>
        </div>
      </section>

      <section class="playlist">
        <button v-for="(song) in songs" :key="song.src"
                :class="(song.src === current.src) ? 'song playing' : 'song'"
                @click="(song.src === current.src) ? '' :  play(song)">
          {{ song.title }} - {{ song.artist }}
        </button>
      </section>
    </main>
  </div>
</template>

<script>
import moment from "moment";
import {parseTime} from "@/utils/time.plugin";
import ImageSong from "@/components/ImageSong";
import TitleSong from "@/components/TitleSong";
import ButtonRandom from "@/components/ButtonRandom";
import ButtonPrevious from "@/components/ButtonPrevious";
import ButtonPlayAndResume from "@/components/ButtonPlayAndPause";
import ButtonNext from "@/components/ButtonNext";
import ButtonRepeat from "@/components/ButtonRepeat";

export default {
  name: 'App',
  components: {ButtonRepeat, ButtonNext, ButtonPlayAndResume, ButtonPrevious, ButtonRandom, TitleSong, ImageSong},
  data() {
    return {
      current: {},
      isRepeat: false,
      isRandom: false,
      index: 0,
      currentTimeConverted: '0:00',
      currentTime: 0,
      value: 0,
      isPlaying: false,
      unShuffledSongs: [],
      songs: [
        {
          title: 'Her Mannelig',
          artist: 'Garmarna',
          src: require('./assets/music/Garmarna-Herr Mannelig.mp3'),
          image: require('./assets/image/garmarna.jpg'),
          durationConverted: null,
          duration: null,
          active: false
        },
        {
          title: 'Voluspa',
          artist: 'Duivelaspack',
          src: require('./assets/music/Duivelspack -Voluspa-Die Weissagung aus der Lieder-Edda.mp3'),
          image: require('./assets/image/duivelaspack.jpg'),
          durationConverted: null,
          duration: null,
          active: false
        },
        {
          title: 'Viking fight music',
          artist: 'Viking fight music',
          src: require('./assets/music/viking.mp3'),
          image: require('./assets/image/vikingfight.jpg'),
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
      console.log('input')
      this.player.currentTime = this.currentTime
      this.currentTimeConverted = `${parseTime(this.currentTime)}`
    },
    changeHandler() {
      console.log('change')
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
      // this.index = this.songs.indexOf(song)
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

    random() {
      this.isRandom = !this.isRandom
      if (this.isRandom) {
        this.unShuffledSongs = this.unShuffledSongs.concat(this.songs)
        this.shuffle(this.songs)
        this.index = this.songs.findIndex(s => s.active)
      } else {
        this.songs = this.unShuffledSongs
        this.index = this.songs.findIndex(s => s.active)
        this.unShuffledSongs = []
      }
    },


    shuffle(array) {
      for (let i = array.length - 1; i > 0; i--) {
        let j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }
    },

    // Повтор песни включен
    repeatStatus() {
      this.isRepeat = !this.isRepeat
    },
    repeat() {
      this.play(this.songs[this.index])
    },

    resume() {
      if (this.isPlaying) {
        this.player.pause()
        this.isPlaying = false;
      } else {
        this.player.currentTime = this.currentTime
        this.player.play();
        this.isPlaying = true;
      }


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

    // дубляж который меня бесит
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

img {
  max-width: 100%;
}

body {
  font-family: 'Sarala', sans-serif;
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
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 0 auto;
}


.controls {
  display: flex;
  justify-content: center;
  align-items: center;
}

button {
  appearance: none;
  background: none;
  border: none;
  outline: none;
  cursor: pointer;
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

.artist__img {
  position: absolute;
  left: 50%;
  top: 50%;
  margin-left: -170px;
  margin-top: -170px;
  z-index: 100;
  border-radius: 50%;
  width: 340px;
}

.top_block_img {
  position: relative;
  height: 418px;
}

.background__artist_img {
  opacity: .2;
  height: 100%;
  width: 100%;
  border-radius: 10px 10px 0 0;
}

.player {
  width: 490px;
  border: 1px solid #000;
  border-radius: 10px;
}

.middle_block_title {
  margin-bottom: 10px;
  margin-top: 10px;
}

.middle_block_title h2 {
  color: #000;
  font-size: 36px;
  text-align: center;
  font-weight: 700;
  margin-top: 0;
  padding: 0;
  margin-bottom: -10px;
}

.middle_block_title p {
  color: #AAAAAA;
  text-align: center;
  font-weight: 400;
  font-size: 18px;
}


.middle_block_range {
  padding: 0 20px;
}

.middle_block_range input[type=range] {
  width: 100%;
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  outline: none;
  overflow: hidden;

}

.middle_block_range input[type=range]::-moz-range-track {
  height: 2px;
  background-color: #C0C0C0;
}

.middle_block_range input[type=range]::-moz-range-thumb {
  height: 9px;
  width: 9px;
  background-color: #000;
  border: 0;
  border-radius: 50%;
  cursor: pointer;
  padding: 0px;
  margin: 0px;
}

.middle_block_range input[type="range"]::-moz-range-progress {
  background-color: #000;
}

.middle_block_range input[type=range]::-webkit-slider-runnable-track {
  height: 2px;
  background-color: #C0C0C0;
}

.middle_block_range input[type=range]::-webkit-slider-thumb {
  background: #000;
  border: 0;
  border-radius: 10px/100%;
  cursor: pointer;
  width: 9px;
  height: 9px;
  box-shadow: -400px 0px 0px 400px #000;
  -webkit-appearance: none;
  margin-top: -4px;
}

.time_range {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: -5px;
}

.time_range p {
  font-size: 16px;
  color: #C0C0C0;
}

.repeat.active svg path {
  fill: #000;
}

.random.active svg path {
  fill: #000;
}

svg {
  pointer-events: none;
}


.bottom__block_controls {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 20px;
  margin-bottom: 10px;
}


.bottom__block_controls button {
  height: 70px;
  width: 70px;
  padding: 10px;
  border-radius: 50%;
  transition: .3s all;
  background-color: transparent;
  display: flex;
  align-items: center;
  justify-content: center;
}

.bottom__block_controls button:hover {
  background-color: #F0F0F0;
}

.bottom__block_controls button.nothover {
  height: auto;
  width: auto;
}

.bottom__block_controls button.nothover:hover {
  background-color: transparent;
}

.bottom__block_controls button.nothover:hover svg path {
  fill: #000;
}

.bottom__block_controls .main-btn {
  height: 98px;
  width: 98px;
  border-radius: 50%;
  background-color: #F0F0F0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.bottom__block_controls .main-btn:hover {
  background-color: #d7d7d7;
}

.bottom__block_controls .play svg {
  margin-left: 5px;
}

.link__playlist {
  text-align: center;
}

.link__playlist button {
  color: #C0C0C0;
  border-bottom: 1px solid #C0C0C0;
  font-size: 16px;
  margin-bottom: 5px;
}


</style>
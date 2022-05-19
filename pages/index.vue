<template>
  <div
    class="
      !bg-gradient-to-r
      !from-red-500
      !via-yellow-500
      !to-green-500
      backdrop-brightness-100 backdrop-blur-150
    "
  >
    <div class="player">
      <center>
        <div class="dashboard">
          <header>
            <h4>Now playing:</h4>
            <h2 v-if="isActive == false">Undefined</h2>
            <h2 v-else class="songname">{{ songname }}</h2>
          </header>
          <div class="cd">
            <div class="cd-thumb">
              <img
                class="cd-thumb-image"
                src="https://thumbs.dreamstime.com/z/error-page-not-found-vector-vinyl-music-broken-graphic-error-page-not-found-vector-vinyl-music-broken-graphic-background-156624909.jpg"
              />
            </div>
          </div>
          <div class="control">
            <div v-if="isRepeat == 0" class="btn btn-repeat" @click="repeat()">
              <Repeat />
            </div>
            <div v-else class="btn btn-repeat" @click="repeat()">
              <Repeat2 />
            </div>
            <div class="btn btn-prev" @click="prev()">
              <Prev />
            </div>
            <div
              v-if="isPlaying == 0 && isActive == false"
              class="btn btn-toggle-play border p-2 bg-red-500"
            >
              <Unplay />
            </div>
            <div
              v-else-if="isPlaying == 0 && isActive == true"
              class="btn btn-toggle-play border p-2 bg-red-500"
              @click="play()"
            >
              <Play />
            </div>
            <div
              v-else
              class="btn btn-toggle-play border p-2 bg-red-500"
              @click="pause()"
            >
              <Pause />
            </div>
            <div class="btn btn-next" @click="next()">
              <Next />
            </div>
            <div v-if="isRandom == 0" class="btn btn-random" @click="random()">
              <Random />
            </div>
            <div v-else class="btn btn-random" @click="random()">
              <Random2 />
            </div>
          </div>
          <input
            id="progress"
            class="progress"
            type="range"
            value="0"
            step="1"
            min="0"
            max="100"
          />
          <div class="inline-flex flex-cols-2 w-80">
            <Fullvol v-if="vol > 0.66" />
            <Medvol v-else-if="0.66 >= vol && vol > 0.33" />
            <Smallvol v-else-if="0.33 >= vol && vol > 0" />
            <Mute v-else />
            <input
              id="slide"
              type="range"
              value="100"
              step="1"
              min="0"
              max="100"
              @input="changeVol($event)"
              class="w-full"
            />
          </div>
          <audio id="audio" src=""></audio>
        </div>
        <div class="playlist">
          <div id="songlist" v-for="(m, index) in music" :key="index">
            <div
              v-if="m !== null"
              :data-index="index"
              class="song"
              @click="loadCurrentSong(index)"
              style="
                display: flex;
                align-items: center;
                margin-bottom: 12px;
                background-color: #fff;
                padding: 8px 16px;
                border-radius: 5px;
                box-shadow: 0 2px 3px rgb(0 0 0 / 10%);
                cursor: pointer;
              "
            >
              <img
                v-if="
                  m.artists == null ||
                  m.artists.data[0] == null ||
                  m.artists.data[0].avatar == null
                "
                style="
                  width: 46px;
                  height: 46px;
                  background-repeat: no-repeat;
                  background-size: cover;
                  background-position: center;
                  border-radius: 50%;
                  margin: 0 8px;
                "
                class="thumb"
                src="https://thumbs.dreamstime.com/z/error-page-not-found-vector-vinyl-music-broken-graphic-error-page-not-found-vector-vinyl-music-broken-graphic-background-156624909.jpg"
              />
              <img
                v-else
                class="thumb"
                style="
                  width: 46px;
                  height: 46px;
                  background-repeat: no-repeat;
                  background-size: cover;
                  background-position: center;
                  border-radius: 50%;
                  margin: 0 8px;
                "
                :src="m.artists.data[0].avatar.url"
              />
              <div class="body" style="flex: 1; padding: 0 16px">
                <h3 class="title" style="margin-bottom: 4px; font-size: 18px">
                  {{ m.title }}
                </h3>
                <p
                  class="author"
                  style="font-size: 12px; color: #999"
                  v-html="m.content"
                ></p>
              </div>
              <div
                class="option"
                style="padding: 16px 8px; color: #999; font-size: 18px"
              >
                <ThreeDots />
              </div>
            </div>
          </div>
        </div>
      </center>
    </div>
  </div>
</template>
<script>
import axios from 'axios'
export default {
  data() {
    return {
      isActive: false,
      currentIndex: 0,
      isTimeUpdate: 0,
      isRepeat: 0,
      isPlaying: 0,
      isRandom: 0,
      music: [],
      songname: '',
      $: null,
      audio: null,
      progr: null,
      cdThumbAnimate: null,
      cdthumbimg: null,
      currentTime: 0,
      vol: 1,
      track: [],
    }
  },
  created() {
    while (this.track.length < 20) {
      var value = Math.floor(Math.random() * 8500) + 1
      if (this.track.indexOf(value) === -1) {
        this.track.push(value)
      }
    }
    for (var i = 0; i < 20; i++) {
      axios
        .get('https://api2.bcdcnt.net/graphql', {
          params: {
            query:
              'query($id: ID!) {\n song(id: $id) {\nid\ntitle\nsubtitle\nslug\ncontent\nviews\ndownloads\nfile_type\nlyric_type\nstatus\nyear\nrecord_year\nallow_comment\ncreated_at\nartists(first: 10, orderBy: [{column: "song_artist.order", order: ASC}]) {\ndata {\nid\ntitle\nslug\navatar {\nurl\n}\n}\n}\ncomposers(first: 10, orderBy: [{column: "song_composer.order", order: ASC}]) { \n data { \n id\n title\n slug\n }\n }\n poets(first: 10) {\n data {\nid\n  title\n slug\n  }\n }\n thumbnail {\n url\n user {\n id\n username\navatar {\nurl\n}\n}\n}\nfile {\nvideo_url\naudio_url\n }\n uploader {\n id\n username\n avatar {\nurl\n}\n}\ndocuments(first: 10) {\ndata {\nid\ntitle\nslug\nthumbnail {\n url\n}\n}\n}\n}\n}',
            variables: { id: `${this.track[i]}` },
          },
        })
        .then((res) => {
          this.music = this.music.concat(res.data.data.song)
        })
    }
  },
  mounted() {
    this.$ = document.querySelector.bind(document)
    this.audio = this.$('#audio')
    this.progr = this.$('#progress')
    this.cdthumbimg = this.$('.cd-thumb-image')
    const audio2 = this.$('#audio')
    const progr2 = this.$('#progress')
    const cdthumbimg2 = this.$('.cd-thumb-image')
    const cdthumb = this.$('.cd-thumb')
    const cdWidth = cdthumbimg2.offsetWidth
    document.onscroll = function () {
      const scrollTop = window.scrollY || document.documentElement.scrollTop
      const newCdWidth = cdWidth - scrollTop
      cdthumbimg2.style.width = newCdWidth > 0 ? newCdWidth + 'px' : 0
      cdthumbimg2.style.height = newCdWidth > 0 ? newCdWidth + 'px' : 0
      cdthumbimg2.style.opacity = newCdWidth / cdWidth
    }
    this.cdThumbAnimate = cdthumb.animate([{ transform: 'rotate(360deg)' }], {
      duration: 25000,
      iterations: Infinity,
    })
    audio2.ontimeupdate = function () {
      if (audio2.duration) {
        const progressPercent = Math.floor(
          (audio2.currentTime / audio2.duration) * 100
        )
        progr2.value = progressPercent
      }
    }
    progr2.oninput = function (e) {
      const seekTime = (audio2.duration / 100) * e.target.value
      audio2.currentTime = seekTime
    }
    let _this = this
    this.cdThumbAnimate.pause()
    this.audio.onended = function () {
      if (_this.isRandom == 1 && _this.isRepeat == 1) {
        _this.songname = _this.music[_this.currentIndex].title
        _this.audio.src = _this.music[_this.currentIndex].file.audio_url
        _this.audio.play()
        if (_this.music[_this.currentIndex].thumbnail != null) {
          _this.cdthumbimg.src = _this.music[_this.currentIndex].thumbnail.url
        } else {
          _this.cdthumbimg.src =
            'https://thumbs.dreamstime.com/z/error-page-not-found-vector-vinyl-music-broken-graphic-error-page-not-found-vector-vinyl-music-broken-graphic-background-156624909.jpg'
        }
      } else if (_this.isRandom == 1 && _this.isRepeat == 0) {
        var x = Math.floor(Math.random() * 19) + 1
        while (_this.music[x].title == null || _this.music[x] == null) {
          x = Math.floor(Math.random() * 19) + 1
        }
        _this.songname = _this.music[x].title
        _this.audio.src = _this.music[x].file.audio_url
        _this.audio.play()
        if (_this.music[x].thumbnail != null) {
          _this.cdthumbimg.src = _this.music[x].thumbnail.url
        } else {
          _this.cdthumbimg.src =
            'https://thumbs.dreamstime.com/z/error-page-not-found-vector-vinyl-music-broken-graphic-error-page-not-found-vector-vinyl-music-broken-graphic-background-156624909.jpg'
        }
      } else if (_this.isRandom == 0 && _this.isRepeat == 1) {
        _this.songname = _this.music[_this.currentIndex].title
        _this.audio.src = _this.music[_this.currentIndex].file.audio_url
        _this.audio.play()
        if (_this.music[_this.currentIndex].thumbnail != null) {
          _this.cdthumbimg.src = _this.music[_this.currentIndex].thumbnail.url
        } else {
          _this.cdthumbimg.src =
            'https://thumbs.dreamstime.com/z/error-page-not-found-vector-vinyl-music-broken-graphic-error-page-not-found-vector-vinyl-music-broken-graphic-background-156624909.jpg'
        }
      } else {
        if (_this.currentIndex == 19) {
          _this.currentIndex = 0
        } else {
          _this.currentIndex = _this.currentIndex + 1
        }
        while (
          _this.music[_this.currentIndex].title == null ||
          _this.music[_this.currentIndex] == null
        ) {
          if (_this.currentIndex == 19) {
            _this.currentIndex = 0
          } else {
            _this.currentIndex = _this.currentIndex + 1
          }
        }
        _this.songname = _this.music[_this.currentIndex].title
        _this.audio.src = _this.music[_this.currentIndex].file.audio_url
        _this.audio.play()
        if (_this.music[_this.currentIndex].thumbnail != null) {
          _this.cdthumbimg.src = _this.music[_this.currentIndex].thumbnail.url
        } else {
          _this.cdthumbimg.src =
            'https://thumbs.dreamstime.com/z/error-page-not-found-vector-vinyl-music-broken-graphic-error-page-not-found-vector-vinyl-music-broken-graphic-background-156624909.jpg'
        }
      }
    }
  },
  methods: {
    changeVol(event) {
      this.$ = document.querySelector.bind(document)
      this.audio = this.$('#audio')
      this.audio.volume = event.target.value / 100
      this.vol = this.audio.volume
    },
    play() {
      this.isPlaying = 1
      this.audio.play()
      this.cdThumbAnimate.play()
    },
    pause() {
      this.isPlaying = 0
      this.audio.pause()
      this.cdThumbAnimate.pause()
    },
    prev() {
      this.isPlaying = 1
      this.isActive = true
      this.audio = this.$('#audio')
      if (this.isRandom == 1) {
        var x = Math.floor(Math.random() * 19) + 1
        while (
          this.music[x].title == null ||
          this.music[x] == null ||
          this.music == null
        ) {
          x = Math.floor(Math.random() * 19) + 1
        }
        this.songname = this.music[x].title
        this.audio.src = this.music[x].file.audio_url
        this.audio.play()
        if (this.music[x].thumbnail != null) {
          this.cdthumbimg.src = this.music[x].thumbnail.url
        } else {
          this.cdthumbimg.src =
            'https://thumbs.dreamstime.com/z/error-page-not-found-vector-vinyl-music-broken-graphic-error-page-not-found-vector-vinyl-music-broken-graphic-background-156624909.jpg'
        }
      } else {
        if (this.currentIndex == 0) {
          this.currentIndex = 19
        } else {
          this.currentIndex = this.currentIndex - 1
        }
        while (
          this.music[this.currentIndex].title == null ||
          this.music[this.currentIndex] == null ||
          this.music == null
        ) {
          if (this.currentIndex == 0) {
            this.currentIndex = 19
          } else {
            this.currentIndex = this.currentIndex - 1
          }
        }
        this.songname = this.music[this.currentIndex].title
        this.audio.src = this.music[this.currentIndex].file.audio_url
        this.audio.play()
        if (this.music[this.currentIndex].thumbnail != null) {
          this.cdthumbimg.src = this.music[this.currentIndex].thumbnail.url
        } else {
          this.cdthumbimg.src =
            'https://thumbs.dreamstime.com/z/error-page-not-found-vector-vinyl-music-broken-graphic-error-page-not-found-vector-vinyl-music-broken-graphic-background-156624909.jpg'
        }
      }
    },
    next() {
      this.isActive = true
      this.isPlaying = 1
      this.audio = this.$('#audio')
      if (this.isRandom == 1) {
        var x = Math.floor(Math.random() * 19) + 1
        while (
          this.music[x].title == null ||
          this.music[x] == null ||
          this.music == null
        ) {
          x = Math.floor(Math.random() * 19) + 1
        }
        this.songname = this.music[x].title
        this.audio.src = this.music[x].file.audio_url
        this.audio.play()
        if (this.music[x].thumbnail != null) {
          this.cdthumbimg.src = this.music[x].thumbnail.url
        } else {
          this.cdthumbimg.src =
            'https://thumbs.dreamstime.com/z/error-page-not-found-vector-vinyl-music-broken-graphic-error-page-not-found-vector-vinyl-music-broken-graphic-background-156624909.jpg'
        }
      } else {
        if (this.currentIndex == 19) {
          this.currentIndex = 0
        } else {
          this.currentIndex = this.currentIndex + 1
        }
        while (
          this.music[this.currentIndex].title == null ||
          this.music[this.currentIndex] == null ||
          this.music == null
        ) {
          if (this.currentIndex == 19) {
            this.currentIndex = 0
          } else {
            this.currentIndex = this.currentIndex + 1
          }
        }
        this.songname = this.music[this.currentIndex].title
        this.audio.src = this.music[this.currentIndex].file.audio_url
        this.audio.play()
        if (this.music[this.currentIndex].thumbnail != null) {
          this.cdthumbimg.src = this.music[this.currentIndex].thumbnail.url
        } else {
          this.cdthumbimg.src =
            'https://thumbs.dreamstime.com/z/error-page-not-found-vector-vinyl-music-broken-graphic-error-page-not-found-vector-vinyl-music-broken-graphic-background-156624909.jpg'
        }
      }
    },
    random() {
      if (this.isRandom == 0) {
        this.isRandom = 1
      } else {
        this.isRandom = 0
      }
    },
    repeat() {
      if (this.isRepeat == 0) {
        this.isRepeat = 1
      } else {
        this.isRepeat = 0
      }
    },
    loadCurrentSong(x) {
      this.isActive = true
      this.isPlaying = 1
      this.currentIndex = x
      this.cdThumbAnimate.play()
      this.audio = this.$('#audio')
      this.cdthumbimg = this.$('.cd-thumb-image')
      this.songname = this.music[x].title
      if (this.music[x].file != null) {
        this.audio.src = this.music[x].file.audio_url
        this.audio.play()
      }
      if (this.music[x].thumbnail != null) {
        this.cdthumbimg.src = this.music[x].thumbnail.url
      } else {
        this.cdthumbimg.src =
          'https://thumbs.dreamstime.com/z/error-page-not-found-vector-vinyl-music-broken-graphic-error-page-not-found-vector-vinyl-music-broken-graphic-background-156624909.jpg'
      }
    },
  },
}
</script>
<style>
.player {
  position: relative;
  max-width: 480px;
  margin: 0 auto;
}
.player .icon-pause {
  display: none;
}
.player.playing .icon-pause {
  display: inline-block;
}
.player.playing .icon-play {
  display: none;
}
.dashboard {
  padding: 16px 16px 14px;
  position: fixed;
  top: 0;
  width: 100%;
  max-width: 480px;
  background-color: #fff;
  border-bottom: 1px solid #ebebeb;
}
header {
  text-align: center;
  margin-bottom: 10px;
}
header h4 {
  font-size: 12px;
}

header h2 {
  font-size: 20px;
}
.songname {
  font-family: 'Arial';
}

.cd {
  display: block;
  margin: auto;
  width: 200px;
}
.cd-thumb {
  padding-top: 0px;
}

.cd-thumb-image {
  height: 200px;
  padding-top: 0px;
  border-radius: 50%;
  background-repeat: no-repeat;
  background-size: cover;
  background-position: center;
}

.control {
  display: flex;
  align-items: center;
  justify-content: space-around;
  padding: 20px 0 10px 0;
}
.control .btn {
  color: #666;
  padding: 18px;
  font-size: 18px;
}

.control .btn-toggle-play {
  width: 55px;
  height: 55px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
}
.progress {
  width: 100%;
  overflow: hidden;
  -webkit-appearance: none;
  height: 6px;
  background: #d3d3d3;
  outline: none;
  opacity: 0.7;
  -webkit-transition: 0.2s;
  transition: opacity 0.2s;
}
.progress:hover::-webkit-slider-thumb {
  background: #ec1f55;
  box-shadow: -500px 0 0 500px var(--primary-color);
}
.progress::-webkit-slider-thumb {
  -webkit-appearance: none;
  cursor: pointer;
  width: 12px;
  height: 6px;
  background: #ec1f55;
  box-shadow: -500px 0 0 500px var(--primary-color);
}
.playlist {
  padding: 12px;
  box-shadow: rgba(100, 100, 111, 0.2) 0px 7px 29px 0px;
  padding-top: 420px;
}
.song:active {
  opacity: 0.8;
}
</style>
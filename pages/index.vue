<template>
  <div class="!bg-gradient-to-r !from-red-500 !via-yellow-500 !to-green-500">
    <div class="player">
      <center>
        <div
          class="
            dashboard
            rounded-xl
            !bg-white/30
            backdrop-filter backdrop-blur-md
          "
        >
          <header class="text-gray-700 font-mono font-extrabold">
            <div>
              <h4
                class="
                  !text-md
                  w-auto
                  bg-gradient-to-r
                  from-green-500
                  via-blue-700
                  to-purple-900
                  text-transparent
                  bg-clip-text
                "
              >
                Now playing:
              </h4>
              <h2
                class="
                  !text-xl
                  w-auto
                  bg-gradient-to-r
                  from-green-500
                  via-blue-700
                  to-purple-900
                  text-transparent
                  bg-clip-text
                "
                v-if="isActive == false"
              >
                Undefined
              </h2>
              <h2
                class="
                  songname
                  !text-xl
                  w-auto
                  bg-gradient-to-r
                  from-green-500
                  via-blue-700
                  to-purple-900
                  text-transparent
                  bg-clip-text
                "
                v-else
              >
                {{ songname }}
              </h2>
            </div>
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
            <div
              v-if="isRepeat == 0"
              class="btn btn-repeat cursor-pointer"
              @click="repeat()"
            >
              <Repeat />
            </div>
            <div v-else class="btn btn-repeat cursor-pointer" @click="repeat()">
              <Repeat2 />
            </div>
            <div class="btn btn-prev cursor-pointer" @click="prev()">
              <Prev />
            </div>
            <div
              v-if="isPlaying == 0 && isActive == false"
              class="btn btn-toggle-play border p-2 bg-red-500 cursor-pointer"
            >
              <Unplay />
            </div>
            <div
              v-else-if="isPlaying == 0 && isActive == true"
              class="btn btn-toggle-play border p-2 bg-red-500 cursor-pointer"
              @click="play()"
            >
              <Play />
            </div>
            <div
              v-else
              class="btn btn-toggle-play border p-2 bg-red-500 cursor-pointer"
              @click="pause()"
            >
              <Pause />
            </div>
            <div class="btn btn-next cursor-pointer" @click="next()">
              <Next />
            </div>
            <div
              v-if="isRandom == 0"
              class="btn btn-random cursor-pointer"
              @click="random()"
            >
              <Random />
            </div>
            <div v-else class="btn btn-random cursor-pointer" @click="random()">
              <Random2 />
            </div>
          </div>
          <input
            id="progress"
            class="progress cursor-pointer"
            type="range"
            value="0"
            step="1"
            min="0"
            max="100"
          />
          <div
            class="inline-flex flex-cols-5 w-300px flex-auto"
            style="height: 30px"
          >
            <Fullvol v-if="vol > 66" />
            <Medvol v-else-if="66 >= vol && vol > 33" />
            <Smallvol v-else-if="33 >= vol && vol > 0" />
            <Mute v-else />
            <div>&nbsp;&nbsp;</div>
            <el-slider
              id="slide"
              v-model="vol"
              @input="changeVol($event)"
              class="w-250px cursor-pointer"
            ></el-slider>
            <div>&nbsp;&nbsp;&nbsp;&nbsp;</div>
            <div
              v-if="isActive == true"
              class="cursor-pointer"
              @click="download()"
            >
              <Download />
            </div>
            <div v-else class="cursor-pointer"><Undown /></div>
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
      amount: 20,
      isActive: false,
      currentIndex: 0,
      isTimeUpdate: 0,
      isRepeat: 0,
      isPlaying: 0,
      isRandom: 0,
      music: [],
      songname: '',
      audio: null,
      progr: null,
      cdthumb: null,
      cdThumbAnimate: null,
      cdthumbimg: null,
      currentTime: 0,
      vol: 100,
      track: [],
      dash: null,
    }
  },
  created() {
    while (this.track.length < this.amount) {
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
    window.addEventListener('scroll', this.handleScroll())
    this.audio = this.$el.querySelector('#audio')
    this.progr = this.$el.querySelector('#progress')
    this.cdthumbimg = this.$el.querySelector('.cd-thumb-image')
    this.cdthumb = this.$el.querySelector('.cd-thumb')
    this.cdThumbAnimate = this.cdthumb.animate(
      [{ transform: 'rotate(360deg)' }],
      {
        duration: 25000,
        iterations: Infinity,
      }
    )
    let _this = this
    this.cdThumbAnimate.pause()
    this.audio.ontimeupdate = function () {
      if (_this.audio.duration) {
        const progressPercent = Math.floor(
          (_this.audio.currentTime / _this.audio.duration) * 100
        )
        _this.progr.value = progressPercent
      }
    }
    this.progr.oninput = function (e) {
      const seekTime = (_this.audio.duration / 100) * e.target.value
      _this.audio.currentTime = seekTime
    }
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
        var x = Math.floor(Math.random() * (_this.amount - 1)) + 1
        while (_this.music[x].title == null || _this.music[x] == null) {
          x = Math.floor(Math.random() * (_this.amount - 1)) + 1
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
        if (_this.currentIndex == _this.amount - 1) {
          _this.currentIndex = 0
        } else {
          _this.currentIndex = _this.currentIndex + 1
        }
        while (
          _this.music[_this.currentIndex].title == null ||
          _this.music[_this.currentIndex] == null
        ) {
          if (_this.currentIndex == _this.amount - 1) {
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
  destroyed() {
    window.removeEventListener('scroll', this.handleScroll())
  },
  methods: {
    handleScroll() {
      this.dash = this.$el.querySelector('.dashboard')
      const cdthumbimg2 = this.$el.querySelector('.cd-thumb-image')
      const cdWidth = cdthumbimg2.offsetWidth
      document.onscroll = function () {
        const scrollTop = window.scrollY || document.documentElement.scrollTop
        const newCdWidth = cdWidth - scrollTop
        cdthumbimg2.style.width = newCdWidth > 0 ? newCdWidth + 'px' : 0
        cdthumbimg2.style.height = newCdWidth > 0 ? newCdWidth + 'px' : 0
        cdthumbimg2.style.opacity = newCdWidth / cdWidth
      }
    },
    changeVol() {
      this.audio = this.$el.querySelector('#audio')
      this.audio.volume = this.vol / 100
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
      this.cdThumbAnimate.play()
      this.audio = this.$el.querySelector('#audio')
      if (this.isRandom == 1) {
        var x = Math.floor(Math.random() * (this.amount - 1)) + 1
        while (
          this.music[x].title == null ||
          this.music[x] == null ||
          this.music == null
        ) {
          x = Math.floor(Math.random() * (this.amount - 1)) + 1
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
          this.currentIndex = this.amount - 1
        } else {
          this.currentIndex = this.currentIndex - 1
        }
        while (
          this.music[this.currentIndex].title == null ||
          this.music[this.currentIndex] == null ||
          this.music == null
        ) {
          if (this.currentIndex == 0) {
            this.currentIndex = this.amount - 1
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
      this.cdThumbAnimate.play()
      this.audio = this.$el.querySelector('#audio')
      if (this.isRandom == 1) {
        var x = Math.floor(Math.random() * this.amount) + 1
        while (
          this.music[x].title == null ||
          this.music[x] == null ||
          this.music == null
        ) {
          x = Math.floor(Math.random() * this.amount) + 1
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
      this.audio = this.$el.querySelector('#audio')
      this.cdthumbimg = this.$el.querySelector('.cd-thumb-image')
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
    download() {
      fetch(`${this.music[this.currentIndex].file.audio_url}`)
        .then((response) => response.blob())
        .then((blob) => {
          const link = document.createElement('a')
          link.href = URL.createObjectURL(blob)
          link.download = `${this.music[this.currentIndex].title}.mp3`
          link.click()
        })
        .catch(console.error)
    },
  },
}
</script>
<style>
.player {
  position: relative;
  max-width: 480px;
  margin: 0 auto;
  min-width: 360px;
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
  min-width: 360px;
  max-width: 480px;
  background-color: transparent;
}
header {
  text-align: center;
  margin-bottom: 10px;
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
  height: 10px;
  background: #d3d3d3;
  outline: none;
  opacity: 0.7;
  -webkit-transition: 0.2s;
  transition: opacity 0.2s;
}
.progress:hover::-webkit-slider-thumb {
  background: purple;
  /* box-shadow: -500px 0 0 500px var(--primary-color); */
}
.progress::-webkit-slider-thumb {
  -webkit-appearance: none;
  cursor: pointer;
  width: 12px;
  height: 12px;
  background: #ec1f55;
  /* box-shadow: -500px 0 0 500px var(--primary-color); */
}
.playlist {
  min-width: 360px;
  padding: 12px;
  box-shadow: rgba(100, 100, 111, 0.2) 0px 7px 29px 0px;
  padding-top: 435px;
}
.song:active {
  opacity: 0.8;
}
</style>
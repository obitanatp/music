<template>
  <main>
      <div>{{all}}</div>
      <br>
      <div>{{music}}</div>
      <br>
      <ul v-for="(m,ind) in music" :key="ind">
          <li>{{m.file}}</li>
      </ul>
      <br>
      <div>{{artist}}</div>
      <br>
      <ul v-for="(art,ind) in artist" :key="ind">
          <li>{{art.title}}</li>
          <li v-if="art.avatar == null">"https://upload.wikimedia.org/wikipedia/commons/thumb/4/4e/Aconcagua2016.jpg/600px-Aconcagua2016.jpg"</li>
          <li v-else>{{art.avatar.url}}</li>
      </ul>
      <button @click="get()">Get</button>
  </main>
</template>

<script>
import axios from 'axios'
export default {
    data(){
        return{
            all : [],
            music : [],
            artist: []
        }
    },
    methods: {
        get(){
            for(var i=0;i<7;i++){
            axios.get('https://api2.bcdcnt.net/graphql',{params: {query: "query($id: ID!) {\n song(id: $id) {\nid\ntitle\nsubtitle\nslug\ncontent\nviews\ndownloads\nfile_type\nlyric_type\nstatus\nyear\nrecord_year\nallow_comment\ncreated_at\nartists(first: 10, orderBy: [{column: \"song_artist.order\", order: ASC}]) {\ndata {\nid\ntitle\nslug\navatar {\nurl\n}\n}\n}\ncomposers(first: 10, orderBy: [{column: \"song_composer.order\", order: ASC}]) { \n data { \n id\n title\n slug\n }\n }\n poets(first: 10) {\n data {\nid\n  title\n slug\n  }\n }\n thumbnail {\n url\n user {\n id\n username\navatar {\nurl\n}\n}\n}\nfile {\nvideo_url\naudio_url\n }\n uploader {\n id\n username\n avatar {\nurl\n}\n}\ndocuments(first: 10) {\ndata {\nid\ntitle\nslug\nthumbnail {\n url\n}\n}\n}\n}\n}"
            ,variables: {id: `${Math.floor(Math.random() * 10000) + 1}`}}}).then(res=>(this.all=this.all.concat(res.data),this.music=this.music.concat(res.data.data.song),this.artist=this.artist.concat(res.data.data.song.artists.data)))
            }
    }
    }
}
</script>

<style>

</style>
<template>
  <ay-loading :loading="loading">
    <table class="music-list">
      <tr class="list-header">
        <th>编号</th>
        <th>封面</th>
        <th>歌名</th>
        <th>歌手</th>
        <th>专辑</th>
        <th>时长</th>
      </tr>
      <tr
        class="album-card"
        v-for="(data, index) in list"
        :key="index"
        @click="addToList(data)"
      >
        <td class="index-wrap">{{ pad(index + 1) }}</td>
        <td class="img-wrap">
          <img class="img" v-lazy="data.al.picUrl" />
        </td>
        <td class="song">{{ data.name }}</td>
        <td class="artists-wrap">
          <span class="artist" v-for="(art, index) in data.ar" :key="index">
            {{ art.name }}
          </span>
        </td>
        <td class="">
          {{ data.name }}
        </td>
        <td class="duration-wrap">
          {{ formatTime(data.dt / 1000) }}
        </td>
      </tr>
    </table>
  </ay-loading>
</template>

<script>
import { getSongDetail } from "@/api";
import { AyLoading } from "@/base";
import { formatTime, pad, formatSong, musicMixin } from "@/utils";
export default {
  name: "SongListSong",
  props: ["ids"],
  mixins: [musicMixin],
  components: { AyLoading },
  data() {
    return {
      list: [],
      loading: false,
    };
  },
  created() {
    this.initList();
  },
  watch: {
    ids() {
      this.initList();
    },
  },
  methods: {
    pad,
    formatTime,
    async initList() {
      this.loading = true;
      const { ids } = this;
      if (ids) {
        const { songs } = await getSongDetail({ ids });
        this.list = songs;
      }
      this.loading = false;
    },
    addToList(song) {
      this.addToPlaylist(
        formatSong({
          id: song.id,
          name: song.name,
          artists: song.ar,
          duration: song.dt,
          mvId: song.mv,
          img: song.al.picUrl,
          albumId: song.al.id,
          albumName: song.al.name,
        })
      );
    },
  },
};
</script>

<style lang="scss" scoped>
.music-list {
  width: 100%;
  border-collapse: collapse;
  border-spacing: 0;
  table-layout: fixed;
  color: $grey-dark;
  .list-header {
    text-align: left;
    th {
      padding: 6px 0;
      color: $black;
      &:first-child {
        padding: 0 24px;
      }
      &:last-child {
        padding: 0 24px;
      }
    }
  }
  .album-card {
    &:nth-child(odd) {
      background-color: $grey-light;
    }
    cursor: pointer;
    height: 64px;
    line-height: 64px;
    width: 100%;
    font-size: $font-size-sm;
    text-align-last: left;
    position: relative;
    transition: 0.3s;
    &:hover {
      background-color: $grey;
    }
    td {
      @include text-ellipsis();
    }
    .index-wrap {
      width: 72px;
      padding: 0 24px;
    }
    .img-wrap {
      width: 72px;
      .img {
        vertical-align: middle;
        height: 48px;
        width: 48px;
      }
    }
    .song {
      color: $black;
    }
    .artists-wrap {
      display: flex;
      .artist {
        &:not(:last-child)::after {
          content: "/";
          display: inline-block;
          width: 16px;
          text-align: center;
        }
      }
    }
    .duration-wrap {
      text-align: right;
      width: 84px;
      padding: 0 24px;
    }
  }
}
</style>

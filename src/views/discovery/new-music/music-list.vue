<template>
  <ay-loading :loading="loading">
    <table class="music-list">
      <tr
        @click="handleGetSong(data)"
        class="album-card"
        v-for="(data, index) in list"
        :key="index"
      >
        <td class="index-wrap">{{ pad(index + 1) }}</td>
        <td class="img-wrap">
          <img class="img" v-lazy="data.album.picUrl" />
        </td>
        <td class="song">{{ data.name }}</td>
        <td class="artists-wrap">
          <span
            class="artist"
            v-for="(art, index) in data.artists"
            :key="index"
          >
            {{ art.name }}
          </span>
        </td>
        <td class="">
          {{ data.album.name }}
        </td>
        <td class="duration-wrap">
          {{ formatTime(data.duration / 1000) }}
        </td>
      </tr>
    </table>
  </ay-loading>
</template>

<script>
import { getTopSong } from "@/api";
import { AyLoading } from "@/base";
import { formatTime, pad, formatSong, musicMixin } from "@/utils";
export default {
  name: "MusicList",
  mixins: [musicMixin],
  components: { AyLoading },
  props: {
    type: {
      // 0:全部 7:华语 96: 欧美 8:日本 16:韩国
      type: Number,
      default: 0,
    },
  },
  data() {
    return {
      list: [],
      loading: false,
    };
  },
  created() {
    this.initList();
  },
  methods: {
    pad,
    formatTime,
    async initList() {
      this.loading = true;
      const { data } = await getTopSong({ type: this.type });
      this.list = data;
      this.loading = false;
    },
    handleGetSong(song) {
      this.setPlaylistLoading(true);
      this.addToPlaylist(
        formatSong({
          id: song.id,
          name: song.name,
          artists: song.artists,
          duration: song.duration,
          mvId: song.mvid,
          img: song.album.picUrl,
          albumId: song.album.id,
          albumName: song.album.name,
        })
      );
      this.setPlaylistLoading(false);
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
  .album-card {
    cursor: pointer;
    &:nth-child(odd) {
      background-color: $grey-light;
    }
    &:hover {
      transition: 0.3s;
      background-color: $grey;
    }
    height: 64px;
    line-height: 64px;
    width: 100%;
    font-size: $font-size-sm;
    text-align-last: left;
    position: relative;
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

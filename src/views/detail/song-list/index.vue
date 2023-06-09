<template>
  <div class="song-list-page-wrap">
    <ay-loading :loading="loading">
      <div class="header">
        <div class="img-wrap">
          <img v-lazy="getImgUrl(list.coverImgUrl, 250, 250)" />
        </div>
        <div class="info-wrap" v-if="!loading">
          <div class="title-wrap">
            <div class="tag">歌单</div>
            <h1 class="title">{{ list.name }}</h1>
          </div>
          <div class="user-wrap" v-if="list.creator">
            <img v-lazy="getImgUrl(list.creator.avatarUrl, 40, 40)" />
            <div class="name">{{ list.creator.nickname }}</div>
            <div class="date" v-if="list.createTime">
              {{ formatDateTime(list.createTime) }}创建
            </div>
          </div>
          <div class="tool-wrap">
            <div class="play-all" @click="playAll">播放全部</div>
          </div>
          <div class="tag-wrap" v-if="list.tags">
            标签：<span v-for="(tag, idx) in list.tags" :key="idx">{{
              tag
            }}</span>
          </div>
          <div class="desc-wrap" v-if="list.description">
            <span>简介：</span>
            <div class="desc">
              <p v-for="(txt, idx) in list.description.split('\n')" :key="idx">
                {{ txt }}
              </p>
            </div>
          </div>
        </div>
        <div class="count-wrap">
          <div class="count" v-if="list.trackCount">
            歌曲数
            <div>
              {{ formatNumber(list.trackCount) }}
            </div>
          </div>
          <div class="count" v-if="list.playCount">
            播放数
            <div>
              {{ formatNumber(list.playCount) }}
            </div>
          </div>
        </div>
      </div>
    </ay-loading>
    <div class="content">
      <ay-tab v-model="activeName" :center="false">
        <ay-tab-pane label="歌曲列表" name="1">
          <songs :ids="ids" ref="songList"></songs>
        </ay-tab-pane>
        <ay-tab-pane :label="`评论(${this.list.commentCount || '-'})`" name="2">
          <comments :id="id" :commentCount="list.commentCount"></comments>
        </ay-tab-pane>
        <ay-tab-pane label="收藏者" name="3">
          <collectors :id="id" :total="list.subscribedCount"></collectors>
        </ay-tab-pane>
      </ay-tab>
    </div>
  </div>
</template>

<script>
import { getPlayListDetail } from "@/api";
import { AyTab, AyTabPane, AyLoading } from "@/base";
import {
  getImgUrl,
  formatDateTime,
  formatNumber,
  formatSong,
  musicMixin,
} from "@/utils";
import Songs from "./songs";
import Comments from "./comments";
import Collectors from "./collectors";
export default {
  name: "SongListDetail",
  mixins: [musicMixin],
  components: {
    AyTab,
    AyTabPane,
    AyLoading,
    Songs,
    Comments,
    Collectors,
  },
  props: ["id"],
  data() {
    return {
      activeName: "1",
      list: {},
      ids: "",
      loading: false,
    };
  },
  created() {
    this.initPlayListDetail();
  },
  watch: {
    id() {
      this.initPlayListDetail();
    },
  },
  methods: {
    getImgUrl,
    formatDateTime,
    formatNumber,
    async initPlayListDetail() {
      this.loading = true;
      const { id } = this;
      const { playlist } = await getPlayListDetail({ id });
      this.list = playlist;
      this.ids = this.list.trackIds.map(({ id }) => id).join(",");
      this.loading = false;
    },
    playAll() {
      if (this.$refs.songList) {
        this.setPlaylistLoading(true);
        const playlist = [];
        this.$refs.songList.list.map((song) => {
          playlist.push(
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
        });
        this.setPlaylist(playlist);
        this.setCurrentSong(playlist[0]);
        this.setPlaylistLoading(false);
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.song-list-page-wrap {
  padding: 18px 0;
  .header {
    min-height: 200px;
    display: flex;
    justify-content: flex-start;
    padding: 0 24px;
    margin-bottom: 48px;
    .img-wrap {
      width: 30%;
      height: 240px;
      img {
        height: 240px;
        width: 240px;
      }
    }
    .info-wrap {
      width: 50%;
      .title-wrap {
        display: flex;
        justify-content: flex-start;
        align-items: center;
        .tag {
          margin: 4px 0;
          padding: 2px 8px;
          border: solid 1px $theme-color;
          color: $theme-color;
          font-size: $font-size-md;
          white-space: nowrap;
        }
        .title {
          font-size: 26px;
          font-weight: 700;
          margin: 0 8px;
        }
      }
      .tool-wrap {
        display: flex;
        justify-content: flex-start;
        .play-all {
          color: $white;
          background-color: $theme-color;
          padding: 4px 8px;
          cursor: pointer;
          font-size: $font-size;
        }
      }
      .user-wrap {
        display: flex;
        justify-content: flex-start;
        align-items: center;
        margin: 12px 0;
        img {
          @include circle(30px);
        }
        .name {
          margin: 0 16px 0 16px;
        }
        .date {
          font-size: $font-size;
          color: $grey-dark;
        }
      }
      .tag-wrap {
        font-size: $font-size;
        margin: 12px 0;
        span {
          color: $blue;
          cursor: pointer;
          &:not(:last-child)::after {
            content: "/";
            display: inline-block;
            margin: 0 6px;
            color: $black;
          }
        }
      }
      .desc-wrap {
        font-size: $font-size;
        display: flex;
        p {
          line-height: 1.4;
        }
      }
    }
    .count-wrap {
      width: 20%;
      color: $grey-dark;
      display: flex;
      align-items: flex-start;
      justify-content: flex-start;
      text-align: right;
      font-size: $font-size-sm;
      .count {
        position: relative;
        &:first-child::after {
          position: absolute;
          top: 0;
          right: 0;
          content: "";
          width: 1px;
          height: 90%;
          display: block;
          background-color: $grey-dark;
          opacity: 0.7;
        }
        padding: 0 8px;
        div {
          font-weight: bold;
          padding: 2px 0;
        }
      }
    }
  }
}
</style>

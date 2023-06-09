<template>
  <transition name="song-detail-page">
    <div class="song-page-wrap" v-if="isDetailShow">
      <div class="content">
        <div class="song-wrap">
          <div class="left">
            <img
              class="play-bar-support"
              :src="require('@/assets/icon/play-bar-support.png')"
            />
            <img
              :class="{ playing }"
              class="play-bar"
              :src="require('@/assets/icon/play-bar.png')"
            />
            <div class="img-outer-border" ref="disc">
              <div
                :class="{ paused: !playing }"
                class="img-outer"
                ref="discRotate"
              >
                <div class="img-wrap">
                  <img v-lazy="getImgUrl(currentSong.img, 400)" />
                </div>
              </div>
            </div>
          </div>
          <div class="right">
            <div class="info-wrap">
              <div class="name">{{ currentSong.name }}</div>
              <div class="artists">歌手：{{ currentSong.artistsText }}</div>
            </div>
            <div class="empty" v-if="nolyric">暂无歌词</div>
            <ul
              @mouseenter="onMouseenter"
              @mouseleave="onMouseleave"
              class="lyric-list"
              ref="lyricList"
              v-else
            >
              <li
                :class="activeLyricIndex === index ? 'active' : ''"
                class="lyric-item"
                ref="lyric"
                v-for="(l, index) in lyricWithTranslation"
                :key="index"
                @click="handleScrollToActiveLyric(l, index)"
              >
                <p
                  v-for="(content, contentIndex) in l.contents"
                  :key="contentIndex"
                >
                  {{ content }}
                </p>
              </li>
            </ul>
          </div>
        </div>
        <div class="bottom">
          <div class="comment-wrap">
            <ay-card :shodow="'hover'">
              <template slot="header">
                <div class="card-header">听友评论({{ total }})</div>
              </template>
              <comment-list
                v-if="current === 1"
                :loading="commentLoading"
                :title="'精彩评论'"
                :comments="hotComments"
              ></comment-list>
              <comment-list
                :loading="commentLoading"
                :title="`最新评论(${total})`"
                :comments="comments"
              ></comment-list>
              <div class="pagination-wrap">
                <ay-pagination
                  v-model="current"
                  :limit="limit"
                  :total="total"
                ></ay-pagination>
              </div>
            </ay-card>
          </div>
          <div class="similar-list-wrap">
            <ay-card :shadow="'hover'">
              <template slot="header">
                <div class="card-header">相似歌单</div>
              </template>
              <ul class="similar-list">
                <li
                  v-for="playlist in simiPlaylist"
                  :key="playlist.id"
                  class="item"
                  @click="toPlaylistDetail(playlist.id)"
                >
                  <horizontal-card
                    :minWidth="'30%'"
                    :imgUrl="playlist.coverImgUrl"
                    :title="playlist.name"
                    :artist="`播放：${formatNumber(playlist.playCount)}`"
                  >
                  </horizontal-card>
                </li>
              </ul>
            </ay-card>
            <ay-card :shadow="'hover'">
              <template slot="header">
                <div class="card-header">相似歌曲</div>
              </template>
              <ul class="similar-list">
                <li
                  v-for="song in simiSongs"
                  :key="song.id"
                  class="item"
                  @click="handleAddToPlaylist(song)"
                >
                  <horizontal-card
                    :minWidth="'30%'"
                    :imgUrl="song.album.picUrl"
                    :title="song.name"
                    :artist="getArtistsText(song.artists)"
                  ></horizontal-card>
                </li>
              </ul>
            </ay-card>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
import { getMusicComment, getLyric, getSimiSong, getSimiPlaylist } from "@/api";
import { AyCard, AyPagination } from "@/base";
import { CommentList, HorizontalCard } from "@/components";
import {
  musicMixin,
  isDefined,
  formatDateTime,
  formatNumber,
  getImgUrl,
  getArtistsText,
  formatSong,
} from "@/utils";
export default {
  name: "CurrentSong",
  mixins: [musicMixin],
  components: { AyCard, AyPagination, CommentList, HorizontalCard },
  data() {
    return {
      limit: 20, // 每页条数
      current: 1, // 评论页
      total: 0, // 评论总数
      hotComments: [], // 热门评论
      comments: [], // 评论
      commentLoading: false, // 评论是否在加载中
      simiSongs: [], // 相似歌曲
      simiPlaylist: [], // 相似歌单
      lyric: [], // 歌词
      tlyric: [], // 翻译
      nolyric: false, // 是否无歌词
      autoScroll: true, // 歌词是否自动滚动
    };
  },
  computed: {
    // 当前歌词
    activeLyricIndex() {
      return this.lyricWithTranslation
        ? this.lyricWithTranslation.findIndex((l, index) => {
            const nextLyric = this.lyricWithTranslation[index + 1];
            return (
              this.currentTime >= l.time &&
              (nextLyric ? this.currentTime < nextLyric.time : true)
            );
          })
        : -1;
    },
    // 用于展示的歌词
    lyricWithTranslation() {
      let ret = [];
      // 空内容的去除
      const lyricFiltered = this.lyric.filter(({ content }) =>
        Boolean(content)
      );
      // content统一转换数组形式
      if (lyricFiltered.length) {
        lyricFiltered.forEach((l) => {
          const { time, content } = l;
          const lyricItem = { time, content, contents: [content] };
          const sameTimeTLyric = this.tlyric.find(
            ({ time: tLyricTime }) => tLyricTime === time
          );
          if (sameTimeTLyric) {
            const { content: tLyricContent } = sameTimeTLyric;
            if (content) {
              lyricItem.contents.push(tLyricContent);
            }
          }
          ret.push(lyricItem);
        });
      } else {
        ret = lyricFiltered.map(({ time, content }) => ({
          time,
          content,
          contents: [content],
        }));
      }
      return ret;
    },
  },
  watch: {
    currentSong(newSong) {
      // 重置分页
      this.current = 1;
      if (newSong.id) {
        // 切歌时重新获取数据
        this.updateComment();
        this.updateLyric();
        this.updateSimiSong();
        this.updateSimiPlaylist();
      } else {
        // 歌曲全部播放完成之后关闭当前页面
        this.setDetailShow(false);
      }
    },
    $route() {
      // 路由切换时隐藏当前页面
      this.setDetailShow(false);
    },
    current() {
      // 获取新一页的评论数据
      this.updateComment();
    },
    activeLyricIndex(idx) {
      // 触发歌词滚动
      if (idx > -1 && this.autoScroll && this.isDetailShow) {
        this.scrollToActiveLyric(idx);
      }
    },
    isDetailShow(show) {
      // 打开页面时手动触发一次歌词滚动
      if (show) {
        this.scrollToActiveLyric(this.activeLyricIndex);
      }
    },
  },
  methods: {
    formatNumber,
    getImgUrl,
    getArtistsText,
    // 格式化歌词内容
    lyricParser(lrc) {
      return {
        lyric: this.parseLyric(lrc.lrc.lyric || ""),
        tlyric: this.parseLyric(lrc.tlyric.lyric || ""),
        lyricuser: lrc.lyricUser,
        transuser: lrc.transUser,
      };
    },
    parseLyric(lrc) {
      const lyrics = lrc.split("\n");
      const lrcObj = [];
      for (let i = 0; i < lyrics.length; i++) {
        const lyric = decodeURIComponent(lyrics[i]);
        const timeReg = /\[\d*:\d*((\.|:)\d*)*\]/g;
        const timeRegExpArr = lyric.match(timeReg);
        if (!timeRegExpArr) continue;
        const content = lyric.replace(timeReg, "");
        for (let k = 0, h = timeRegExpArr.length; k < h; k++) {
          const t = timeRegExpArr[k];
          const min = Number(String(t.match(/\[\d*/i)).slice(1));
          const sec = Number(String(t.match(/:\d*/i)).slice(1));
          const time = min * 60 + sec;
          if (content !== "") {
            lrcObj.push({ time: time, content });
          }
        }
      }
      return lrcObj;
    },
    // 鼠标移入停止歌词自动滚动
    onMouseenter() {
      this.autoScroll = false;
    },
    // 鼠标移开开启歌词自动滚动
    onMouseleave() {
      this.autoScroll = true;
    },
    // 鼠标点击歌词
    handleScrollToActiveLyric(lrcData, idx) {
      const audio = document.getElementById("audio");
      if (audio) {
        audio.currentTime = lrcData.time;
        this.scrollToActiveLyric(idx);
      }
    },
    // 滚动至当前歌词
    scrollToActiveLyric(activeLyricIndex) {
      const { lyricList } = this.$refs;
      if (!isDefined(lyricList)) return;
      const child = lyricList.children[activeLyricIndex];
      const top = child.offsetTop - lyricList.clientHeight / 2;
      lyricList.scrollTo({
        top,
        behavior: "smooth",
      });
    },
    // 跳转歌单详情页
    toPlaylistDetail(id) {
      this.$router.push({ name: "SongListDetail", params: { id } });
    },
    // 添加歌曲到歌单中
    handleAddToPlaylist(song) {
      this.addToPlaylist(
        formatSong({
          id: song.id,
          name: song.name,
          artists: song.artists,
          duration: song.duration,
          mvId: song.mvid,
          img: song.album.picUrl,
        })
      );
    },
    // 格式化评论数据
    formatCommentData(comment) {
      const res = {
        avatar: comment.user.avatarUrl,
        name: comment.user.nickname,
        date: formatDateTime(comment.time),
        content: comment.content,
        count: comment.likedCount,
      };
      if (comment.beReplied.length) {
        res.replyName = comment.beReplied[0].user.nickname;
        res.replyContent = comment.beReplied[0].content;
      }
      return res;
    },
    // 请求评论
    async updateComment() {
      this.commentLoading = true;
      const { currentSong, limit, current } = this;
      const { id } = currentSong;
      const { total, hotComments, comments } = await getMusicComment({
        id,
        limit,
        offset: limit * (current - 1),
      });
      this.total = total;
      this.hotComments = hotComments;
      this.comments = comments;
      this.commentLoading = false;
    },
    // 请求歌词
    async updateLyric() {
      const { id } = this.currentSong;
      const result = await getLyric({ id });
      this.nolyric = !isDefined(result.lrc) || !result.lrc.lyric;
      if (!this.nolyric) {
        const { lyric, tlyric } = this.lyricParser(result);
        this.lyric = lyric;
        this.tlyric = tlyric;
      }
    },
    // 请求相似歌曲
    async updateSimiSong() {
      const { id } = this.currentSong;
      const { songs } = await getSimiSong({ id });
      this.simiSongs = songs;
    },
    // 请求相似歌单
    async updateSimiPlaylist() {
      const { id } = this.currentSong;
      const { playlists } = await getSimiPlaylist({ id });
      this.simiPlaylist = playlists;
    },
  },
};
</script>

<style lang="scss" scoped>
.song-page-wrap {
  position: fixed;
  top: 64px;
  bottom: 64px;
  left: 0;
  height: calc(100vh - 128px);
  width: 100%;
  z-index: $current-song-index;
  background-color: white;
  overflow-y: auto;
  .content {
    max-width: 1080px;
    margin: 0 auto;
    .song-wrap {
      display: flex;
      margin-bottom: 20px;
      .left {
        $disc-width: 400px;
        $support-width: 30px;

        height: 540px;
        width: $disc-width;
        position: relative;
        display: flex;
        justify-content: center;
        padding-top: 53px;
        .play-bar-support {
          position: absolute;
          top: -$support-width/2;
          left: $disc-width/2;
          width: $support-width;
          height: $support-width;
          z-index: 2;
        }
        .play-bar {
          position: absolute;
          top: 0;
          left: $disc-width/2 + 10px;
          width: 100px;
          height: 146px;
          z-index: 1;
          transform-origin: 0 0;
          transform: rotate(-30deg);
          transition: all 0.3s;
          &.playing {
            transform: rotate(5deg);
          }
        }
        .img-outer-border {
          @include circle(340px);
          display: flex;
          justify-content: center;
          align-items: center;
          background-color: $grey;
          .img-outer {
            @include circle(310px);
            display: flex;
            justify-content: center;
            align-items: center;
            background-color: $black;
            animation: rotate 20s linear infinite;
            &.paused {
              animation-play-state: paused;
            }
            .img-wrap {
              width: 200px;
              height: 200px;
              flex-shrink: 0;
              img {
                @include circle(100%);
              }
            }
          }
        }
      }
      .right {
        margin-left: 120px;
        width: 100%;
        .info-wrap {
          margin: 40px 0;
          .name {
            font-size: 24px;
            margin-bottom: 16px;
          }
          .artists {
            font-size: $font-size-sm;
            margin-bottom: 16px;
          }
        }
        .empty {
          text-align: center;
        }
        .lyric-list {
          position: relative;
          height: 350px;
          mask-image: linear-gradient(
            180deg,
            hsla(0, 0%, 100%, 0) 0,
            hsla(0, 0%, 100%, 0.6) 15%,
            #fff 25%,
            #fff 75%,
            hsla(0, 0%, 100%, 0.6) 85%,
            hsla(0, 0%, 100%, 0)
          );
          overflow-y: auto;
          .lyric-item {
            font-size: $font-size-sm;
            color: $grey-dark;
            line-height: 1.6;
            margin-bottom: 8px;
            cursor: pointer;
            transition: 0.3s;
            &:hover {
              color: $black;
            }
            &.active {
              font-weight: bold;
              font-size: $font-size;
              color: $black;
            }
          }
          &::-webkit-scrollbar {
            width: 0;
          }
          &:hover {
            &::-webkit-scrollbar {
              width: 4px;
            }
          }
        }
      }
    }
    .bottom {
      display: flex;
      .comment-wrap {
        width: 60%;
        .pagination-wrap {
          float: right;
        }
      }
      .similar-list-wrap {
        width: 30%;
        padding-left: 10%;
        .similar-list {
          margin-bottom: 60px;
          .item {
            width: 100%;
            cursor: pointer;
            transition: 0.3s;
            padding: 12px 4px;
            border-radius: 6px;
            &:hover {
              background-color: $grey;
            }
          }
        }
      }
    }
  }
}
</style>

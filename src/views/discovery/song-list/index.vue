<template>
  <div ref="playlist" class="page-wrap">
    <ay-popover
      trigger="click"
      placement="bottom"
      v-if="catSelectionList.length"
    >
      <template slot="content">
        <div class="tag-selection">
          <div
            class="tag-all"
            :class="currentTag.name === all.name ? 'active' : ''"
            @click="selectTag(all)"
          >
            {{ all.name }}
          </div>
          <div
            class="tag-wrap"
            v-for="cat in catSelectionList"
            :key="cat.title"
          >
            <div class="header">{{ cat.title }}</div>
            <div class="content">
              <div
                class="item"
                :class="currentTag.name === item.name ? 'active' : ''"
                v-for="item in cat.list"
                :key="item.name"
                @click="selectTag(item)"
              >
                {{ item.name }}
              </div>
            </div>
          </div>
        </div>
      </template>
      <div class="current-tag">{{ currentTag.name }}</div>
    </ay-popover>
    <tag-list
      :title="'热门标签：'"
      :tags="tags"
      @tagChange="selectTag"
    ></tag-list>
    <ay-loading :loading="listLoading">
      <ul class="list-wrap">
        <li class="list-item" v-for="song in songlist" :key="song.id">
          <song-card
            @click="handleGetSong(song.id)"
            :count="formatNumber(song.playCount).toString()"
            :imgUrl="song.coverImgUrl"
            :artistName="song.creator.nickname"
            :name="song.name"
            :id="song.id"
          ></song-card>
        </li>
      </ul>
    </ay-loading>
    <div class="pagination-wrap">
      <ay-pagination
        v-model="pagination.currentPage"
        :pageCount="pagination.pageCount"
        :limit="params.limit"
        :total="total"
      ></ay-pagination>
    </div>
  </div>
</template>

<script>
import {
  getCatList,
  getHotCatList,
  getTopPlayList,
  getPlayListDetail,
  getSongDetail,
} from "@/api";
import { AyPopover, AyLoading, AyPagination } from "@/base";
import { SongCard, TagList } from "@/components";
import { formatNumber, musicMixin, formatSong } from "@/utils";
const OFFSET_VAL = 0;
export default {
  name: "SongList",
  mixins: [musicMixin],
  components: { AyPopover, AyLoading, AyPagination, SongCard, TagList },
  data() {
    return {
      listLoading: false,
      catLoading: false,
      currentTag: {},
      all: {},
      /**
       * 0: "语种"
       * 1: "风格"
       * 2: "场景"
       * 3: "情感"
       * 4: "主题"
       */
      categories: {},
      sub: [],
      tags: [],
      songlist: [],
      pagination: {
        currentPage: 1,
        pageCount: 7,
      },
      total: 0,
      params: {
        order: "hot",
        limit: 50,
        offset: OFFSET_VAL,
        cat: "",
      },
      contentEl: document.getElementById("content__ref"),
    };
  },
  created() {
    this.initCatList();
    this.initHotCatList();
  },
  computed: {
    catSelectionList() {
      const categories = Object.values(this.categories);
      const catSelectionList = [];
      for (let i = 0; i < categories.length; i++) {
        catSelectionList.push({
          title: categories[i],
          list: this.filterCatList(i),
        });
      }
      return catSelectionList;
    },
  },
  watch: {
    params: {
      handler() {
        this.initSongList();
      },
      deep: true,
    },
    "pagination.currentPage": function (page) {
      this.params.offset = (page - 1) * this.params.limit;
    },
    currentTag(tag) {
      const { params } = this;
      params.offset = OFFSET_VAL;
      params.cat = tag.name;
      this.params = params;
    },
  },
  methods: {
    formatNumber,
    filterCatList(type = 0) {
      return this.sub.filter((item) => item.category === type);
    },
    selectTag(tag) {
      this.currentTag = tag;
    },
    async initCatList() {
      this.catLoading = true;
      const { sub, categories, all } = await getCatList();
      this.sub = sub;
      this.categories = categories;
      this.all = all;
      this.currentTag = all;
      this.catLoading = false;
    },
    async initHotCatList() {
      const { tags } = await getHotCatList();
      this.tags = tags;
    },
    async initSongList() {
      this.listLoading = true;
      const { params } = this;
      const { playlists, total } = await getTopPlayList(params);
      this.total = total;
      this.songlist = playlists;
      this.contentEl &&
        this.contentEl.scrollTo({
          left: 0,
          top: 0,
          behavior: "smooth",
        });
      this.listLoading = false;
    },
    async handleGetSong(id) {
      this.setPlaylistLoading(true);
      const { playlist } = await getPlayListDetail({ id });
      const { trackIds } = playlist;
      await this.getSongDetail(trackIds.map(({ id }) => id).join(","));
      this.setPlaylistLoading(false);
    },
    async getSongDetail(ids) {
      const { songs } = await getSongDetail({ ids });
      const playlist = [];
      songs.map((song) => {
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
    },
  },
};
</script>

<style lang="scss" scoped>
.page-wrap {
  @include list(20%);
  .current-tag {
    border: $grey 1px solid;
    padding: 12px 16px;
    font-size: $font-size-sm;
  }
  .pagination-wrap {
    margin-top: 32px;
    float: right;
  }
}
.tag-selection {
  height: 300px;
  overflow-y: scroll;
  padding: 0 20px;
  .tag-all {
    width: 100%;
    border: solid 1px $grey;
    height: 36px;
    line-height: 36px;
    text-align: center;
    margin-bottom: 18px;
    cursor: pointer;
  }
  .active {
    border: solid 1px $theme-color;
  }
  .tag-wrap {
    margin-bottom: 18px;
    .header {
      height: 36px;
      line-height: 36px;
      font-weight: bold;
    }
    .content {
      display: flex;
      width: 300px;
      flex-wrap: wrap;
      font-size: $font-size-sm;
      .item {
        cursor: pointer;
        border: solid 1px $grey;
        width: 84px;
        height: 32px;
        line-height: 32px;
        text-align: center;
      }
      .active {
        border: solid 1px $theme-color;
      }
    }
  }
}
</style>

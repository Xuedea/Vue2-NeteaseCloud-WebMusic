<template>
  <div class="mv-card-list">
    <div class="card-wrap">
      <ay-card :headerStyle="cardHeaderStyle">
        <template slot="header">
          <div class="card-header">
            <div class="left">
              <div class="title">最新MV</div>
              <div class="tag-wrap">
                <tag-list
                  :active="true"
                  :defaultActive="newActiveTag"
                  :tags="areaTags"
                  @tagChange="onNewTagChange"
                  :wrapStyle="tagWrapStyle"
                ></tag-list>
              </div>
            </div>
            <div class="right" @click="toDetail('AllMv')">更多></div>
          </div>
        </template>
        <ay-loading :loading="newMvsLoading">
          <ul class="list-wrap">
            <li
              @click="toMv(mv.id)"
              class="list-item"
              v-for="mv in newMvs"
              :key="mv.id"
            >
              <mv-card
                :imgUrl="mv.cover"
                :count="formatNumber(mv.playCount).toString()"
                :name="mv.name"
                :artistName="getArtistsText(mv.artists)"
              ></mv-card>
            </li>
          </ul>
        </ay-loading>
      </ay-card>
    </div>
    <div class="card-wrap">
      <ay-card :headerStyle="cardHeaderStyle">
        <template slot="header">
          <div class="card-header">
            <div class="left">
              <div class="title">热播MV</div>
            </div>
            <div class="right" @click="toDetail('AllMv')">更多></div>
          </div>
        </template>
        <ay-loading :loading="hotMvsLoading">
          <ul class="list-wrap">
            <li
              @click="toMv(mv.id)"
              class="list-item"
              v-for="mv in hotMvs"
              :key="mv.id"
            >
              <mv-card
                :imgUrl="mv.cover"
                :count="formatNumber(mv.playCount).toString()"
                :name="mv.name"
                :artistName="getArtistsText(mv.artists)"
              ></mv-card>
            </li></ul
        ></ay-loading>
      </ay-card>
    </div>
    <div class="card-wrap">
      <ay-card :headerStyle="cardHeaderStyle">
        <template slot="header">
          <div class="card-header">
            <div class="left">
              <div class="title">网易出品</div>
            </div>
            <div class="right" @click="toDetail('AllMv')">更多></div>
          </div>
        </template>
        <ay-loading :loading="exclusiveMvLoading">
          <ul class="list-wrap">
            <li
              @click="toMv(mv.id)"
              class="list-item"
              v-for="mv in exclusiveMvs"
              :key="mv.id"
            >
              <mv-card
                :imgUrl="mv.cover"
                :count="formatNumber(mv.playCount).toString()"
                :name="mv.name"
                :artistName="getArtistsText(mv.artists)"
              ></mv-card>
            </li>
          </ul>
        </ay-loading>
      </ay-card>
    </div>
    <div class="card-wrap">
      <ay-card :headerStyle="cardHeaderStyle">
        <template slot="header">
          <div class="card-header">
            <div class="left">
              <div class="title">MV排行榜</div>
              <div class="tag-wrap">
                <tag-list
                  :active="true"
                  :defaultActive="rankActiveTag"
                  :tags="areaTags"
                  @tagChange="onRankTagChange"
                  :wrapStyle="tagWrapStyle"
                ></tag-list>
              </div>
            </div>
            <div class="right" @click="toDetail('MvRankList')">更多></div>
          </div>
        </template>
        <div class="rank">
          <ay-loading :loading="mvRankLoading">
            <ul class="list-wrap">
              <li
                @click="toMv(mv.id)"
                class="list-item"
                v-for="(mv, idx) in mvRank"
                :key="mv.id"
              >
                <div class="rank-card-wrap">
                  <div class="idx">
                    {{ pad(idx + 1) }}
                  </div>
                  <div class="horizontal-card-wrap">
                    <horizontal-card
                      :title="mv.name"
                      :artist="getArtistsText(mv.artists)"
                    >
                      <template slot="banner">
                        <mv-card
                          :imgUrl="mv.cover"
                          :count="formatNumber(mv.playCount).toString()"
                        ></mv-card>
                      </template>
                    </horizontal-card>
                  </div>
                </div>
              </li>
            </ul>
          </ay-loading>
        </div>
      </ay-card>
    </div>
  </div>
</template>

<script>
import { getTopMv, getAllMv, getNewMv, getExclusiveMv } from "@/api";
import { AyCard, AyLoading } from "@/base";
import { TagList, MvCard, HorizontalCard } from "@/components";
import { areaTags } from "@/config";
import { formatNumber, getArtistsText, pad } from "@/utils";
const defaultParams = {
  limit: 6,
  offset: 0,
};
export default {
  name: "MvList",
  components: { AyCard, AyLoading, TagList, MvCard, HorizontalCard },
  data() {
    const cardHeaderStyle = {
      padding: "6px 16px 0 16px",
    };
    const tagWrapStyle = {
      margin: "0",
    };
    return {
      areaTags,
      cardHeaderStyle,
      tagWrapStyle,
      newActiveTag: areaTags[0], // 被选中地区（最新 MV）
      rankActiveTag: areaTags[0], // 被选中地区（ MV 排行榜）
      hotMvs: [], // 热门 MV
      exclusiveMvs: [], // 网易出品 MV
      newMvs: [], // 最新 MV
      mvRank: [], // MV 排行榜
      hotMvsLoading: false,
      exclusiveMvLoading: false,
      newMvsLoading: false,
      mvRankLoading: false,
    };
  },
  created() {
    this.getHotMvList();
    this.getExclusiveMvList();
    this.getNewMvList();
    this.getMvRank();
  },
  watch: {
    newActiveTag() {
      this.newMvs = [];
      this.getNewMvList();
    },
    rankActiveTag() {
      this.mvRank = [];
      this.getMvRank();
    },
  },
  computed: {},
  methods: {
    formatNumber,
    getArtistsText,
    pad,
    onNewTagChange(tag) {
      this.newActiveTag = tag;
    },
    onRankTagChange(tag) {
      this.rankActiveTag = tag;
    },
    toDetail(name) {
      this.$router.push({ name });
    },
    toMv(id) {
      id && this.$router.push({ name: "MvDetail", params: { id } });
    },
    // 获取热门 MV
    async getHotMvList() {
      this.hotMvsLoading = true;
      const { data } = await getAllMv({ order: "最热", ...defaultParams });
      this.hotMvs = data;
      this.hotMvsLoading = false;
    },
    // 获取网易出品 MV
    async getExclusiveMvList() {
      this.exclusiveMvLoading = true;
      const { data } = await getExclusiveMv(defaultParams);
      this.exclusiveMvs = data;
      this.exclusiveMvLoading = false;
    },
    // 获取最新 MV
    async getNewMvList() {
      this.newMvsLoading = true;
      const { newActiveTag } = this;
      const { data } = await getNewMv({
        area: newActiveTag.name,
        ...defaultParams,
      });
      this.newMvs = data;
      this.newMvsLoading = false;
    },
    // 获取 MV 排行榜
    async getMvRank() {
      this.mvRankLoading = true;
      const { rankActiveTag } = this;
      const { data } = await getTopMv({
        limit: 10,
        offset: 0,
        area: rankActiveTag.name,
      });
      this.mvRank = data;
      this.mvRankLoading = false;
    },
  },
};
</script>

<style lang="scss" scoped>
.mv-card-list {
  .card-wrap {
    margin: 24px 0;
    @include list(33.3%);
    .list-wrap {
      min-height: 100px;
    }
    .card-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      .left {
        display: flex;
        height: 42px;
        line-height: 42px;
        .title {
          font-size: 21px;
        }
        .tag-wrap {
          display: flex;
          align-items: center;
        }
      }
      .right {
        color: $grey-dark;
        font-size: $font-size-sm;
        cursor: pointer;
      }
    }
    .rank {
      @include list(50%);
      .rank-card-wrap {
        display: flex;
        width: 100%;
        .idx {
          font-size: 32px;
          font-weight: bold;
          color: $grey-dark;
          margin-right: 12px;
          width: 10%;
        }
        .horizontal-card-wrap {
          height: 100%;
          width: 90%;
        }
      }
    }
  }
}
</style>

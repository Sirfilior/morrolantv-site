<template>
  <main>
    <section class="hero subpage w-100">
      <div class="image-wrapper">
        <picture>
          <source
            :srcset="require(`~/assets/img/webp/${heroimg}.webp`)"
            type="image/webp"
          />
          <source
            :srcset="require(`~/assets/img/webp/${heroimg}.jpg`)"
            type="image/jpeg"
          />
          <img
            class="image"
            :src="require(`~/assets/img/webp/${heroimg}.jpg`)"
            alt="Alt Text!"
          />
        </picture>
      </div>
    </section>
    <section class="section">
      <div class="container">
        <h1 class="title is-1">Worker Empire Tool</h1>
        <div class="node-calculator">
          <div v-if="$fetchState.pending" class="loader"></div>
          <div v-else-if="inactive" class="error-wrapper">
            <div class="error">
              <h3 class="title">Maintenance</h3>
              <h5 class="subtitle">We are working on the tool</h5>
              <p>
                We are working on improving or fixing the node tool.Check back
                later.
              </p>
            </div>
          </div>
          <div v-else-if="$fetchState.error" class="error-wrapper">
            <div class="error">
              <h3 class="title">API ERROR</h3>
              <h5 class="subtitle">COULD NOT FETCH DATA</h5>
              <p>Our API is having trouble at the moment, check back later.</p>
            </div>
          </div>
          <div v-else>
            <div class="nodecalc-header">
              <h3 class="title is-4">Guide</h3>
              <p class="mb-3">
                This tool provides a general comparison of node profitability.
                Many factors influence your net gain, and these numbers serve as
                an estimate to help you find the better nodes more quickly.
              </p>
              <p class="mb-2">
                <strong class="has-text-info"
                  >To get numbers more precise to your situation, you can click
                  the node and change the following values:</strong
                >
              </p>
              <ul>
                <li class="mb-1">
                  <strong>Worker:</strong> The tool will automatically select
                  the best worker based on your Active Hours (set in Options).
                  Active Hours should be based on what portion of the day you
                  can keep your workers fed. You don't have to be fully active
                  during this time - generally you only need to feed workers
                  every 3 to 4 hours.
                </li>
                <li class="mb-1">
                  <strong>Path CP:</strong> Default pathing is determined
                  incrementally based on whether a previous node along the way
                  is likely to be taken. Edit this if you are using a different
                  path. This is for the path ONLY, the value for the material
                  node itself cannot be changed.
                </li>
                <li class="mb-1">
                  <strong>Work Speed:</strong> Fine tune your worker's work
                  speed with this value.
                </li>
                <li class="mb-1">
                  <strong>Move Speed:</strong> Fine tune your worker's move
                  speed with this value.
                </li>
                <li class="mb-3">
                  <strong>Link:</strong> You can combine two or more nodes in a
                  group via "Add Links". This will combine the profit and CP
                  cost across the group, and average them together. This is
                  useful for when the path CP is split between multiple nodes,
                  or when backtracking, or other various special cases.
                </li>
                <li class="mb-3">
                  <strong>Claim:</strong> This option allows you to
                  <span style="color: #aeff5c">claim</span> nodes. This will be
                  saved to your account once logged in. The total cp and
                  profit/day of claimed nodes is listed at the bottom of the
                  page.
                </li>
                <li class="mb-1">
                  <strong>Materials:</strong> The green arrow indicates an item
                  is "<strong
                    style="color: #2e599b"
                    data-tooltip="item is sold out with orders at max price"
                    >maxed</strong
                  >" and a red symbol indicates an item is "<strong
                    style="color: #2e599b"
                    data-tooltip="n>10000 are listed at minprice. Not inclued in profit"
                    >flooded</strong
                  >". You can manually click any material to toggle the profit
                  on or off. Flooded items are toggled off by default.
                </li>
              </ul>
              <div class="is-flex justify-between materialUpdate">
                <h5 class="mb-3">
                  Presets for options can be saved in your profile!
                </h5>
                <h5 class="mb-3">
                  {{ timeUpdated }}
                </h5>
              </div>
            </div>
            <div
              class="nodecalc-filter-wrapper is-flex align-center justify-between mb-3"
            >
              <div class="is-flex">
                <div
                  class="filter-list mr-3"
                  :class="selectedRegion != '' ? 'hasSelected' : ''"
                >
                  <div
                    v-for="region in filteredRegions"
                    :key="region"
                    class="region-filter"
                    :class="region == selectedRegion ? 'on' : 'off'"
                    :data-tooltip="region.toUpperCase()"
                    @click="selectRegion(region)"
                  >
                    <img
                      :src="generateRegionImage(region)"
                      class="region-img"
                    />
                  </div>
                  <div class="materialfilter-wrapper">
                    <div class="field">
                      <div class="control">
                        <input
                          v-model.number.lazy="materialFilter"
                          type="text"
                          class="input"
                          placeholder="Filter materials"
                        />
                      </div>
                    </div>
                  </div>
                </div>
              </div>
              <div>
                <div
                  class="sort-options is-flex align-center"
                  style="position: relative"
                >
                  <transition name="fade">
                    <div v-if="optionsOpen" class="node-options-wrapper">
                      <div class="is-flex">
                        <div class="buttons has-addons mb-0 mr-2">
                          <button
                            class="button mb-0"
                            :class="
                              this.$store.state.playerRegion === 'NA'
                                ? 'is-selected is-warning'
                                : ''
                            "
                            @click="setMarketRegion('NA')"
                          >
                            NA
                          </button>
                          <button
                            class="button mb-0"
                            :class="
                              this.$store.state.playerRegion === 'EU'
                                ? 'is-selected is-warning'
                                : ''
                            "
                            @click="setMarketRegion('EU')"
                          >
                            EU
                          </button>
                        </div>
                        <div class="filter-additional">
                          <div
                            class="fishing-filter"
                            :class="{ active: showFishing }"
                            data-tooltip="Hide/Show fishing nodes"
                            @click="showFishing = !showFishing"
                          >
                            <fa :icon="['fas', 'fish']" />
                          </div>
                        </div>
                      </div>
                      <div class="mt-2">
                        <div class="field is-horizontal is-narrow">
                          <div class="field-label is-normal">
                            <label class="label" style="white-space: nowrap"
                              >Active H</label
                            >
                          </div>
                          <div class="field-body">
                            <div class="field">
                              <div class="control">
                                <input
                                  v-model.number.lazy="activeHours"
                                  type="number"
                                  class="input"
                                  placeholder="24"
                                />
                              </div>
                            </div>
                          </div>
                        </div>
                      </div>
                      <div class="buttons has-addons mb-0 mt-2">
                        <button
                          class="button mb-0"
                          :class="
                            editMode === 'basic' ? 'is-selected is-warning' : ''
                          "
                          @click="setEditMode('basic')"
                        >
                          Basic
                        </button>
                        <button
                          class="button mb-0"
                          :class="
                            editMode === 'advanced'
                              ? 'is-selected is-warning'
                              : ''
                          "
                          @click="setEditMode('advanced')"
                        >
                          Advanced
                        </button>
                      </div>
                    </div>
                  </transition>
                  <button
                    class="button is-primary functional mr-2"
                    @click="optionsOpen = !optionsOpen"
                  >
                    <span>Options</span>
                  </button>
                  <div class="linkingbuttons">
                    <button
                      v-if="!nodesError"
                      class="button is-primary functional mr-2"
                      :class="{ enabled: linkingActive }"
                      @click="handleLinking()"
                    >
                      {{
                        unlinkingActive
                          ? 'Stop'
                          : linkingActive
                          ? 'Done'
                          : 'Add Links'
                      }}
                    </button>
                    <button
                      v-else
                      class="button is-primary functional mr-2"
                      :data-tooltip="nodesError"
                      :class="{ enabled: linkingActive }"
                      @click="handleLinking()"
                    >
                      {{
                        unlinkingActive
                          ? 'Stop'
                          : linkingActive
                          ? 'Done'
                          : 'Add Links'
                      }}
                    </button>
                    <button
                      v-if="!unlinkingActive"
                      class="button is-primary functional mr-2 unlinkbtn"
                      data-tooltip="Unlink - Click either one of a group node"
                      :class="{ enabled: unlinkingActive }"
                      @click="startUnlink()"
                    >
                      Unlink
                    </button>
                  </div>
                  <button
                    class="button is-primary mr-2"
                    :class="{ enabled: takingActive }"
                    @click="handleTaking()"
                  >
                    {{ takingActive ? 'Done' : 'Claim' }}
                  </button>
                  <button class="button is-primary mr-2" @click="updateList()">
                    Re-Sort
                  </button>
                  <client-only>
                    <button
                      v-if="$auth.loggedIn"
                      class="button is-primary save functional"
                      :class="{ enabled: saving, action: !saveState }"
                      @click="saveNodes()"
                    >
                      {{ saving ? 'Saving..' : 'Save' }}
                    </button>
                  </client-only>
                </div>
              </div>
            </div>
            <div v-show="nodeGroupsCalculated" class="nodecalc-list columns">
              <div
                v-for="node in nodes"
                :key="node.id"
                class="node-wrapper column is-one-third"
              >
                <Node
                  v-if="node.Materials[0]"
                  :id="node.id"
                  :name="node.name"
                  :contribution="node.contribution"
                  :cp-add="node.cpAdd"
                  :taken="node.taken"
                  :image="node.image"
                  :materials="node.Materials"
                  :region="node.region"
                  :workload="node.workload"
                  :preset-workspeed="node.workspeed"
                  :preset-movespeed="node.movespeed"
                  :preset-luck="node.luck"
                  :distances="JSON.parse(node.distances)"
                  :lodging="node.lodging"
                  :group="JSON.parse(node.group)"
                  :group-cp="node.groupCP ? node.groupCP : null"
                  :group-profit="node.groupProfit ? node.groupProfit : null"
                  @recalculated="updateListAuto"
                />
              </div>
            </div>
            <div v-if="takenCp > 0" class="summary-fixed">
              <div class="summary-inner">
                <span
                  >Total CP: <strong>{{ takenCp }}</strong></span
                >
                <span
                  >Silver/Day:
                  <strong>{{ parseValue(takenProfit) }}</strong></span
                >
              </div>
            </div>
          </div>
        </div>
        <div class="modal" :class="error ? 'is-active' : ''">
          <div class="modal-background"></div>
          <div class="modal-content">
            <h4 class="title is-4">Error</h4>
            <h5 class="subtitle is-5">{{ errorMessage }}</h5>
            <button
              class="button is-primary"
              aria-label="close"
              @click="reloadPage()"
            >
              OK
            </button>
          </div>
        </div>
      </div>
    </section>
    <BackTop />
  </main>
</template>
<script>
import { createNamespacedHelpers } from 'vuex'
import Parser from '../../helper/parsers'
const { mapState, mapGetters } = createNamespacedHelpers('nodes')

export default {
  async fetch() {
    if (this.$auth.loggedIn) {
      const userData = await Promise.all([
        this.$axios.$get('/user/userNodes', {
          headers: {
            Authorization: this.$auth.getToken('auth0'),
          },
        }),
        this.$axios.$get('/user/materialPreferences', {
          headers: {
            Authorization: this.$auth.getToken('auth0'),
          },
        }),
      ]).then((res) => ({ nodes: res[0], materialPRef: res[1] }))
      if (userData.materialPRef) {
        this.$store.commit(
          'nodes/SET_MATERIAL_PREFERENCES',
          JSON.parse(userData.materialPRef.disabledMaterials)
        )
        this.$store.commit(
          'nodes/SET_MATERIAL_OVERRIDES',
          JSON.parse(userData.materialPRef.overrideMaterials)
        )
      }
      if (userData.nodes) {
        this.materialUpdateTime = userData.nodes[0].Materials
          ? userData.nodes[0].Materials.updatedAt
          : ''
        this.$store.commit('nodes/SET_NODES', userData.nodes)
      }
      if (sessionStorage.getItem('usernodes')) {
        this.restoreUsernodes()
      }
    } else {
      // Fetch default node data
      const nodes = await this.$axios.$get('/nodes').then((res) => res)
      if (nodes) {
        this.materialUpdateTime = nodes[0].Materials
          ? nodes[0].Materials[0].updatedAt
          : ''
        this.$store.commit('nodes/SET_NODES', nodes)
      }
    }
  },
  data() {
    return {
      heroimg: 'world',
      error: null,
      errorMessage: '',
      autoUpdate: false,
      saving: false,
      linking: false,
      selectedRegion: '',
      showFishing: false,
      materialFilter: '',
      optionsOpen: false,
      animate: false,
      materialUpdateTime: '',
      regions: [
        'balenos',
        'calpheon',
        'drieghan',
        'kamasylvia',
        'mediah',
        'serendia',
        'valencia',
        'margoria',
        'odyllita',
      ],
    }
  },
  computed: {
    inactive() {
      return false
    },
    timeUpdated() {
      if (this.materialUpdateTime) {
        const [date, time] = this.materialUpdateTime.split('T')
        return `Materials Updated: ${date} ${time.split('.')[0]} UTC+0`
      } else {
        return ''
      }
    },
    filteredRegions() {
      if (this.showFishing) return this.regions
      return this.regions.filter((r) => r !== 'margoria')
    },
    nodes() {
      if (this.nodeGroupsCalculated) {
        let nodes = this.getNodesByProfit
        if (this.selectedRegion)
          nodes = nodes.filter(
            (node) => node.region.toLowerCase() === this.selectedRegion
          )
        if (!this.showFishing) {
          nodes = nodes.filter((node) => !node.name.includes('Island'))
        }
        if (this.materialFilter) {
          nodes = nodes.filter((node) => {
            for (const m of node.Materials) {
              if (
                m.name.toUpperCase().includes(this.materialFilter.toUpperCase())
              ) {
                return true
              }
            }
          })
        }
        this.$store.commit('nodes/SET_MOUNTED_NODES_COUNT', nodes.length)
        return nodes
      } else {
        this.$store.commit(
          'nodes/SET_MOUNTED_NODES_COUNT',
          this.getNodesUnsorted.length
        )
        return this.getNodesUnsorted
      }
    },
    activeHours: {
      // getter
      get() {
        return this.$store.state.nodes.activeHours
      },
      // setter
      set(newValue) {
        let val = newValue
        if (val > 24) val = 24
        else if (val < 0) val = 0
        this.$store.commit('nodes/SET_ACTIVE_HOURS', val)
      },
    },
    nodeTransition() {
      return this.animate ? 'flip-list' : 'disabled-list'
    },
    ...mapGetters(['getNodesByProfit', 'getNodesUnsorted', 'getChangedNodes']),
    ...mapState([
      'workers',
      'nodesError',
      'nodesCalculated',
      'nodeGroupsCalculated',
      'profitsUpdated',
      'nodeUserListLoaded',
      'linkingActive',
      'takingActive',
      'unlinkingActive',
      'linkOrigin',
      'linkTarget',
      'disabledItems',
      'overrideFloodedItems',
      'editMode',
      'saveState',
      'takenCp',
      'takenProfit',
    ]),
  },
  fetchOnServer: false,
  beforeMount() {
    this.showFishing = this.$store.state.showFishing
  },
  methods: {
    loginNodes() {
      this.$auth.loginWith('auth0')
    },
    updateList() {
      this.$store.commit('nodes/PROFITS_UPDATED')
    },
    setMarketRegion(r) {
      this.$store.commit('SET_MARKET_REGION', r)
    },
    setEditMode(m) {
      this.$store.commit('nodes/SET_EDIT_MODE', m)
    },
    updateListAuto() {
      if (this.autoUpdate) {
        this.$store.commit('nodes/PROFITS_UPDATED')
      }
    },
    generateRegionImage(region) {
      return require(`~/assets/img/tools/general/REGION_${region}.png`)
    },
    selectRegion(region) {
      if (region === this.selectedRegion) {
        this.selectedRegion = ''
      } else {
        this.selectedRegion = region
      }
    },
    handleLinking() {
      if (this.unlinkingActive) {
        // Stop unlinking on same stop button
        this.$store.commit('nodes/HANDLE_UNLINKING', false)
      } else {
        // Toggle linking on link button
        this.$store.commit('nodes/HANDLE_LINKING', !this.linkingActive)
      }
    },
    startUnlink() {
      this.$store.commit('nodes/HANDLE_UNLINKING', true)
    },
    handleTaking() {
      this.$store.commit('nodes/HANDLE_TAKING', !this.takingActive)
    },
    parseValue(v) {
      return Parser.parseValue(v)
    },
    async saveNodes() {
      this.saving = true
      await Promise.all([
        this.$axios.$post('/user/userNodes', {
          nodes: [...this.getChangedNodes],
          headers: {
            Authorization: this.$auth.getToken('auth0'),
          },
        }),
        this.$axios.$post('/user/materialPreferences', {
          materials: {
            disabled: JSON.stringify(Array.from(this.disabledItems)),
            override: JSON.stringify(Array.from(this.overrideFloodedItems)),
          },
          headers: {
            Authorization: this.$auth.getToken('auth0'),
          },
        }),
      ])
        .then(() => {
          if (this.hasBackup) sessionStorage.removeItem('usernodes')
          this.saving = false
          this.$store.commit('nodes/SET_SAVESTATE', true)
        })
        .catch(() => {
          this.saving = false
          this.persistUsernodes()
          this.errorMessage =
            'Could not save nodes. Press OK to reload the page.'
          this.error = true
        })
    },
    persistUsernodes() {
      sessionStorage.setItem('usernodes', JSON.stringify(this.getChangedNodes))
    },
    restoreUsernodes() {
      try {
        const nodes = JSON.parse(sessionStorage.getItem('usernodes'))
        this.$store.commit('nodes/RESTORE_USERNODES', nodes)
      } catch (e) {
        sessionStorage.removeItem('usernodes')
      }
      sessionStorage.removeItem('usernodes')
    },
    reloadPage() {
      window.location.reload()
    },
  },
}
</script>
<style lang="scss" scoped>
.nodecalc-header {
  margin-bottom: 50px;
  border-bottom: 1px solid;
}
.nodecalc-filter-wrapper {
  position: sticky;
  top: 77px;
  background: $background-primary;
  z-index: 2;
  padding-top: 15px;
}

.materialfilter-wrapper {
  display: flex;
  align-items: center;
  margin-left: 15px;
}

.nodecalc-list {
  flex-wrap: wrap;
}
.filter-list {
  display: flex;
  flex-wrap: wrap;
  img {
    height: 70px;
  }
  &.hasSelected {
    .region-filter.off {
      img {
        filter: grayscale(100%);
      }
    }
  }
}
.region-filter {
  cursor: pointer;
}
.filter-additional {
  display: flex;
  align-items: center;
}
.fishing-filter {
  opacity: 0.4;
  &.active {
    opacity: 1;
  }
  svg {
    height: 35px;
    width: 35px;
  }
}

.linkingbuttons {
  position: relative;
  .unlinkbtn {
    position: absolute;
    top: -40px;
    left: 0;
    width: 93%;
    display: none;
  }
  &:hover {
    .unlinkbtn {
      display: block;
    }
  }
}

.node-options-wrapper {
  position: absolute;
  top: -185px;
  background: #162637;
  padding: 20px 10px;
  width: 250px;
  display: flex;
  align-items: center;
  flex-direction: column;
}

@media (max-width: 1535px) {
  .node-wrapper.column.is-one-third {
    width: 50%;
  }
}

@media (max-width: 768px) {
  .node-wrapper.column.is-one-third {
    width: 100%;
  }
  .sort-options {
    padding-bottom: 10px;
  }
  .materialUpdate,
  .nodecalc-filter-wrapper {
    display: block !important;
  }
  .node-options-wrapper {
    bottom: -207px;
    top: auto;
  }
}

.summary-fixed {
  background: $grey-blue;
  border: 1px solid white;
  position: fixed;
  bottom: 12px;
  right: 77px;
  padding: 20px;
  z-index: 10000;
}
</style>
<style lang="scss">
$nodelinks-colors: white, $blue, $red, $yellow, $purple, #57889b, #a14a64,
  #c5be59, #a31a5f, #4fce99, #5081af, #b93daf, #e0925d, #2d9299 #9b3535, green,
  purple, #4fec98 orange, cyan, aquamarine, brown, darkGoldenRod, crimson,
  darkOrchid, lightseagreen, moccasin, olivedrab;
.node-wrapper.grouped {
  border: 2px solid;
  @for $i from 1 through length($nodelinks-colors) {
    &.group-#{$i} {
      border-color: nth($nodelinks-colors, $i);
    }
  }
}

.node-name {
  color: #ffffff;
}

.node-wrapper.taken {
  .node-name {
    color: #aeff5c;
  }
}
.node-wrapper.maxWorker {
  &::before {
    content: '';
    position: absolute;
    width: 0;
    height: 0;
    border-top: 10px solid $yellow;
    border-right: 10px solid transparent;
    border-left: 10px solid transparent;
  }
}
</style>

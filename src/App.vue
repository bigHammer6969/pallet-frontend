<template>
  <div class="pallet-page">
    <div class="container-xxl py-3">

      <!-- 頂部：標題 + Summary + 棧板設定 + 棧板切換 -->
      <header class="pallet-header mb-2">
        <!-- Summary 列：壓縮一點高度 -->
        <div class="header-summary-row mb-2">
          <div class="summary-item">
            <div class="summary-label">棧板尺寸</div>
            <div class="summary-value">
              {{ palletLength }} × {{ palletWidth }}
            </div>
            <div class="summary-sub">最大高 {{ maxHeight }} cm</div>
          </div>

          <div class="summary-item">
            <div class="summary-label">棧板資訊</div>
            <div class="summary-value">
              第 {{ (currentPalletIndex || 0) + 1 }} 塊 / 共 {{ pallets.length || 0 }} 塊
            </div>
            <div class="summary-sub">
              本棧板：{{ currentPallet ? currentPallet.totalBoxes : 0 }} 箱
            </div>
          </div>

          <div class="summary-item">
            <div class="summary-label">總箱數</div>
            <div class="summary-value">
              {{ totalBoxes }}
            </div>
            <div class="summary-sub">
              總重 {{ totalWeight.toFixed(1) }} kg
            </div>
          </div>

          <div
            class="summary-item"
            :class="{ 'summary-item-warning': warningMessage }"
          >
            <div class="summary-label">最高堆疊</div>
            <div class="summary-value">
              {{ actualHeight }} cm
            </div>
            <div class="summary-sub">
              {{ warningMessage || '堆疊條件正常' }}
            </div>
          </div>
        </div>

        <!-- 棧板設定 + 切換 + 視角控制 -->
        <div class="d-flex flex-wrap align-items-end gap-3 header-bottom-row">

          <!-- 棧板設定（變瘦一點） -->
          <div class="header-pallet-config flex-grow-1">
            <div class="row g-2 align-items-end">
              <div class="col-4">
                <label class="form-label form-label-sm">棧板長</label>
                <input
                  type="number"
                  v-model.number="palletLength"
                  class="form-control form-control-sm"
                  min="0"
                />
              </div>
              <div class="col-4">
                <label class="form-label form-label-sm">棧板寬</label>
                <input
                  type="number"
                  v-model.number="palletWidth"
                  class="form-control form-control-sm"
                  min="0"
                />
              </div>
              <div class="col-4">
                <label class="form-label form-label-sm">最大堆疊高度</label>
                <input
                  type="number"
                  v-model.number="maxHeight"
                  class="form-control form-control-sm"
                  min="0"
                />
              </div>
            </div>
          </div>

          <!-- 棧板切換 + 視角控制 -->
          <div class="header-3d-controls d-flex flex-wrap align-items-center gap-2">
            <nav
              v-if="pallets && pallets.length > 1"
              class="pallet-switch-nav"
              aria-label="棧板切換"
            >
              <ul class="pagination pagination-sm mb-0">
                <li class="page-item disabled">
                  <span class="page-link pallet-label">棧板</span>
                </li>

                <li
                  class="page-item"
                  :class="{ disabled: currentPalletIndex === 0 }"
                >
                  <button
                    type="button"
                    class="page-link"
                    @click="prevPallet"
                  >
                    ‹
                  </button>
                </li>

                <li
                  v-for="item in palletPageItems"
                  :key="item.key"
                  class="page-item"
                  :class="{
                    active: item.type === 'page' && item.index === currentPalletIndex,
                    disabled: item.type === 'ellipsis'
                  }"
                >
                  <button
                    v-if="item.type === 'page'"
                    type="button"
                    class="page-link"
                    @click="onPalletChipClick(item.index)"
                  >
                    {{ item.index + 1 }}
                  </button>
                  <span
                    v-else
                    class="page-link"
                  >
                    …
                  </span>
                </li>

                <li
                  class="page-item"
                  :class="{ disabled: currentPalletIndex === pallets.length - 1 }"
                >
                  <button
                    type="button"
                    class="page-link"
                    @click="nextPallet"
                  >
                    ›
                  </button>
                </li>
              </ul>
            </nav>

            <button
              type="button"
              class="btn btn-outline-secondary btn-sm"
              @click="resetCamera"
            >
              重置視角
            </button>
            <button
              type="button"
              class="btn btn-outline-primary btn-sm"
              @click="resetSelection"
            >
              清除選取層
            </button>
          </div>
        </div>
      </header>

      <!-- 中間 + 下方：箱型橫排 + 3D 區 -->
      <div class="pallet-layout">
        <div class="pallet-layout-inner">

          <!-- 中間：箱型橫向一整排（固定高度較矮） -->
<section class="panel-card box-panel">
<div class="panel-header d-flex justify-content-between align-items-center box-header">
  <span class="fw-semibold">箱型設定</span>

  <div class="d-flex gap-2">
    <button
      type="button"
      class="btn btn-outline-primary btn-sm"
      @click="addBoxType"
    >
      新增箱型
    </button>

    <button
      type="button"
      class="btn btn-primary btn-sm"
      @click="onCalculate"
    >
      重新計算
    </button>
  </div>
</div>


  <div class="panel-body box-panel-body">
    <div class="text-muted mb-1" style="font-size: 0.78rem;">
      未填「箱數」的箱型不會參與排版與 3D 顯示。請至少設定一種「有箱數」的箱型。
    </div>

    <!-- ★★ 一整排橫向滾動的卡片軌道 ★★ -->
    <div class="box-card-track">
      <div
        class="box-card"
        v-for="(box, idx) in boxTypes"
        :key="box.id"
      >
        <div
          class="box-card-color"
          :style="{ backgroundColor: getTypeColorCss(idx) }"
        ></div>

        <div class="box-card-content">
          <div class="box-card-header">
            <input
              type="text"
              v-model="box.name"
              class="form-control form-control-sm box-name-input"
              :placeholder="'箱型 ' + String.fromCharCode(65 + idx)"
            />
            <button
              type="button"
              class="btn-delete-box"
              @click="removeBoxType(idx)"
              :disabled="boxTypes.length <= 1"
            >
              刪除
            </button>
          </div>

          <!-- 直向欄位 -->
          <div class="box-card-body">
            <div class="box-field">
              <label class="box-label">長 (cm)</label>
              <input
                type="number"
                v-model.number="box.length"
                class="form-control form-control-sm"
                min="0"
              />
            </div>

            <div class="box-field">
              <label class="box-label">寬 (cm)</label>
              <input
                type="number"
                v-model.number="box.width"
                class="form-control form-control-sm"
                min="0"
              />
            </div>

            <div class="box-field">
              <label class="box-label">高 (cm)</label>
              <input
                type="number"
                v-model.number="box.height"
                class="form-control form-control-sm"
                min="0"
              />
            </div>

            <div class="box-field">
              <label class="box-label">重量 (kg)</label>
              <div class="input-group input-group-sm">
                <input
                  type="number"
                  v-model.number="box.weight"
                  class="form-control form-control-sm"
                  min="0"
                  step="0.1"
                />
                <span class="input-group-text">kg</span>
              </div>
            </div>

            <div class="box-field">
              <label class="box-label">箱數</label>
              <input
                type="number"
                v-model.number="box.maxCount"
                class="form-control form-control-sm"
                min="0"
              />
            </div>
          </div>

          <div class="box-card-footer">
            <span class="small text-muted">
              {{ box.length || 0 }} × {{ box.width || 0 }} × {{ box.height || 0 }}
            </span>
            <span class="small text-muted" v-if="boxStats[idx]">
              已用：{{ boxStats[idx].boxes || 0 }} 箱
            </span>
          </div>
        </div>
      </div>
    </div>
  </div>
</section>


          <!-- 下方：3D 區，吃掉剩下高度 -->
          <section class="panel-card three-panel">
            <div class="panel-header">
             
            </div>
            <div class="panel-body three-panel-body">
              <div
                ref="threeContainer"
                class="three-container"
              ></div>
            </div>
          </section>

        </div>
      </div>

    </div>
  </div>
</template>

<script>
import * as THREE from "three";
import { markRaw } from "vue";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";

export default {
  name: "App",
data() {
  return {
    palletLength: 120,
    palletWidth: 100,
    maxHeight: 150,

    // 一進來就先給三張卡：A、B、C
    boxTypes: [
      {
        id: 1,
        name: "箱型 A",
        length: 0,
        width: 0,
        height: 0,
        weight: 0,
        maxCount: 0
      },
      {
        id: 2,
        name: "箱型 B",
        length: 0,
        width: 0,
        height: 0,
        weight: 0,
        maxCount: 0
      },
      {
        id: 3,
        name: "箱型 C",
        length: 0,
        width: 0,
        height: 0,
        weight: 0,
        maxCount: 0
      }
    ],
    nextBoxId: 4,

    // 下面照舊
    pallets: [],
    currentPalletIndex: 0,
    boxStats: [],
    totalLayers: 0,
    totalBoxes: 0,
    totalWeight: 0,
    actualHeight: 0,
    remainingSummary: [],
    scene: null,
    camera: null,
    renderer: null,
    controls: null,
    animationId: null,
    group: null,
    sectionGroup: null,
    raycaster: null,
    pointer: null,
    boxMeshes: [],
    selectedLayer: -1,
    _onPointerDown: null
  };
},
  computed: {
    warningMessage() {
      if (
        this.palletLength <= 0 ||
        this.palletWidth <= 0 ||
        this.maxHeight <= 0
      ) {
        return "請輸入有效的棧板長、寬與最大堆疊高度。";
      }
      return "請至少設定一種『有箱數』的箱型（長、寬、高與箱數）。";
    },

    currentPallet() {
      if (!this.pallets || this.pallets.length === 0) return null;
      const idx = this.currentPalletIndex || 0;
      return this.pallets[Math.min(idx, this.pallets.length - 1)];
    },

    currentLayers() {
      const p = this.currentPallet;
      return p && p.layerInfo ? p.layerInfo : [];
    },

    palletPageItems() {
      const total = this.pallets ? this.pallets.length : 0;
      const current = this.currentPalletIndex || 0;
      const items = [];

      if (total <= 1) return items;

      const maxVisible = 7; // 總共最多顯示幾個數字（不含 ‹ › 棧板）

      // 棧板數量少：全部列出
      if (total <= maxVisible) {
        for (let i = 0; i < total; i++) {
          items.push({
            type: "page",
            index: i,
            key: "page-" + i
          });
        }
        return items;
      }

      // 棧板多：頭尾 + 中間窗口 + ...
      const first = 0;
      const last = total - 1;

      const addPage = (idx) => {
        items.push({
          type: "page",
          index: idx,
          key: "page-" + idx
        });
      };

      const addEllipsis = (pos) => {
        items.push({
          type: "ellipsis",
          key: "ellipsis-" + pos
        });
      };

      // 先加第一頁
      addPage(first);

      // 以 current 為中心的窗格
      let start = current - 1;
      let end = current + 1;

      // 不要吃掉第一頁
      if (start < 1) {
        start = 1;
        end = start + 2;
      }
      // 不要吃掉最後一頁
      if (end > last - 1) {
        end = last - 1;
        start = end - 2;
      }
      if (start < 1) start = 1;

      // 左邊需要 ... 嗎？
      if (start > 1) {
        addEllipsis("left");
      }

      // 中間的連續頁碼
      for (let i = start; i <= end; i++) {
        addPage(i);
      }

      // 右邊需要 ... 嗎？
      if (end < last - 1) {
        addEllipsis("right");
      }

      // 最後一頁
      addPage(last);

      return items;
    }
  },
  mounted() {
    this.initThree();
    this.onCalculate();
    window.addEventListener("resize", this.onWindowResize);
  },
  beforeUnmount() {
    window.removeEventListener("resize", this.onWindowResize);

    if (this.renderer && this._onPointerDown) {
      this.renderer.domElement.removeEventListener(
        "pointerdown",
        this._onPointerDown
      );
    }

    if (this.animationId) cancelAnimationFrame(this.animationId);

    this.clearMainGroup();
    this.clearSectionGroup();

    if (this.renderer) {
      this.renderer.dispose();
      this.renderer = null;
    }
  },
  methods: {
    onCalculate() {
      this.resetSelection();
      this.currentPalletIndex = 0;
      this.recalcLayout();
      this.rebuildScene();
    },

    resetCamera() {
      if (!this.camera || !this.controls) return;
      this.camera.position.set(3.8, 4.2, 5.0);
      this.controls.target.set(0, 0.8, 0);
      this.controls.update();
    },

    resetSelection() {
      this.selectedLayer = -1;
      this.resetHighlight();
      this.clearSectionGroup();
    },

    onPalletChipClick(idx) {
      this.currentPalletIndex = idx;
      this.onPalletChange && this.onPalletChange();
    },
    prevPallet() {
      if (!this.pallets || this.pallets.length === 0) return;
      const total = this.pallets.length;
      this.currentPalletIndex =
        (this.currentPalletIndex - 1 + total) % total;
      this.onPalletChange && this.onPalletChange();
    },
    nextPallet() {
      if (!this.pallets || this.pallets.length === 0) return;
      const total = this.pallets.length;
      this.currentPalletIndex =
        (this.currentPalletIndex + 1) % total;
      this.onPalletChange && this.onPalletChange();
    },

    addBoxType() {
      const labelIndex = this.nextBoxId - 1; // id=1 對應 A, id=2 對應 B ...
      const baseCode = "A".charCodeAt(0);
      const labelCharCode = baseCode + (labelIndex % 26);
      const label = String.fromCharCode(labelCharCode);

      this.boxTypes.push({
        id: this.nextBoxId++,
        name: `箱型 ${label}`,
        length: 0,
        width: 0,
        height: 0,
        weight: 0,
        maxCount: 0
      });

      this.$nextTick(() => {
        this.onWindowResize();
      });
    },

    removeBoxType(index) {
      if (this.boxTypes.length <= 1) return;
      this.boxTypes.splice(index, 1);

      this.$nextTick(() => {
        this.onWindowResize();
      });
    },

    onPalletChange() {
      this.resetSelection();
      this.rebuildScene();
    },

    // 顏色：給 three.js 用
    getTypeColor(index) {
      const palette = [
        0x3498db, // 藍
        0xe67e22, // 橘
        0x9b59b6, // 紫
        0x2ecc71, // 綠
        0xf1c40f, // 黃
        0xe74c3c, // 紅
        0x1abc9c  // 青綠
      ];
      if (index < palette.length) {
        return new THREE.Color(palette[index]);
      }
      const hue = (index * 0.15) % 1;
      return new THREE.Color().setHSL(hue, 0.6, 0.55);
    },

    // 顏色：給 legend 用
    getTypeColorCss(index) {
      const c = this.getTypeColor(index);
      return `#${c.getHexString()}`;
    },

    // 多棧板堆疊（教科書版：先純層，再混層）
    recalcLayout() {
      const palletL = Math.floor(Number(this.palletLength) || 0);
      const palletW = Math.floor(Number(this.palletWidth) || 0);
      const maxH = Number(this.maxHeight) || 0;

      if (palletL <= 0 || palletW <= 0 || maxH <= 0) {
        this.pallets = [];
        this.boxStats = [];
        this.totalLayers = 0;
        this.totalBoxes = 0;
        this.totalWeight = 0;
        this.actualHeight = 0;
        this.remainingSummary = [];
        return;
      }

      const typeCount = this.boxTypes.length;
      const stats = [];
      const types = [];
      const weights = [];
      const maxCounts = [];

      // 轉成內部使用結構
      for (let i = 0; i < typeCount; i++) {
        const box = this.boxTypes[i] || {};

        const length = Number(box.length) || 0;
        const width = Number(box.width) || 0;
        const height = Number(box.height) || 0;
        const weight = Number(box.weight) || 0;
        const maxCountRaw = Number(box.maxCount) || 0;

        // 沒寫箱數 / <=0：直接當作「不參與排版」
        const hasCount = maxCountRaw > 0;

        const valid =
          hasCount &&
          length > 0 &&
          width > 0 &&
          height > 0 &&
          length <= palletL &&
          width <= palletW &&
          height <= maxH;

        stats.push({
          perRow: 0,
          perCol: 0,
          perLayer: 0,
          layers: 0,
          boxes: 0,
          weightSum: 0
        });

        if (!valid) {
          types.push(null);
          weights.push(0);
          maxCounts.push(0);
          continue;
        }

        const gridW = length;
        const gridH = width;

        types.push({
          index: i,
          length,
          width,
          height,
          gridW,
          gridH
        });
        weights.push(weight);
        maxCounts.push(maxCountRaw);
      }

      const anyUsable = types.some((t) => t && t.length > 0);
      if (!anyUsable) {
        this.pallets = [];
        this.boxStats = [];
        this.totalLayers = 0;
        this.totalBoxes = 0;
        this.totalWeight = 0;
        this.actualHeight = 0;
        this.remainingSummary = [];
        return;
      }

      // 給剩餘箱型做貪婪時用的優先順序（體積越大、越優先）
      const typePriorityRank = new Array(typeCount).fill(typeCount);
      const tmpPriority = [];
      for (let i = 0; i < typeCount; i++) {
        const t = types[i];
        if (!t) continue;
        const area = t.gridW * t.gridH;
        const volume = area * t.height;
        tmpPriority.push({ idx: i, volume });
      }
      tmpPriority.sort((a, b) => b.volume - a.volume); // 體積大在前
      tmpPriority.forEach((t, rank) => {
        typePriorityRank[t.idx] = rank;
      });

      const remainCount = maxCounts.slice();

      const pallets = [];
      const totalUsedPerType = new Array(typeCount).fill(0);
      const layerBaseSetPerType = new Array(typeCount)
        .fill(0)
        .map(() => new Set());

      const maxPallets = Infinity;

      // 單棧板排版：先堆「純層 perfect tiler」，再 greedy 填洞
      const packOnePallet = (remainArr) => {
        const cols = palletL;
        const rows = palletW;

        // 每個格子記錄目前高度
        const heightMap = new Array(cols);
        for (let x = 0; x < cols; x++) {
          heightMap[x] = new Array(rows).fill(0);
        }

        const placements = [];
        const usedPerType = new Array(typeCount).fill(0);
        const layerBasesPerTypeLocal = new Array(typeCount)
          .fill(0)
          .map(() => new Set());
        let maxHeightUsed = 0;

        // 先找「可以整層鋪平」的箱型（perfect tiler）
        const perfectOptions = [];
        for (let i = 0; i < typeCount; i++) {
          const t = types[i];
          if (!t) continue;

          if (cols % t.length === 0 && rows % t.width === 0) {
            const perRow = cols / t.length;
            const perCol = rows / t.width;
            const perLayerCount = perRow * perCol;
            const layerH = t.height;

            if (perLayerCount <= 0 || layerH <= 0) continue;

            const maxLayersByHeight = Math.floor(maxH / layerH);
            const maxLayersByCount = Math.floor(
              (remainArr[i] || 0) / perLayerCount
            );

            const layersPossible = Math.min(
              maxLayersByHeight,
              maxLayersByCount
            );

            if (layersPossible > 0) {
              perfectOptions.push({
                typeIndex: i,
                perRow,
                perCol,
                perLayerCount,
                layerH,
                layersPossible
              });
            }
          }
        }

        // 冠軍箱型
        perfectOptions.sort((a, b) => {
          if (b.perLayerCount !== a.perLayerCount) {
            return b.perLayerCount - a.perLayerCount;
          }
          return a.layerH - b.layerH;
        });

        // 階段 1：先堆純層
        if (perfectOptions.length > 0) {
          const champion = perfectOptions[0];
          const tIndex = champion.typeIndex;
          const t = types[tIndex];

          const layersToBuild = champion.layersPossible;
          const layerH = champion.layerH;
          const perRow = champion.perRow;
          const perCol = champion.perCol;
          const perLayerCount = champion.perLayerCount;

          for (let layerIdx = 0; layerIdx < layersToBuild; layerIdx++) {
            const base = layerIdx * layerH;

            for (let r = 0; r < perCol; r++) {
              const gy = r * t.width;
              for (let c = 0; c < perRow; c++) {
                const gx = c * t.length;

                placements.push({
                  typeIndex: tIndex,
                  gx,
                  gy,
                  gw: t.length,
                  gh: t.width,
                  height: layerH,
                  base
                });

                for (let x = gx; x < gx + t.length; x++) {
                  const colArr = heightMap[x];
                  for (let y = gy; y < gy + t.width; y++) {
                    colArr[y] = base + layerH;
                  }
                }
              }
            }

            usedPerType[tIndex] += perLayerCount;
            layerBasesPerTypeLocal[tIndex].add(base);
            maxHeightUsed = base + layerH;
          }
        }

        // 階段 2：greedy 填洞
        const getPlacementScore = (typeIndex, base) => {
          const t = types[typeIndex];
          if (!t) return -Infinity;

          const area = t.gridW * t.gridH;
          const volume = area * t.height;
          const priority = typePriorityRank[typeIndex] ?? typeCount;

          return (
            volume * 100000 +
            area * 1000 +
            (typeCount - priority) * 5000 +
            -base * 50
          );
        };

        const canPlaceBox = (typeIndex, gx, gy) => {
          const t = types[typeIndex];
          if (!t) return false;
          const gw = t.gridW;
          const gh = t.gridH;
          const h = t.height;

          if (gx + gw > cols || gy + gh > rows) return false;

          const base = heightMap[gx][gy];
          for (let x = gx; x < gx + gw; x++) {
            const colArr = heightMap[x];
            for (let y = gy; y < gy + gh; y++) {
              if (colArr[y] !== base) return false;
            }
          }

          if (base + h > maxH) return false;

          const limit = remainArr[typeIndex] || 0;
          const alreadyUsed = usedPerType[typeIndex] || 0;
          if (alreadyUsed >= limit) {
            return false;
          }

          return true;
        };

        const placeBox = (typeIndex, gx, gy) => {
          const t = types[typeIndex];
          const gw = t.gridW;
          const gh = t.gridH;
          const h = t.height;
          const base = heightMap[gx][gy];

          placements.push({
            typeIndex,
            gx,
            gy,
            gw,
            gh,
            height: h,
            base
          });

          for (let x = gx; x < gx + gw; x++) {
            const colArr = heightMap[x];
            for (let y = gy; y < gy + gh; y++) {
              colArr[y] += h;
            }
          }

          usedPerType[typeIndex] += 1;
          layerBasesPerTypeLocal[typeIndex].add(base);
          if (base + h > maxHeightUsed) maxHeightUsed = base + h;
        };

        const maxPlacementGlobal = 5000;
        let placementCount = 0;

        while (placementCount < maxPlacementGlobal) {
          const baseSet = new Set();
          for (let x = 0; x < cols; x++) {
            const colArr = heightMap[x];
            for (let y = 0; y < rows; y++) {
              const v = colArr[y];
              if (v < maxH) baseSet.add(v);
            }
          }

          if (baseSet.size === 0) break;

          const bases = Array.from(baseSet).sort((a, b) => a - b);
          let placedInThisRound = false;

          for (let bIndex = 0; bIndex < bases.length; bIndex++) {
            const base = bases[bIndex];
            let best = null;

            for (let gx = 0; gx < cols; gx++) {
              for (let gy = 0; gy < rows; gy++) {
                if (heightMap[gx][gy] !== base) continue;
                if (base >= maxH) continue;

                for (let tIndex = 0; tIndex < typeCount; tIndex++) {
                  if (!types[tIndex]) continue;
                  if (!canPlaceBox(tIndex, gx, gy)) continue;

                  const score = getPlacementScore(tIndex, base);
                  if (!best || score > best.score) {
                    best = { tIndex, gx, gy, score };
                  }
                }
              }
            }

            if (best) {
              placeBox(best.tIndex, best.gx, best.gy);
              placementCount++;
              placedInThisRound = true;
              break;
            }
          }

          if (!placedInThisRound) break;
        }

        if (placements.length === 0) {
          return null;
        }

        let totalBoxes = 0;
        let totalWeight = 0;
        for (let i = 0; i < typeCount; i++) {
          const count = usedPerType[i];
          totalBoxes += count;
          totalWeight += count * (weights[i] || 0);
        }

        const baseHeightsSet = new Set();
        for (const p of placements) {
          baseHeightsSet.add(p.base);
        }
        const baseHeights = Array.from(baseHeightsSet).sort(
          (a, b) => a - b
        );

        const layerInfo = baseHeights.map((base, idx) => {
          const boxes = placements
            .filter((p) => p.base === base)
            .map((p) => ({
              typeIndex: p.typeIndex,
              x: p.gx,
              y: p.gy,
              w: p.gw,
              h: p.gh,
              height: p.height,
              base: p.base
            }));
          return {
            globalLayer: idx,
            baseHeight: base,
            boxes
          };
        });

        return {
          placements,
          usedPerType,
          layerBasesPerTypeLocal,
          maxHeightUsed,
          totalBoxes,
          totalWeight,
          layerInfo
        };
      };

      // 多棧板：一塊一塊排
      for (let p = 0; p < maxPallets; p++) {
        const anyLeft = remainCount.some((cnt) => (cnt || 0) > 0);
        if (!anyLeft) break;

        const result = packOnePallet(remainCount);
        if (!result || !result.placements.length) break;

        pallets.push({
          layerInfo: result.layerInfo,
          totalBoxes: result.totalBoxes,
          totalWeight: result.totalWeight,
          actualHeight: result.maxHeightUsed
        });

        for (let i = 0; i < typeCount; i++) {
          const used = result.usedPerType[i] || 0;
          remainCount[i] = Math.max((remainCount[i] || 0) - used, 0);
          totalUsedPerType[i] += used;
          result.layerBasesPerTypeLocal[i].forEach((b) => {
            layerBaseSetPerType[i].add(b);
          });
        }
      }

      if (!pallets.length) {
        this.pallets = [];
        this.boxStats = [];
        this.totalLayers = 0;
        this.totalBoxes = 0;
        this.totalWeight = 0;
        this.actualHeight = 0;
        this.remainingSummary = [];
        return;
      }

      let totalBoxesAll = 0;
      let totalWeightAll = 0;
      let maxHeightAll = 0;
      let totalLayersAll = 0;

      for (const p of pallets) {
        totalBoxesAll += p.totalBoxes;
        totalWeightAll += p.totalWeight;
        if (p.actualHeight > maxHeightAll) maxHeightAll = p.actualHeight;
        totalLayersAll += p.layerInfo ? p.layerInfo.length : 0;
      }

      const boxStats = [];
      for (let i = 0; i < typeCount; i++) {
        const weight = weights[i] || 0;
        const boxes = totalUsedPerType[i];
        const layersCount = layerBaseSetPerType[i].size;
        const weightSum = boxes * weight;

        boxStats.push({
          perRow: 0,
          perCol: 0,
          perLayer: layersCount > 0 ? boxes / layersCount : 0,
          layers: layersCount,
          boxes,
          weightSum
        });
      }

      const remainingSummary = [];
      for (let i = 0; i < typeCount; i++) {
        const maxCountRaw = Number(this.boxTypes[i].maxCount) || 0;
        if (maxCountRaw > 0) {
          const remaining = Math.max(
            maxCountRaw - totalUsedPerType[i],
            0
          );
          remainingSummary.push({
            index: i,
            name: this.boxTypes[i].name,
            remaining
          });
        }
      }

      this.pallets = pallets;
      this.boxStats = boxStats;
      this.totalLayers = totalLayersAll;
      this.totalBoxes = totalBoxesAll;
      this.totalWeight = totalWeightAll;
      this.actualHeight = maxHeightAll;
      this.remainingSummary = remainingSummary;
    },

    // three.js 初始化
    initThree() {
      const container = this.$refs.threeContainer;
      const width = container.clientWidth || 600;
      const height = container.clientHeight || 400;

      this.scene = markRaw(new THREE.Scene());
      this.scene.background = new THREE.Color(0xf5f7fb);
      this.scene.fog = new THREE.Fog(0xf5f7fb, 6, 20);

      this.camera = markRaw(
        new THREE.PerspectiveCamera(45, width / height, 0.1, 1000)
      );
      this.camera.position.set(3.8, 4.2, 5.0);
      this.camera.lookAt(0, 0.8, 0);

      this.renderer = markRaw(
        new THREE.WebGLRenderer({ antialias: true })
      );
      this.renderer.setSize(width, height);
      this.renderer.setPixelRatio(window.devicePixelRatio || 1);
      this.renderer.shadowMap.enabled = true;
      container.appendChild(this.renderer.domElement);

      this.controls = markRaw(
        new OrbitControls(this.camera, this.renderer.domElement)
      );
      this.controls.enableDamping = true;
      this.controls.dampingFactor = 0.1;
      this.controls.target.set(0, 0.8, 0);
      this.controls.maxPolarAngle = Math.PI * 0.49;
      this.controls.minDistance = 3;
      this.controls.maxDistance = 20;

      const ambientLight = new THREE.AmbientLight(0xffffff, 0.8);
      this.scene.add(ambientLight);

      const hemiLight = new THREE.HemisphereLight(
        0xffffff,
        0xdedede,
        0.5
      );
      this.scene.add(hemiLight);

      const dirLight = new THREE.DirectionalLight(0xffffff, 0.8);
      dirLight.position.set(8, 10, 6);
      dirLight.castShadow = true;
      dirLight.shadow.mapSize.set(1024, 1024);
      this.scene.add(dirLight);

      const grid = new THREE.GridHelper(20, 20, 0xd0d4e4, 0xe0e4f0);
      grid.position.y = 0;
      this.scene.add(grid);

      this.raycaster = markRaw(new THREE.Raycaster());
      this.pointer = new THREE.Vector2();

      this._onPointerDown = this.onPointerDown.bind(this);
      this.renderer.domElement.addEventListener(
        "pointerdown",
        this._onPointerDown
      );

      const animate = () => {
        this.animationId = requestAnimationFrame(animate);
        if (this.controls) this.controls.update();
        if (this.renderer && this.scene && this.camera) {
          this.renderer.render(this.scene, this.camera);
        }
      };
      animate();
    },

    clearMainGroup() {
      if (!this.group) return;
      this.scene.remove(this.group);
      this.group.traverse((obj) => {
        if (obj.geometry && obj.geometry.dispose) obj.geometry.dispose();
        if (obj.material) {
          if (Array.isArray(obj.material)) {
            obj.material.forEach(
              (m) => m && m.dispose && m.dispose()
            );
          } else if (obj.material.dispose) {
            obj.material.dispose();
          }
        }
      });
      this.group = null;
      this.boxMeshes = [];
    },

    clearSectionGroup() {
      if (!this.sectionGroup) return;
      this.scene.remove(this.sectionGroup);
      this.sectionGroup.traverse((obj) => {
        if (obj.geometry && obj.geometry.dispose) obj.geometry.dispose();
        if (obj.material && obj.material.dispose) obj.material.dispose();
      });
      this.sectionGroup = null;
    },

    rebuildScene() {
      if (!this.scene) return;

      this.clearMainGroup();
      this.clearSectionGroup();

      const layers = this.currentLayers;
      if (!layers || !layers.length) {
        return;
      }

      this.group = markRaw(new THREE.Group());
      this.scene.add(this.group);

      const scale = 0.01;

      const palletLen = this.palletLength * scale;
      const palletWid = this.palletWidth * scale;
      const palletThk = 0.05;

      // 棧板本體
      const palletGeo = new THREE.BoxGeometry(
        palletLen,
        palletThk,
        palletWid
      );
      const palletMat = new THREE.MeshPhongMaterial({ color: 0x8b5a2b });
      const palletMesh = new THREE.Mesh(palletGeo, palletMat);
      palletMesh.position.set(0, palletThk / 2, 0);
      palletMesh.receiveShadow = true;
      this.group.add(palletMesh);

      for (let layerIdx = 0; layerIdx < layers.length; layerIdx++) {
        const layer = layers[layerIdx];
        if (!layer || !layer.boxes || !layer.boxes.length) continue;

        for (let i = 0; i < layer.boxes.length; i++) {
          const b = layer.boxes[i];
          const tIndex = b.typeIndex;
          const boxType = this.boxTypes[tIndex];
          if (!boxType) continue;

          const boxHeightCm =
            Number(b.height) || Number(boxType.height) || 0;
          if (boxHeightCm <= 0) continue;

          const boxLenWorld = b.w * scale;
          const boxWidWorld = b.h * scale;
          const boxHeiWorld = boxHeightCm * scale;

          const boxGeo = new THREE.BoxGeometry(
            boxLenWorld,
            boxHeiWorld,
            boxWidWorld
          );

          const colorObj = this.getTypeColor(tIndex);
          const boxMat = new THREE.MeshPhongMaterial({
            color: colorObj,
            transparent: true,
            opacity: 0.95
          });

          const worldX = (b.x + b.w / 2) * scale - palletLen / 2;
          const worldZ = (b.y + b.h / 2) * scale - palletWid / 2;
          const worldY =
            palletThk + b.base * scale + boxHeiWorld / 2;

          const mesh = new THREE.Mesh(boxGeo, boxMat);
          mesh.position.set(worldX, worldY, worldZ);
          mesh.userData = {
            globalLayer: layerIdx,
            typeIndex: tIndex
          };
          mesh.castShadow = true;
          mesh.receiveShadow = true;
          this.group.add(mesh);
          this.boxMeshes.push(mesh);

          const edgesGeo = new THREE.EdgesGeometry(boxGeo);
          const edgeMat = new THREE.LineBasicMaterial({
            color: colorObj
          });
          const edgeLines = new THREE.LineSegments(
            edgesGeo,
            edgeMat
          );
          edgeLines.position.copy(mesh.position);
          this.group.add(edgeLines);
        }
      }

      this.group.position.y = 0.05;
      this.resetHighlight();
    },

    resetHighlight() {
      this.boxMeshes.forEach((box) => {
        const tIndex = box.userData.typeIndex ?? 0;
        const colorObj = this.getTypeColor(tIndex);
        box.material.color.set(colorObj);
        box.material.opacity = 0.95;
      });
    },

    onPointerDown(event) {
      if (!this.raycaster || !this.boxMeshes.length) return;

      const rect = this.renderer.domElement.getBoundingClientRect();
      this.pointer.x =
        ((event.clientX - rect.left) / rect.width) * 2 - 1;
      this.pointer.y =
        -((event.clientY - rect.top) / rect.height) * 2 + 1;

      this.raycaster.setFromCamera(this.pointer, this.camera);
      const intersects = this.raycaster.intersectObjects(
        this.boxMeshes
      );

      if (intersects.length > 0) {
        const hit = intersects[0].object;
        const layerIndex = hit.userData.globalLayer ?? -1;
        if (layerIndex >= 0) {
          this.selectedLayer = layerIndex;
          this.highlightLayer(layerIndex);
          this.buildSectionView(layerIndex);
        }
      }
    },

    highlightLayer(layerIndex) {
      this.boxMeshes.forEach((box) => {
        const tIndex = box.userData.typeIndex ?? 0;
        const baseColor = this.getTypeColor(tIndex);

        if (box.userData.globalLayer === layerIndex) {
          box.material.color.set(0xffc107);
          box.material.opacity = 1;
        } else {
          box.material.color.set(baseColor);
          box.material.opacity = 0.25;
        }
      });
    },

    buildSectionView(layerIndex) {
      this.clearSectionGroup();

      const layers = this.currentLayers;
      const layer = layers[layerIndex];
      if (!layer || !layer.boxes || !layer.boxes.length) return;

      this.sectionGroup = markRaw(new THREE.Group());
      this.scene.add(this.sectionGroup);

      const scale = 0.01;
      const palletLenWorld = this.palletLength * scale;
      const palletWidWorld = this.palletWidth * scale;

      const offsetX = palletLenWorld * 1.4;

      for (let i = 0; i < layer.boxes.length; i++) {
        const b = layer.boxes[i];
        const tIndex = b.typeIndex;
        const boxType = this.boxTypes[tIndex];
        if (!boxType) continue;

        const boxHeightCm =
          Number(b.height) || Number(boxType.height) || 0;
        if (boxHeightCm <= 0) continue;

        const boxLenWorld = b.w * scale;
        const boxWidWorld = b.h * scale;
        const boxHeiWorld = boxHeightCm * scale;

        const boxGeo = new THREE.BoxGeometry(
          boxLenWorld,
          boxHeiWorld,
          boxWidWorld
        );

        const colorObj = this.getTypeColor(tIndex);
        const boxMat = new THREE.MeshPhongMaterial({
          color: colorObj,
          transparent: true,
          opacity: 0.9
        });

        const worldX =
          (b.x + b.w / 2) * scale -
          palletLenWorld / 2 +
          offsetX;
        const worldZ =
          (b.y + b.h / 2) * scale - palletWidWorld / 2;
        const worldY = boxHeiWorld / 2;

        const mesh = new THREE.Mesh(boxGeo, boxMat);
        mesh.position.set(worldX, worldY, worldZ);
        mesh.castShadow = true;
        mesh.receiveShadow = true;
        this.sectionGroup.add(mesh);

        const edgesGeo = new THREE.EdgesGeometry(boxGeo);
        const edgeMat = new THREE.LineBasicMaterial({
          color: colorObj
        });
        const edgeLines = new THREE.LineSegments(
          edgesGeo,
          edgeMat
        );
        edgeLines.position.copy(mesh.position);
        this.sectionGroup.add(edgeLines);
      }

      this.sectionGroup.position.y = 0.02;
    },

    onWindowResize() {
      if (!this.renderer || !this.camera) return;
      const container = this.$refs.threeContainer;
      const width = container.clientWidth || 600;
      const height = container.clientHeight || 400;

      this.camera.aspect = width / height;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(width, height);
    }
  }
};
</script>

<style scoped>
.pallet-page {
  min-height: 100vh;
  background: radial-gradient(circle at top left, #f4f7ff, #e9edf7 40%, #dde3f0 80%);
  display: flex;
  flex-direction: column;
}

.container-xxl {
  flex: 1 1 auto;
  display: flex;
  flex-direction: column;
}



.pallet-header {
  background: rgba(255, 255, 255, 0.96);
  border-radius: 14px;
  padding: 0.7rem 1rem;
  box-shadow: 0 8px 18px rgba(15, 23, 42, 0.08);
  backdrop-filter: blur(8px);
}

.header-summary-row {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 0.45rem;
  margin-bottom: 0.35rem;
}

.summary-item {
  background: #f6f7fb;
  border-radius: 10px;
  padding: 0.35rem 0.55rem;
  border: 1px solid rgba(148, 163, 184, 0.4);
}

.summary-item-warning {
  border-color: #f97316;
  background: #fff7ed;
}

.summary-label {
  font-size: 0.7rem;
  color: #64748b;
  margin-bottom: 0.08rem;
}

.summary-value {
  font-weight: 700;
  font-size: 0.95rem;
  color: #0f172a;
  line-height: 1.2;
}

.summary-sub {
  font-size: 0.7rem;
  color: #94a3b8;
  line-height: 1.2;
}

.pallet-layout {
  flex: 1 1 auto;
  margin-top: 0.5rem;
}

.pallet-layout-inner {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  height: 100%;
}

.panel-card {
  background: #ffffff;
  border-radius: 14px;
  border: 1px solid rgba(148, 163, 184, 0.35);
  box-shadow: 0 8px 20px rgba(15, 23, 42, 0.08);
  display: flex;
  flex-direction: column;
}

.panel-header {
  padding: 0.6rem 0.9rem;
  border-bottom: 1px solid rgba(226, 232, 240, 0.9);
  font-size: 0.9rem;
}

.panel-body {
  padding: 0.7rem 0.9rem 0.85rem;
}

.box-panel {
  flex: 0 0 260px;
}

.box-panel-body {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.box-card-track {
  flex: 1 1 auto;
  min-height: 0;
  display: flex;
  align-items: stretch;
  gap: 0.7rem;
  overflow-x: auto;
  overflow-y: hidden;
  padding: 0.3rem 0.1rem 0.2rem 0;
  flex-wrap: nowrap;
}

.box-card-track::-webkit-scrollbar {
  height: 6px;
}

.box-card-track::-webkit-scrollbar-track {
  background: transparent;
}

.box-card-track::-webkit-scrollbar-thumb {
  background: rgba(148, 163, 184, 0.9);
  border-radius: 999px;
}

.box-card-track::-webkit-scrollbar-thumb:hover {
  background: rgba(100, 116, 139, 0.95);
}

.box-card {
  flex: 1 1 calc(33.333% - 0.7rem);
  min-width: 200px;
  display: grid;
  grid-template-columns: 4px minmax(0, 1fr);
  border-radius: 10px;
  background: #f9fafb;
  border: 1px solid rgba(203, 213, 225, 0.95);
}

.box-card-track > .box-card:only-child {
  flex: 1 1 auto;
  max-width: none;
}

.box-card-color {
  border-radius: 10px 0 0 10px;
}

.box-card-content {
  padding: 0.4rem 0.55rem 0.5rem;
  display: flex;
  flex-direction: column;
  height: 100%;
}

.box-card-header {
  display: flex;
  align-items: center;
  gap: 0.45rem;
  margin-bottom: 0.3rem;
}

.box-name-input {
  font-size: 0.8rem;
}

.btn-delete-box {
  border: 1px solid #fecaca;
  background: #fef2f2;
  color: #b91c1c;
  border-radius: 999px;
  padding: 0.1rem 0.55rem;
  font-size: 0.72rem;
  line-height: 1.2;
  cursor: pointer;
  white-space: nowrap;
}

.btn-delete-box:hover:not(:disabled) {
  background: #fee2e2;
  border-color: #fca5a5;
  color: #991b1b;
}

.btn-delete-box:disabled {
  opacity: 0.5;
  cursor: default;
}

.box-card-body {
  font-size: 0.8rem;
  display: flex;
  flex-direction: column;
}

.box-field {
  display: flex;
  flex-direction: column;
  margin-bottom: 0.25rem;
}

.box-field:last-child {
  margin-bottom: 0;
}

.box-label {
  font-size: 0.7rem;
  color: #6b7280;
}

.box-card-footer {
  margin-top: 0.25rem;
  display: flex;
  justify-content: space-between;
  font-size: 0.7rem;
}

input[type="number"]::-webkit-outer-spin-button,
input[type="number"]::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

input[type="number"] {
  -moz-appearance: textfield;
}

.three-panel {
  flex: 1 1 auto;
}

.three-panel-body {
  display: flex;
  flex-direction: column;
  flex: 1 1 auto;
}

.three-container {
  width: 100%;
  flex: 1 1 auto;
  min-height: 260px;
  background: radial-gradient(circle at top, #f1f5f9, #e2e8f0);
  border-radius: 12px;
}

.pagination .page-link {
  border-radius: 999px !important;
}

.pagination {
  border-radius: 999px;
  padding: 2px 6px;
}

.pallet-label {
  border-radius: 999px !important;
}

.page-item.active .page-link {
  border-radius: 999px !important;
}

@media (max-width: 991.98px) {
  .header-summary-row {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .box-panel {
    flex: 0 0 260px;
  }

  .box-card {
    flex: 0 0 230px;
  }

  .three-container {
    min-height: 240px;
  }
}

@media (max-width: 575.98px) {
  .header-main {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.4rem;
  }

  .header-summary-row {
    grid-template-columns: 1fr;
  }

  .box-panel {
    flex: 0 0 280px;
  }

  .box-card-track {
    gap: 0.5rem;
  }

  .box-card {
    flex: 0 0 220px;
  }

  .three-container {
    min-height: 220px;
  }
}


</style>




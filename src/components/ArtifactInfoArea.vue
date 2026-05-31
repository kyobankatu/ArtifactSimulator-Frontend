<template>
  <div class="artifact-info-area">
    <p class="artifact-info-title">オプション選択</p>
    <div class="def-tabs">
      <button
        :class="['def-tab', { active: activeMode == 'old' }]"
        @click="switchMode('old')">
        ver5.8 以前
      </button>
      <button
        :class="['def-tab', { active: activeMode == 'new' }]"
        @click="switchMode('new')">
        Luna Ⅰ 以降
      </button>
    </div>
    <div v-if="activeMode == 'old'" class="artifact-info-inner-area">
      <div v-if="artifactImg">
        <img class="artifact-img" :src="artifactImg" alt="Selected Image" />
      </div>
      <div class="artifact-info-input-area">
        <div class="option-area">
          <div class="option-num-area info-section">
            <p>サブオプション数</p>
            <input class="visually-hidden" type="radio" name="option" value="3" id="option-3"
              :checked="option === 3"
              @change="$emit('update:option', 3)" />
            <label for="option-3">3オプション</label>
            <input class="visually-hidden" type="radio" name="option" value="4" id="option-4"
              :checked="option === 4"
              @change="$emit('update:option', 4)" />
            <label for="option-4">4オプション</label>
          </div>
          <div class="main-option-area info-section">
            <p>メインオプション</p>
            <select class="def-selecter" :value="position" @change="$emit('update:position', $event.target.value)">
              <option value="生の花">生の花</option>
              <option value="死の羽">死の羽</option>
              <option value="時の砂">時の砂</option>
              <option value="空の杯">空の杯</option>
              <option value="理の冠">理の冠</option>
            </select>
            <select class="def-selecter" :value="main_op" @change="$emit('update:main_op', $event.target.value)">
              <option
                v-for="option in filteredMainOptions"
                :key="option.value"
                :value="option.value">
                {{ option.text }}
              </option>
            </select>
          </div>
        </div>
        <div class="suboption-area info-section">
          <p>サブオプション</p>
          <div class="suboptions def-checkbox">
            <input type="checkbox" id="is-crit-rate" :checked="is_crit_rate" @change="$emit('update:is_crit_rate', $event.target.checked)" />
            <label for="is-crit-rate">会心率</label>
            <input type="checkbox" id="is-crit-dmg" :checked="is_crit_dmg" @change="$emit('update:is_crit_dmg', $event.target.checked)" />
            <label for="is-crit-dmg">会心ダメージ</label>
            <input type="checkbox" id="is-atk" :checked="is_atk" @change="$emit('update:is_atk', $event.target.checked)" />
            <label for="is-atk">攻撃力</label>
            <input type="checkbox" id="is-hp" :checked="is_hp" @change="$emit('update:is_hp', $event.target.checked)" />
            <label for="is-hp">HP</label>
            <input type="checkbox" id="is-em" :checked="is_em" @change="$emit('update:is_em', $event.target.checked)" />
            <label for="is-em">元素熟知</label>
          </div>
        </div>
        <div class="artifact-info-right-area">
          <div class="reinforce-num-area info-section">
            <p>強化回数</p>
            <select class="def-selecter" :value="count" @change="$emit('update:count', Number($event.target.value))">
              <option
                v-for="n in reinforceOptions"
                :key="n"
                :value="n">
                {{ n }}
              </option>
            </select>
          </div>
          <div class="level-area info-section">
            <p>レベル</p>
            <select class="def-selecter" :value="levelGroup" @change="$emit('update:level', Number($event.target.value))">
              <option value="0">0～3</option>
              <option value="4">4～7</option>
              <option value="8">8～11</option>
              <option value="12">12～15</option>
              <option value="16">16～19</option>
              <option value="20">20</option>
            </select>
          </div>
        </div>
      </div>
    </div>
    <div v-else class="artifact-info-inner-area">
      <div v-if="artifactImg">
        <img class="artifact-img" :src="artifactImg" alt="Selected Image" />
      </div>
      <div class="artifact-info-input-area">
        <div class="option-area">
          <div class="option-num-area info-section">
            <p>サブオプション数</p>
            <input class="visually-hidden" type="radio" name="option" value="3" id="option-3"
              :checked="option === 3"
              @change="$emit('update:option', 3)" />
            <label for="option-3">3オプション</label>
            <input class="visually-hidden" type="radio" name="option" value="4" id="option-4"
              :checked="option === 4"
              @change="$emit('update:option', 4)" />
            <label for="option-4">4オプション</label>
          </div>
          <div class="main-option-area info-section">
            <p>メインオプション</p>
            <select class="def-selecter" :value="position" @change="$emit('update:position', $event.target.value)">
              <option value="生の花">生の花</option>
              <option value="死の羽">死の羽</option>
              <option value="時の砂">時の砂</option>
              <option value="空の杯">空の杯</option>
              <option value="理の冠">理の冠</option>
            </select>
            <select class="def-selecter" :value="main_op" @change="$emit('update:main_op', $event.target.value)">
              <option
                v-for="option in filteredMainOptions"
                :key="option.value"
                :value="option.value">
                {{ option.text }}
              </option>
            </select>
          </div>
        </div>
        <div class="suboption-area info-section">
          <p>サブオプション</p>
          <div class="suboptions def-checkbox">
            <input type="checkbox" id="is-crit-rate" :checked="is_crit_rate" @change="$emit('update:is_crit_rate', $event.target.checked)" />
            <label for="is-crit-rate">会心率</label>
            <input type="checkbox" id="is-crit-dmg" :checked="is_crit_dmg" @change="$emit('update:is_crit_dmg', $event.target.checked)" />
            <label for="is-crit-dmg">会心ダメージ</label>
            <input type="checkbox" id="is-atk" :checked="is_atk" @change="$emit('update:is_atk', $event.target.checked)" />
            <label for="is-atk">攻撃力</label>
            <input type="checkbox" id="is-hp" :checked="is_hp" @change="$emit('update:is_hp', $event.target.checked)" />
            <label for="is-hp">HP</label>
            <input type="checkbox" id="is-em" :checked="is_em" @change="$emit('update:is_em', $event.target.checked)" />
            <label for="is-em">元素熟知</label>
          </div>
        </div>
        <div class="artifact-info-right-area">
          <div class="reinforce-num-area info-section">
            <p>強化回数</p>
            <select class="def-selecter" :value="count" @change="$emit('update:count', Number($event.target.value))">
              <option
                v-for="n in reinforceOptions"
                :key="n"
                :value="n">
                {{ n }}
              </option>
            </select>
          </div>
          <div class="level-area info-section">
            <p>レベル</p>
            <select class="def-selecter" :value="levelGroup" @change="$emit('update:level', Number($event.target.value))">
              <option value="0">0～3</option>
              <option value="4">4～7</option>
              <option value="8">8～11</option>
              <option value="12">12～15</option>
              <option value="16">16～19</option>
              <option value="20">20</option>
            </select>
          </div>
        </div>
      </div>
      <div class="active-option-area info-section">
        <p>アクティブ化待ちオプション(スコア)</p>
        <div class="active-option-inner-area">
          <select class="def-selecter" v-model="selected">
            <option
              v-for="opt in setActiveOptions"
              :key="opt.value"
              :value="opt.value"
            >
              {{ opt.text }}
            </option>
          </select>
          <div class="range-slider">
            <input
              id="active-op-slider"
              type="range"
              min="0"
              max="7.8"
              step="0.1"
              :value="activeValueFormatted"
              @input="updateActiveValue($event.target.value)"
            />
            <input
              class="active-op-value"
              type="number"
              min="0"
              max="7.8"
              step="0.1"
              :value="activeValueFormatted"
              @blur="updateActiveValue($event.target.value)"
            />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "ArtifactInfoArea",
  props: {
    artifactImg: String,
    option: Number,
    position: String,
    main_op: String,
    filteredMainOptions: Array,
    is_crit_rate: Boolean,
    is_crit_dmg: Boolean,
    is_atk: Boolean,
    is_hp: Boolean,
    is_em: Boolean,
    count: Number,
    level: Number,
    activeValueFormatted: String,
    activeOption: String,
    scoreType: String
  },
  data() {
    return {
      activeMode: "new",
      selected: "else",
    };
  },
  mounted() {
    this.selected = this.activeOption;
  },
  computed: {
    // アクティブオプションの選択肢
    setActiveOptions() {
      var options = [];
      if (!this.is_crit_rate) {
        options.push({ value: "crit-rate", text: "会心率" });
      }
      if (!this.is_crit_dmg) {
        options.push({ value: "crit-dmg", text: "会心ダメージ" });
      }
      if (!this.is_atk && this.scoreType == "atk") {
        options.push({ value: "atk%", text: "攻撃力" });
      }
      if (!this.is_hp && this.scoreType == "hp") {
        options.push({ value: "hp%", text: "HP" });
      }
      if (!this.is_em && this.scoreType == "em") {
        options.push({ value: "em%", text: "元素熟知" });
      }
      options.push({ value: "else", text: "その他" });

      return options;
    },
    // レベルのグループ化 (0-3, 4-7, 8-11, 12-15, 16-19, 20)
    levelGroup() {
      return Math.floor(this.level / 4) * 4;
    },
    // 強化回数の選択肢 (レベルに応じて最大値を変更)
    reinforceOptions() {
      const max = 6 - Math.floor(this.level / 4);
      return Array.from({ length: max - 1 }, (_, i) => i + 1);
    }
  },

  methods: {
    // タブ切り替え
    switchMode(mode) {
      this.activeMode = mode;
      this.$emit('update:activeMode', mode);
    },

    // 選択肢にselectedが含まれているか
    ensureSelectedValid() {
      const contains = this.setActiveOptions.some(o => o.value == this.selected);
      if (!contains) {
        this.selected = "else";
        this.$emit('update:activeOption', 'else');
      }
    },

    // アクティブオプションの値を更新した時に呼ばれる
    updateActiveValue(value) {
      let activeValue = parseFloat(value, 10);
      activeValue = Math.floor(activeValue * 10) / 10;
      if (isNaN(activeValue)) {
        activeValue = 0.0;
      } else if (activeValue < 0) {
        activeValue = 0.0;
      } else if (7.8 < activeValue) {
        activeValue = 7.8
      }

      const activeValueFormatted = activeValue.toFixed(1);
      this.$emit('updateActiveValue', {score: activeValue, formatted: activeValueFormatted});
    }
  },

  watch: {
    // activeOptionを更新したら同期
    activeOption(newVal) {
      this.selected = newVal;
      this.ensureSelectedValid();
    },
    // selectedが変わったら親に通知
    selected(newVal) {
      this.$emit('update:activeOption', newVal);
    },
    // scoreTypeが変わったらactiveOptionをリセット
    scoreType() {
      this.setActiveOptions;
      this.ensureSelectedValid();
    }
  }
}
</script>

<style scoped>

.artifact-info-area {
  margin-bottom: 20px;
  width: fit-content;
  max-width: 90%;

  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;

  background-color: #424858;
  border-radius: 5px;
  outline: 1px solid #d3bb8f;
  outline-offset: -5px;
}

.artifact-info-title {
  font-size: min(calc(14vw / 5), 20px);
  color: #d3bb8f;
  margin: min(calc(11vw / 5), 16px);
}

.artifact-info-inner-area {
  margin: 0px min(calc(14vw / 5), 20px) min(calc(14vw / 5), 20px);
  padding: min(calc(14vw / 5), 20px);
  width: fit-content;
  height: fit-content;

  background: #424858;
  border: #d3bb8f 1px solid;

  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;

  z-index: 1;
}

.artifact-info-title {
  font-size: min(calc(14vw / 5), 20px);
  color: #d3bb8f;
  margin: min(calc(11vw / 5), 16px);
}

.artifact-img {
  margin-bottom: min(calc(22vw / 5), 30px);

  width: min(calc(150vw / 5), 320px);
  height: auto;

  box-shadow: 5px 5px 5px rgb(0 0 0 / 70%);
}

.artifact-info-input-area {
  display: flex;
  flex-direction: row;
  justify-content: flex-start;
  gap: min(calc(22vw / 5), 30px);
}

.option-area {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  gap: min(calc(22vw / 5), 30px);
}

.option-num-area label {
  margin: 0px min(calc(9vw / 5), 16px) min(calc(9vw / 5), 16px);
  padding-left: min(calc(22vw / 5), 30px);

  position: relative;
  cursor: pointer;
  font-size: min(calc(9vw / 5), 16px);
  color: #fff;
  white-space: nowrap;
  transition: background-color 0.3s, transform 0.2s ease-in-out; /* ホバー時のアニメーション */
}

.option-num-area input[type="radio"] {
  position: absolute;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  width: min(calc(9vw / 5), 16px);
  height: min(calc(9vw / 5), 16px);
  opacity: 0;
  pointer-events: none;
}

.option-num-area p {
  padding: 0px min(calc(9vw / 5), 16px);

  font-size: min(calc(9vw / 5), 16px);
  color: #fff;
  white-space: nowrap;
}

/* option-num-areaのアニメーション */

.option-num-area label::before {
  content: "";
  position: absolute;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  width: min(calc(9vw / 5), 16px);
  height: min(calc(9vw / 5), 16px);
  border: 2px solid #fff;
  border-radius: 50%;
  background-color: transparent;
  transition: all 0.3s ease;
}

.option-num-area label::after {
  content: "";
  position: absolute;
  left: calc(2px + min(calc(9vw / 5), 16px) * 0.1);
  top: 50%;
  transform: translateY(-50%);
  width: calc(min(calc(9vw / 5), 16px) * 0.8);
  height: calc(min(calc(9vw / 5), 16px) * 0.8);
  border-radius: 50%;
  background-color: #fff;
  opacity: 0;
  transition: all 0.3s ease;
}

.option-num-area input[type="radio"]:checked + label::after {
  opacity: 1; /* チェック時に〇を表示 */
}

.option-num-area #option-3:not(:checked) + label::after {
  opacity: 0; /* 未選択状態で非表示 */
  transform: translateY(100%);
}

.option-num-area #option-4:not(:checked) + label::after {
  opacity: 0; /* 未選択状態で非表示 */
  transform: translateY(-100%);
}

.option-num-area input[type="radio"]:checked + label::before {
  background-color: #4B5368; /* チェック時の背景色 */
  transform: translateY(-50%) scale(1.2); /* チェック時に少し拡大 */
}

.option-num-area label:hover {
  background-color: rgba(255, 255, 255, 0); /* ホバー時の背景色 */
  transform: translateY(-2px); /* ホバー時に少し浮き上がる */
}

.option-num-area label:hover::before {
  transform: translateY(-50%) scale(1.1); /* ホバー時にラジオボタンが少し大きくなる */
}

.option-num-area input[type="radio"]:checked + label {
  color: #d3bb8f; /* チェックされたラベルの文字色 */
}

.visually-hidden {
  position: absolute;
  white-space: nowrap;
  border: 0;
  clip: rect(0 0 0 0);
  clip-path: inset(50%);
  overflow: hidden;
  height: 1px;
  width: 1px;
  margin: -1px;
  padding: 0;
}

/* ここまでアニメーション */

.main-option-area p {
  padding: 0px min(calc(9vw / 5), 16px);

  font-size: min(calc(9vw / 5), 16px);
  color: #fff;
  white-space: nowrap;
}

.suboption-area p {
  padding: 0px min(calc(9vw / 5), 16px);

  font-size: min(calc(9vw / 5), 16px);
  color: #fff;
  white-space: nowrap;
}

.suboptions {
  padding: 0px min(calc(9vw / 5), 16px) min(calc(9vw / 5), 16px);
}

.artifact-info-right-area {
  height: 100%;

  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  gap: min(calc(11vw / 5), 20px);
}

.reinforce-num-area p {
  padding: 0px min(calc(9vw / 5), 16px);

  font-size: min(calc(9vw / 5), 16px);
  color: #fff;
  white-space: nowrap;
}

.level-area p {
  padding: 0px min(calc(9vw / 5), 16px);

  font-size: min(calc(9vw / 5), 16px);
  color: #fff;
  white-space: nowrap;
}

.active-option-area {
  margin-top: min(calc(22vw / 5), 30px);

  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
}

.active-option-inner-area {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
}

.active-option-area p {
  padding: 0px min(calc(9vw / 5), 16px);

  font-size: min(calc(9vw / 5), 16px);
  color: #fff;
  white-space: nowrap;
}

.range-slider {
  margin-bottom: 5px;

  display: flex;
  align-items: center;
}

.range-slider input[type="range"] {
  width: min(calc(100vw / 5), 200px);
}

.active-op-value {
  width: min(calc(45vw / 5), 70px);

  text-align: center;
  font-size: min(calc(9vw / 5), 16px);
  background: none;
  border: none;
  color: #fff;
}
</style>

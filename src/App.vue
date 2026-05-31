<template>
  <body>
    <div class="app">
      <OverlayLoading
        :loading="loading"
        :paimonImg="paimonImg"
        :loading_msg="loading_msg"
        :images="images"
        :opacities="opacities"
      />

      <OverlayInfo
        :show_scan_info="show_scan_info"
        :show_data_info="show_data_info"
        :show_info_box="show_info_box"
        :show_close_button="show_close_button"
        @close-scan-info="scanInfo(false)"
        @close-data-info="dataInfo(false)"
      />
      
      <ArtifactImport
        :fileName="fileName"
        @file-change="handleFileChange"
        @scan-image="scanImage"
        @show-info="scanInfo(true)"
      />
      
      <div class="input-area">
        <ArtifactInfoArea
          :artifactImg="artifactImg"
          :option="option"
          :position="position"
          :main_op="main_op"
          :filteredMainOptions="filteredMainOptions"
          :is_crit_rate="is_crit_rate"
          :is_crit_dmg="is_crit_dmg"
          :is_atk="is_atk"
          :is_hp="is_hp"
          :is_em="is_em"
          :count="count"
          :level="level"
          :activeMode="activeMode"
          :activeOption="activeOption"
          :activeValueFormatted="activeValueFormatted"
          :scoreType="score_type"
          @update:option="option = $event"
          @update:position="position = $event"
          @update:main_op="main_op = $event"
          @update:is_crit_rate="is_crit_rate = $event"
          @update:is_crit_dmg="is_crit_dmg = $event"
          @update:is_atk="is_atk = $event"
          @update:is_hp="is_hp = $event"
          @update:is_em="is_em = $event"
          @update:count="count = $event"
          @update:level="level = $event"
          @update:activeMode="activeMode = $event"
          @update:activeOption="activeOption = $event"
          @updateActiveValue="activeValue = $event.score, activeValueFormatted = $event.formatted"
        />

        <div class="input-area-right">
          <ScoreInfoArea
            :score_type="score_type"
            :initScore_formatted="initScore_formatted"
            :searchScore_formatted="searchScore_formatted"
            @update:score_type="score_type = $event"
            @updateInitScore="initScore = $event.score, initScore_formatted = $event.formatted"
            @updateSearchScore="searchScore = $event.score, searchScore_formatted = $event.formatted"
          />

          <ElixirArea
            :scoreType="score_type"
            :elixir="elixir"
            :elixirOption="elixir_option"
            @update:elixir="elixir = $event"
            @update:elixirOption="elixir_option = $event"
          />
        </div>
      </div>

      <OutputArea
        :distributionImg="distributionImg"
        :bold_space_formatted="bold_space_formatted"
        :percentile="percentile"
        :average="average"
        :variance="variance"
        :skewness="skewness"
        :kurtosis="kurtosis"
        @update-output="updateOutput"
        @show-info="dataInfo(true)"
        @updateBoldSpace="bold_space = $event.score, bold_space_formatted = $event.formatted, bold = $event.bold"
      />
    </div>
  </body>
</template>

<script>
import paimon_def from "@/assets/paimon.webp";
import paimon_suc from "@/assets/paimon-success.webp";
import paimon_fail from "@/assets/paimon-failed.webp";
import OverlayLoading from './components/OverlayLoading.vue';
import OverlayInfo from './components/OverlayInfo.vue';
import ArtifactImport from './components/ArtifactImport.vue';
import ArtifactInfoArea from './components/ArtifactInfoArea.vue';
import ScoreInfoArea from './components/ScoreInfoArea.vue';
import OutputArea from './components/OutputArea.vue';
import ElixirArea from './components/ElixirArea.vue';
import { scanImageApi, getDistributionApi, getDataApi } from './scripts/api.js';
import { scanInfo, dataInfo } from './scripts/info.js';

export default {
  name: 'App',
  components: {
    OverlayLoading,
    OverlayInfo,
    ArtifactImport,
    ArtifactInfoArea,
    ScoreInfoArea,
    OutputArea,
    ElixirArea
  },
  data() {
    return {
      // --- オーバーレイ関連 ---
      show_scan_info:      false,               // スキャンinfoの表示フラグ
      show_data_info:      false,               // 出力データinfoの表示フラグ
      show_info_box:       false,               // infoボックスの表示フラグ
      show_close_button:   false,               // infoボックスの閉じるボタンの表示フラグ

      // --- モード関連 ---
      activeMode:          "new",              // 表示中のモード ('old' or 'new')

      // --- ロード・スキャン関連 ---
      loading:             false,               // ロード状態フラグ
      scanning:            false,               // スキャン中のフラグ
      paimonImg:           paimon_def,          // ロード中パイモン画像
      loading_msg:         "パイモンが計算中...",  // ロード中メッセージ
      images: [                                 // ロード中の画像配列
        require('@/assets/artifact-0.webp'),
        require('@/assets/artifact-1.webp'),
        require('@/assets/artifact-2.webp'),
        require('@/assets/artifact-3.webp'),
        require('@/assets/artifact-4.webp'),
      ],                                        // ロード画像配列
      opacities:           [0, 0, 0, 0, 0],     // ロード中アイコンの透明度
      intervalId:          null,                // ロードアニメーションのインターバルID

      // --- ファイル・画像関連 ---
      artifactImg:         null,         // スキャンした画像のURL
      selectedFile:        null,         // 選択したファイル
      fileName:            '',           // 選択したファイル名

      // --- スコア・オプション関連 ---
      initScore:           0,            // 初期スコア
      initScore_formatted: "0.0",        // 小数点第１位までの初期スコア
      searchScore:         40,           // 調査スコア
      searchScore_formatted:"40.0",      // 小数点第１位までの調査スコア
      option:              3,            // 初期オプション数
      count:               1,            // 強化回数
      level:               0,            // レベル

      // --- 聖遺物部位・メインオプション関連 ---
      position:            "生の花",      // 部位
      main_op:             "atk",        // メインステータス
      mainOptions: {                     // 部位ごとのメインオプション
        "生の花": [
          { value: "hp", text: "HP実数値" },
        ],
        "死の羽": [
          { value: "atk", text: "攻撃力実数値" },
        ],
        "時の砂": [
          { value: "hp%", text: "HP%" },
          { value: "atk%", text: "攻撃力%" },
          { value: "def%", text: "防御力%" },
          { value: "er", text: "元素チャージ効率" },
          { value: "em", text: "元素熟知" },
        ],
        "空の杯": [
          { value: "atk%", text: "攻撃力%" },
          { value: "hp%", text: "HP%" },
          { value: "def%", text: "防御力%" },
          { value: "em", text: "元素熟知" },
          { value: "element", text: "元素ダメージ" },
          { value: "physical", text: "物理ダメージ" },
        ],
        "理の冠": [
          { value: "atk%", text: "攻撃力%" },
          { value: "hp%", text: "HP%" },
          { value: "def%", text: "防御力%" },
          { value: "em", text: "元素熟知" },
          { value: "crit-rate", text: "会心率" },
          { value: "crit-dmg", text: "会心ダメージ" },
          { value: "heal", text: "治療効果" },
        ],
      },                                  // 部位ごとのメインオプション

      // --- アクティブオプション関連 ---
      activeOption:           "else",          // 表示中のアクティブオプション
      activeValue:            0.0,             // アクティブオプションの値
      activeValueFormatted:   "0.0",           // 小数点第１位までのアクティブオプションの値

      // --- サブオプションの有効フラグ ---
      is_crit_rate:        false,         // 会心率のフラグ
      is_crit_dmg:         false,         // 会心ダメージのフラグ
      is_atk:              false,         // 攻撃力%のフラグ
      is_hp:               false,         // HP%のフラグ
      is_em:               false,         // 熟知のフラグ

      // --- スコア計算・エリクサー関連 ---
      score_type:          "atk",                     // スコア計算方法(攻撃力換算, HP換算, 熟知換算)
      elixir:              false,                     // エリクサー使用フラグ
      elixir_option:       ["crit-dmg", "crit-rate"], // エリクサーのオプション

      // --- 棒グラフ・分布画像関連 ---
      bold:                true,          // 棒グラフ表示フラグ
      bold_space:          5.0,           // 棒グラフの間隔
      bold_space_formatted:"5.0",         // 小数点第１位までの棒グラフ間隔
      distributionImg:     null,          // 確率分布グラフ画像

      // --- 出力データ関連 ---
      percentile:          [["100.0", "0"], ["0", "0"], ["25", "0"], ["50", "0"], ["75", "0"], ["100", "0"]], // パーセンタイル
      average:             0,             // 平均
      variance:            0,             // 分散
      skewness:            0,             // 歪度
      kurtosis:            0,             // 尖度
    };
  },

  computed: {
    /*
    部位に応じてメインオプションをフィルタリングする

    生の花:
      - HP実数値
    死の羽:
      - 攻撃力実数値
    時の砂:
      - HP%
      - 攻撃力%
      - 防御力%
      - 元素チャージ効率
      - 元素熟知
    空の杯:
      - 攻撃力%
      - HP%
      - 防御力%
      - 元素熟知
      - 元素ダメージ
      - 物理ダメージ
    理の冠:
      - 攻撃力%
      - HP%
      - 防御力%
      - 元素熟知
      - 会心率
      - 会心ダメージ
      - 治療効果
    */
    filteredMainOptions() {
      return this.mainOptions[this.position];
    },
  },

  watch: {
    // メインオプションが変更されたときに、main_op に最初の要素を設定
    position(pos) {
      if (this.scanning) {
        this.scanning = false;
      } else {
        this.main_op = this.mainOptions[pos][0].value;
      }
    },
  },

  mounted() {
    // 初期化時に main_op を最初の値に設定
    this.main_op = this.mainOptions[this.position][0].value;
  },

  methods: {
    // スキャンに関するinfoを表示
    scanInfo(open) {
      scanInfo(this, open);
    },

    // 出力データに関するinfoを表示
    dataInfo(open) {
      dataInfo(this, open);
    },

    // ファイル選択時に呼ばれる
    handleFileChange(event) {
      const file = event.target.files[0];
      if (file && file.type.startsWith('image/')) {
        this.selectedFile = file;
        this.fileName = file.name;
        const artifactImg = URL.createObjectURL(file);
        this.artifactImg = artifactImg;
      } else {
        alert('画像ファイルを選択してください');
      }
    },

    // APIを使用して画像を解析
    async scanImage() {
      if (!this.selectedFile) {
        alert('画像を選択してください');
        return;
      }
      
      // ローディング状態の設定
      this.loading = true;
      this.scanning = true;
      this.paimonImg = paimon_def;
      this.loading_msg = "パイモンが計算中...";

      // アニメーションの開始
      this.startOpacityAnimation();

      // 送信するデータの準備
      const formData = new FormData();
      formData.append('image', this.selectedFile);
      formData.append('score_type', this.score_type);
      formData.append('is_new', this.activeMode == "new");

      try {
        // api.js内の関数を呼び出して画像をスキャン
        const data = await scanImageApi(formData);

        this.initScore = data.init;                             // 初期スコア
        this.initScore_formatted = this.initScore.toFixed(1);   // 小数点第１位までの初期スコア
        this.is_atk = data.is_atk;                              // 攻撃力%のフラグ
        this.is_hp = data.is_hp;                                // HP%のフラグ
        this.is_em = data.is_em;                                // 元素熟知のフラグ
        this.is_crit_rate = data.is_crit_rate;                  // 会心率のフラグ
        this.is_crit_dmg = data.is_crit_dmg;                    // 会心ダメージのフラグ
        this.option = data.option;                              // オプションの数
        this.position = data.position ;                         // 部位
        this.main_op = data.main_op;                            // メインステータス
        this.score_type = data.score_type;                      // スコア計算方法
        this.level = data.level;                                // レベル
        // 新しいモードの場合、アクティブオプションとその値を更新
        if (this.activeMode == "new" && this.option == 3) {
          this.activeOption = data.active_op;                       // アクティブオプション
          this.activeValue = data.active_op_value;                  // アクティブオプションの値
          this.activeValueFormatted = this.activeValue.toFixed(1);  // 小数点第１位までのアクティブオプションの値
        }

        // チェックボックスやスライダーの値を更新
        const atk_box = document.getElementById('is-atk');
        const hp_box = document.getElementById('is-hp');
        const em_box = document.getElementById('is-em');
        const crit_rate_box = document.getElementById('is-crit-rate');
        const crit_dmg_box = document.getElementById('is-crit-dmg');
        const init_slider = document.getElementById('init-slider');
        const active_op_slider = document.getElementById('active-op-slider');
        const option_3 = document.getElementById('option-3');
        const option_4 = document.getElementById('option-4');

        // チェックボックスの状態を設定
        atk_box.checked = this.is_atk
        hp_box.checked = this.is_hp
        em_box.checked = this.is_em
        crit_rate_box.checked = this.is_crit_rate
        crit_dmg_box.checked = this.is_crit_dmg

        // スライダーの値を設定
        init_slider.value = this.initScore;
        if (this.activeMode == "new") {
          active_op_slider.value = this.activeValue;
        }

        // ラジオボタンの値を設定
        if (this.option == 3){
          option_3.checked = true
          option_4.checked = false
        } else {
          option_3.checked = false
          option_4.checked = true
        }

        /*
        レベルによって強化回数を設定

        レベル    強化回数
        0-3      5回
        4-7      4回
        8-11     3回
        12-15    2回
        16-19    1回
        20       0回
        */

        this.count = 5 - Math.floor(this.level / 4);
        if (this.count == 0) {
          this.count = 1;
        }

        // 成功した場合の画像変更
        this.paimonImg = paimon_suc;
        this.updateLoadingMsg(false);
      } catch (error) {
        console.error('画像スキャンに失敗しました:', error);

        // 失敗した場合の画像変更
        this.paimonImg = paimon_fail;
        this.updateLoadingMsg(true);
      } finally {
        this.stopOpacityAnimation();
        // 1秒後にloadingをfalseに設定
        setTimeout(() => {
          this.loading = false;
          this.opacities = [0, 0, 0, 0, 0]
        }, 2000);
      }
    },

    // ローディングのアニメーションを開始
    startOpacityAnimation() {
      this.opacities = [0, 0, 0, 0, 0]
      let index = 0;
      this.intervalId = setInterval(() => {
        if (index < this.opacities.length) {
          this.opacities[index] = 1;
          index++;
        } else {
          this.opacities = [0, 0, 0, 0, 0]
          index = 0;
        }
      }, 500);
    },

    // ローディング終了時にアニメーションを停止
    stopOpacityAnimation() {
      if (this.intervalId) {
        clearInterval(this.intervalId);
        this.intervalId = null;
        this.opacities = [1, 1, 1, 1, 1]
      }
    },

    // 出力データの更新
    async updateOutput() {
      try {
        // リクエストボディの作成
        const requestBody = {
          option: this.option,                        // オプション数
          main_op: this.main_op,                      // メインステータス
          crit_dmg: this.is_crit_dmg,                 // 会心ダメージのフラグ
          crit_rate: this.is_crit_rate,               // 会心率のフラグ
          atk: this.is_atk,                           // 攻撃力%のフラグ
          hp: this.is_hp,                             // HP%のフラグ
          em: this.is_em,                             // 元素熟知のフラグ
          init: this.initScore,                       // 初期スコア
          score: this.searchScore,                    // 調査スコア
          count: this.count,                          // 強化回数
          start_count: Math.floor(this.level / 4),    // 開始時の強化回数
          score_type: this.score_type,                // スコア計算方法
          elixir: this.elixir,                        // エリクシル使用フラグ
          elixir_option: this.elixir_option,          // エリクシルのオプション
          is_new: (this.activeMode == "new"),         // 新しいモードかどうか
          active_op: this.activeOption,               // アクティブオプション
          active_op_value: this.activeValue,          // アクティブオプションの
          bold: this.bold,                            // 棒グラフ表示フラグ
          bold_space: this.bold_space         ,       // 棒グラフの間隔
        };
        
        // ローディング状態の設定
        this.loading = true;
        this.paimonImg = paimon_def;
        this.loading_msg = "パイモンが計算中...";

        // アニメーションの開始
        this.startOpacityAnimation();

        // 画像取得
        const distBlob = await getDistributionApi(requestBody);
        this.distributionImg = URL.createObjectURL(distBlob);

        // データ取得
        const data = await getDataApi(requestBody);

        this.percentile = data.percentile
        this.average = data.average
        this.variance = data.variance
        this.skewness = data.skewness
        this.kurtosis = data.kurtosis

        // 成功した場合の画像変更
        this.paimonImg = paimon_suc;
        this.updateLoadingMsg(false);
      } catch (error) {
        console.log(error);
        // 失敗した場合の画像変更
        this.paimonImg = paimon_fail;
        this.updateLoadingMsg(true);
      } finally {
        this.stopOpacityAnimation();
        // 1秒後にloadingをfalseに設定
        setTimeout(() => {
          this.loading = false;
          this.opacities = [0, 0, 0, 0, 0]
        }, 2000);
      }
    },

    // ロード中のメッセージを更新
    updateLoadingMsg(is_failed) {
      // 成功時と失敗時のメッセージを定義
      const msg_success = ["オイラなら余裕だぜ！", "えっへん！", "ふぅ...やっと終わったぜ...", "オイラわかったぞ！"]
      const msg_fail = ["わ、わからないぞ...", "うーん...わからないぜ！", "ん？どういうことだ？"]
      if (is_failed) {
        const random = Math.floor(Math.random() * msg_fail.length);
        this.loading_msg = msg_fail[random];
      } else {
        const random = Math.floor(Math.random() * msg_success.length);
        this.loading_msg = msg_success[random];
      }
    }
  },
};
</script>

<style>

* {
  font-family: Genshin;
}

body {
  margin: 0;
  padding: 0;
  height: fit-content;
  width: 100vw;

  background-image: url(./assets/background.png);
  background-repeat: repeat;
  background-size: calc(200vw / 5) calc(200vw / 5);
}

.app {
  padding-top: min(calc(50vh / 5), 50px);
  height: fit-content;

  display: flex;
  flex-direction: column;
  align-items: center;
}

/* ロード画面のスタイル */
.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(255, 255, 255, 0.7);
  backdrop-filter: blur(5px);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  z-index: 9999;
  visibility: hidden;
}

.overlay p {
  margin: min(calc(9vw / 5), 16px) 0px;
  font-size: min(calc(15vw / 5), 24px);
  color: #555;
}

.overlay.active {
  visibility: visible;
}

/* infoボタンのスタイル */
.info-icon {
  padding: 0px;
  margin: 0px;
  width: min(calc(20vw / 5), 30px);
  height: min(calc(20vw / 5), 30px);
  
  visibility: hidden;
}

.info-icon img {
  width: auto;
  height: 100%;

  border-radius: 50%;
  border: #d3bb8f solid 1px;
  box-shadow: 3px 5px 5px rgb(0 0 0 / 70%);

  transition-duration: 100ms;
  visibility: visible;
}

/* 値入力エリア */
.input-area {
  margin: 0px min(calc(14vw / 5), 20px) min(calc(14vw / 5), 20px);
  width: 100%;
  height: fit-content;

  display: flex;
  justify-content: center;
  align-items: flex-start;
  gap: min(calc(14vw / 5), 20px);
  flex-wrap: wrap;
}

/* 入力エリアの右側 */
.input-area-right {
  width: fit-content;
  height: fit-content;

  display: flex;
  flex-direction: column; /* スコア情報を縦に並べる */
  justify-content: flex-start;
  align-items: center;
}

/* 基本的なボタンスタイル */
.def-button {
  padding: min(calc(6vw / 5), 10px) min(calc(14vw / 5), 20px);
  width: fit-content;

  background-color: #4B5368;
  color: white;
  border-radius: 5px;
  border: none;
  outline: none;
  cursor: pointer;
  font-size: 16px;
  text-align: center;
  user-select: none;
  box-shadow: 0 3px 5px rgb(0 0 0 / 50%);
  white-space: nowrap;

  transition-duration: 100ms;
}

.def-button:hover, .info-icon img:hover {
  background-color: #424858;
  transform: scale(0.99, 0.99) translateY(2px);
}

/* 基本的なチェックボックス */
.def-checkbox input[type="checkbox"] {
  display: none;
}

.def-checkbox input[type="checkbox"] + label {
  padding-left: min(calc(26vw / 5), 35px);
  margin: 0px min(calc(22vw / 5), 30px) min(calc(22vw / 5), 30px);

  font-size: min(calc(9vw / 5), 16px);
  display: block;
  position: relative;
  color: #fff;
  cursor: pointer;
  white-space: nowrap;
}

.def-checkbox input[type="checkbox"] + label:before {
  content: '';
  display: block;
  width: min(calc(14vw / 5), 20px);
  height: min(calc(14vw / 5), 20px);
  border: 2px solid #d3bb8f;
  position: absolute;
  left: 0;
  top: 0;
  opacity: .6;
  -webkit-transition: all .12s, border-color .08s;
  transition: all .12s, border-color .08s;
}

.def-checkbox input[type="checkbox"]:checked + label:before {
  width: min(calc(9vw / 5), 10px);
  top: -5px;
  left: 5px;
  border-radius: 0;
  opacity: 1;
  border-top-color: transparent;
  border-left-color: transparent;
  -webkit-transform: rotate(45deg);
  transform: rotate(45deg);
}

/* 基本的なスイッチボタン */
.def-switch {
  position: relative;
  display: inline-block;
  width: min(calc(30vw / 5), 48px);
  height: min(calc(16vw / 5), 24px);
  vertical-align: middle;
}

.def-switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.def-switch-slider {
  position: absolute;
  cursor: pointer;
  top: 0; left: 0; right: 0; bottom: 0;
  background: #4B5368;
  border: 1px solid #d3bb8f;
  border-radius: 24px;
  transition: background 0.3s;
}

.def-switch input:checked + .def-switch-slider {
  background: #d3bb8f;
}

.def-switch-slider:before {
  content: "";
  position: absolute;
  left: min(calc(2vw / 5), 3px);
  top: min(calc(2vw / 5), 3px);
  width: min(calc(9vw / 5), 16px);
  height: min(calc(9vw / 5), 16px);
  border-radius: 50%;
  background: #d3bb8f;
  transition: transform 0.3s;
  box-shadow: 0 2px 4px rgba(0,0,0,0.2);
}

.def-switch input:checked + .def-switch-slider:before {
  transform: translateX(min(calc(16vw / 5), 24px));
  background: #4B5368;
}

/* 基本的なボックス */
.info-section {
  width: fit-content;
  height: fit-content;

  display: flex;
  flex-direction: column; /* ラジオボタンを縦に並べる */
  justify-items: center;
  align-items: center;

  background-color: #495366;
  box-shadow: 3px 5px 5px rgb(0 0 0 / 70%);
}

/* 基本的なタブ */
.def-tabs {
  width: fit-content;
  max-width: 90%;

  display: flex;
  justify-content: center;
  gap: min(calc(11vw / 5), 16px);

  transform: translateY(1px);

  z-index: 2;
}

.def-tab {
  padding: min(calc(6vw / 5), 10px) min(calc(12vw / 5), 18px);

  background: #424857;
  color: #d3bb8f;
  border-radius: 8px 8px 0 0;
  border: 1px solid transparent;
  font-size: min(calc(12vw / 5), 16px);

  cursor: pointer;

  position: relative;

  transform: translateY(-2px);

  transition: background 0.12s, color 0.12s, transform 0.08s;
}

.def-tab:hover {
  background: #4b5968;
}

.def-tab.active {
  border: 1px solid #d3bb8f;
  border-bottom-color: transparent;

  transform: translateY(0px);
}

/* 基本的なセレクトボックス */
.def-selecter {
  padding: min(calc(5vw / 5), 8px);
  margin: 0px min(calc(9vw / 5), 16px) min(calc(9vw / 5), 16px);

  font-size: min(calc(9vw / 5), 16px);
  background-color: #4B5368;
  color: #d3bb8f;
  border-radius: 5px;
  border: none;
  outline: none;
  cursor: pointer;
  width: 80%;
  appearance: none;
  transition: background-color 0.3s ease;
}

.def-selecter:hover {
  background-color: #424858;
}

.def-selecter:focus {
  background-color: #3a404f;
  border: 1px solid #d3bb8f;
}

.def-selecter::-ms-expand {
  display: none;
}

.def-selecter option {
  background-color: #4B5368;
  color: white;
}

/* 基本的なスライダー */
.range-slider input[type="range"] {
  -webkit-appearance: none;
  height: min(calc(4vw / 5), 8px);
  background: #383d4b;
  border-radius: 5px;
  outline: none;
  cursor: pointer;
}

.range-slider input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: min(calc(16vw / 5), 22px);
  height: min(calc(16vw / 5), 22px);
  background: #d3bb8f;
  border: min(calc(2vw / 5), 3px) solid #495366;
  border-radius: 50%;
  cursor: pointer;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
  transition: transform 0.3s ease;
}

.range-slider input[type="range"]::-webkit-slider-thumb:hover {
  transform: scale(1.2); /* ホバー時に拡大 */
}

.range-slider input[type="range"]::-moz-range-thumb {
  width: 16px;
  height: 16px;
  background: #d3bb8f;
  border: 1px solid #495366;
  border-radius: 50%;
  cursor: pointer;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
  transition: transform 0.3s ease;
}

.range-slider input[type="range"]::-moz-range-thumb:hover {
  transform: scale(1.2);
}

</style>

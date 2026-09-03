<template>
  <div id="login-box" :style=" background ? 'background: var(--el-bg-color)' : ''" v-loading="oauthLoading" element-loading-text="登录中...">
    <div id="background-wrap" v-if="!settingStore.settings.background">
      <div v-for="(c, i) in clouds" :key="i" class="cloud-track" :style="c.track">
        <div class="cloud-bob" :style="c.bob">
          <div class="cloud" :style="c.shape" aria-hidden="true">
            <svg class="cloud-svg" viewBox="0 0 350 140" aria-hidden="true" focusable="false">
              <path class="cloud-silhouette" :d="c.path"></path>
              <path class="cloud-belly" d="M8 91c23 12 52 14 84 10 38-5 76 0 115-1 38-1 74 2 108-4 9 8 3 20-13 27-38 8-72 2-99 7-46 4-91-1-126 2-22 1-39-3-46-8-16-2-24-14-23-33Z"></path>
              <path class="cloud-highlight" d="M31 65c3-21 18-36 39-36 17 0 31 9 39 24 11-23 31-40 57-40 17 0 32 6 43 17-23-7-49-3-68 14-15 13-23 30-41 33-20 4-42-5-69-12Z"></path>
            </svg>
          </div>
        </div>
      </div>
      <div class="suffix-drift far" aria-hidden="true">
        <span
            v-for="it in suffixFar"
            :key="it.key"
            :class="['chip-track', it.cls, { 'drift-off': it.off }]"
            :style="it.track"
        ><span class="suffix-chip" :style="it.chip">{{ it.text }}</span></span>
      </div>
      <div class="suffix-drift near" aria-hidden="true">
        <span
            v-for="it in suffixNear"
            :key="it.key"
            :class="['chip-track', it.cls, { 'drift-off': it.off }]"
            :style="it.track"
        ><span class="suffix-chip" :style="it.chip">{{ it.text }}</span></span>
      </div>
    </div>
    <div v-else :style="background"></div>
    <div class="domain-count" v-if="domainTotal">{{ $t('domainCount', { total: domainTotal }) }}</div>
    <div class="form-wrapper">
      <div class="container">
        <span class="form-title">{{ settingStore.settings.title }}</span>
        <span class="form-desc" v-if="show === 'login'">{{ $t('loginTitle') }}</span>
        <span class="form-desc" v-else>{{ $t('regTitle') }}</span>
        <div v-show="show === 'login'">
          <el-input :class="!hideLoginDomain ? 'email-input' : ''" v-model="form.email"
                    type="text" :placeholder="$t('emailAccount')" autocomplete="off" @keyup.enter="submit">
            <template #append v-if="!hideLoginDomain">
              <div @click.stop="openSelect">
                <el-select
                    v-if="show === 'login'"
                    ref="mySelect"
                    v-model="suffix"
                    :placeholder="$t('select')"
                    class="select"
                >
                  <el-option
                      v-for="item in domainList"
                      :key="item"
                      :label="item"
                      :value="item"
                  />
                </el-select>
                <div style="color: var(--el-text-color-primary)">
                  <span>{{ suffix }}</span>
                  <Icon class="setting-icon" icon="mingcute:down-small-fill" width="20" height="20"/>
                </div>
              </div>
            </template>
          </el-input>
          <el-input v-model="form.password" :placeholder="$t('password')" type="password" autocomplete="off" @keyup.enter="submit">
          </el-input>
          <el-button class="btn" type="primary" @click="submit" :loading="loginLoading"
          >{{ $t('loginBtn') }}
          </el-button>
          <el-button v-for="p in oauthProviders" :key="p.key" class="btn" style="margin-top: 10px" @click="oauthLogin(p.key)">
            <el-avatar v-if="p.iconType === 'image'" :src="p.icon" :size="18" style="margin-right: 10px" />
            <Icon v-else :icon="p.icon" width="18" height="18" style="margin-right: 10px" />
            {{ p.label }}
          </el-button>
        </div>
        <div v-show="show !== 'login'">
          <el-input :class="!hideLoginDomain ? 'email-input' : ''" v-model="registerForm.email" type="text" :placeholder="$t('emailAccount')"
                    autocomplete="off" @keyup.enter="submitRegister">
            <template #append v-if="!hideLoginDomain">
              <div @click.stop="openSelect">
                <el-select
                    v-if="show !== 'login'"
                    ref="mySelect"
                    v-model="suffix"
                    :placeholder="$t('select')"
                    class="select"
                >
                  <el-option
                      v-for="item in domainList"
                      :key="item"
                      :label="item"
                      :value="item"
                  />
                </el-select>
                <div>
                  <span>{{ suffix }}</span>
                  <Icon class="setting-icon" icon="mingcute:down-small-fill" width="20" height="20"/>
                </div>
              </div>
            </template>
          </el-input>
          <el-input v-model="registerForm.password" :placeholder="$t('password')" type="password" autocomplete="off" @keyup.enter="submitRegister"/>
          <el-input v-model="registerForm.confirmPassword" :placeholder="$t('confirmPwd')" type="password"
                    autocomplete="off" @keyup.enter="submitRegister"/>
          <el-input v-if="settingStore.settings.regKey === 0" v-model="registerForm.code" :placeholder="$t('regKey')"
                    type="text" autocomplete="off" @keyup.enter="submitRegister"/>
          <el-input v-if="settingStore.settings.regKey === 2" v-model="registerForm.code"
                    :placeholder="$t('regKeyOptional')" type="text" autocomplete="off" @keyup.enter="submitRegister"/>
          <div v-show="verifyShow"
               class="register-turnstile"
               :data-sitekey="settingStore.settings.siteKey"
               data-callback="onTurnstileSuccess"
               data-error-callback="onTurnstileError"
               data-after-interactive-callback="loadAfter"
               data-before-interactive-callback="loadBefore"
          >
            <span style="font-size: 12px;color: #F56C6C" v-if="botJsError">{{ $t('verifyModuleFailed') }}</span>
          </div>
          <el-button class="btn" style="margin: 0" type="primary" @click="submitRegister" :loading="registerLoading"
          >{{ $t('regBtn') }}
          </el-button>
          <el-button v-for="p in oauthProviders" :key="p.key" class="btn" style="margin-top: 10px" @click="oauthLogin(p.key)">
            <el-avatar v-if="p.iconType === 'image'" :src="p.icon" :size="18" style="margin-right: 10px" />
            <Icon v-else :icon="p.icon" width="18" height="18" style="margin-right: 10px" />
            {{ p.label }}
          </el-button>
        </div>
        <template v-if="settingStore.settings.register === 0">
          <div class="switch" @click="show = 'register'" v-if="show === 'login'">{{ $t('noAccount') }}
            <span>{{ $t('regSwitch') }}</span></div>
          <div class="switch" @click="show = 'login'" v-else>{{ $t('hasAccount') }} <span>{{ $t('loginSwitch') }}</span>
          </div>
        </template>
      </div>
    </div>
    <el-dialog class="bind-dialog" v-model="showBindForm"  title="注册邮箱" >
      <div class="bind-container">
        <el-input :class="!hideLoginDomain ? 'email-input' : ''" v-model="bindForm.email" type="text" :placeholder="$t('emailAccount')" autocomplete="off" @keyup.enter="bind">
          <template #append v-if="!hideLoginDomain">
            <div @click.stop="openSelect">
              <el-select
                  ref="mySelect"
                  v-model="suffix"
                  :placeholder="$t('select')"
                  class="select"
              >
                <el-option
                    v-for="item in domainList"
                    :key="item"
                    :label="item"
                    :value="item"
                />
              </el-select>
              <div>
                <span>{{ suffix }}</span>
                <Icon class="setting-icon" icon="mingcute:down-small-fill" width="20" height="20"/>
              </div>
            </div>
          </template>
        </el-input>
        <el-input v-if="settingStore.settings.regKey === 0" v-model="bindForm.code" :placeholder="$t('regKey')"
                  type="text" autocomplete="off" @keyup.enter="bind"/>
        <el-input v-if="settingStore.settings.regKey === 2" v-model="bindForm.code"
                  :placeholder="$t('regKeyOptional')" type="text" autocomplete="off" @keyup.enter="bind"/>
        <el-button class="btn" type="primary" @click="bind" :loading="bindLoading"
        >绑定
        </el-button>
      </div>
    </el-dialog>
    <a v-show="settingStore.settings.projectLink" class="github" href="https://github.com/maillab/cloud-mail">
      <Icon icon="mingcute:github-line" color="#1890ff" width="20" height="20" />
    </a>
  </div>
</template>

<script setup>
import router from "@/router";
import {useRoute} from "vue-router";
import {computed, nextTick, reactive, ref, shallowRef, watch} from "vue";
import {login} from "@/request/login.js";
import {register} from "@/request/login.js";
import {websiteConfig} from "@/request/setting.js";
import {isEmail} from "@/utils/verify-utils.js";
import {useSettingStore} from "@/store/setting.js";
import {useAccountStore} from "@/store/account.js";
import {useUserStore} from "@/store/user.js";
import {useUiStore} from "@/store/ui.js";
import {Icon} from "@iconify/vue";
import {cvtR2Url} from "@/utils/convert.js";
import {loginUserInfo} from "@/request/my.js";
import {permsToRouter} from "@/perm/perm.js";
import {useI18n} from "vue-i18n";
import {oauthBindUser, oauthLinuxDoLogin, oauthGithubLogin, oauthGoogleLogin} from "@/request/ouath.js";

const {t} = useI18n();
const accountStore = useAccountStore();
const userStore = useUserStore();
const uiStore = useUiStore();
const settingStore = useSettingStore();
const route = useRoute();
const loginLoading = ref(false)
const bindLoading = ref(false)
const oauthLoading = ref(false);
const showBindForm = ref(false);
const show = ref('login')

const oauthKeys = ['linuxdo', 'github', 'google']

const oauthProvider = computed(() => {
  const fromState = route.query.state
  if (oauthKeys.includes(fromState)) return fromState
  const fromStore = sessionStorage.getItem('oauthProvider')
  return oauthKeys.includes(fromStore) ? fromStore : null
})

const oauthProviders = computed(() => {
  const allProviders = [
    { key: 'google', label: 'Google', icon: 'devicon:google', iconType: 'iconify' },
    { key: 'github', label: 'GitHub', icon: 'codicon:github-inverted', iconType: 'iconify' },
    { key: 'linuxdo', label: 'LinuxDo', icon: '/image/linuxdo.webp', iconType: 'image' },
  ]
  return allProviders.filter(p => settingStore.settings[p.key + 'Switch'] === 0)
})

const bindForm = reactive({
  email: '',
  oauthUserId: '',
  code: ''
})

const form = reactive({
  email: '',
  password: '',

});
const mySelect = ref()
const suffix = ref('')
const registerForm = reactive({
  email: '',
  password: '',
  confirmPassword: '',
  code: null
})
const domainList = settingStore.domainList;

// 邮箱后缀像云一样飘过：环境变量里配置多少个域名，这里就渲染多少个
// 每次进入页面换一副排布，避免刷新后看到完全相同的队形。
// 只影响"谁在哪条轨道、整体从什么时刻开始"，不影响轨道内的等间隔，
// 因此不重叠的保证依然成立。
const driftSeed = Math.floor(Math.random() * 233280)

// 背景云。原本是 5 朵写死的 div，靠文档流上下排开，想加密就会往视口外堆，
// 所以改成绝对定位后按参数生成：数量、大小、速度、高度都在这里调。
const NARROW = typeof window !== 'undefined'
  && window.matchMedia('(max-width: 767px)').matches

// ── 统一的深度采样器：云与后缀共用 ────────────────────────────────
// d ∈ [0,1]，0 = 最远（屏幕下方、地平线雾带），1 = 最近（屏幕上方、天顶）。
// 大小 / 速度 / 雾色 / 模糊 / 起伏 / z 序全部由 d 派生。改造前这些量各自
// 独立随机，等于把本该同源的属性各摇一次骰子，画面在物理上自相矛盾。
const lerp = (a, b, t) => a + (b - a) * t
const mixc = (a, b, t) => a.map((v, i) => Math.round(lerp(v, b[i], t)))
const rgbs = (c) => `rgb(${c[0]},${c[1]},${c[2]})`   // 逗号语法，兼容老 WebView

// 采样背景天空渐变（#login-box 的 linear-gradient），p = 纵向 0~1
const SKY = [[0, [41, 128, 185]], [0.5, [109, 213, 250]], [1, [255, 255, 255]]]
const skyAt = (p) => {
  p = Math.min(1, Math.max(0, p))
  const i = p < 0.5 ? 0 : 1
  const [p0, c0] = SKY[i], [p1, c1] = SKY[i + 1]
  return c0.map((v, k) => lerp(v, c1[k], (p - p0) / (p1 - p0)))
}
const haze = (d) => 0.68 * Math.pow(1 - d, 1.6)
const hazeColor = (p, d) => mixc(skyAt(p), [244, 249, 251], 0.30 + 0.22 * (1 - d))
// 雾交叉线 p=0.70：线以上天空比云暗（远云应更亮），线以下天空反而比云亮
// （底部渐变到白），远云必须画得更暗更灰，否则会被白色地平线整个吞掉。
const cloudBody = (p, d) =>
  mixc(p < 0.70 ? [255, 255, 255] : [204, 220, 232], hazeColor(p, d), haze(d))
const cloudBelly = (p, d) => {
  const h = haze(d)
  return mixc(cloudBody(p, d),
    mixc([183, 206, 226], hazeColor(Math.min(1, p + 0.06), d), h), 0.62 * (1 - h))
}
const flat = (d) => 1 - 0.22 * Math.pow(1 - d, 1.1)   // 掠射角压扁：远处的云更扁，但仍保留云顶轮廓

const CLOUD_PATHS = [
  'M34 118C17 118 6 108 6 93C6 79 17 68 31 66C32 44 49 28 71 28C88 28 103 37 111 51C121 27 141 13 166 13C194 13 217 34 220 62C231 49 246 43 261 46C280 49 294 64 297 81C314 81 327 92 327 105C327 118 316 127 301 127C276 131 247 128 204 132C158 135 113 130 77 133C58 134 41 130 34 126C20 126 8 124 6 118Z',
  'M29 118C14 118 4 107 4 92C4 77 16 66 33 64C35 48 47 36 62 31C77 26 91 30 101 41C110 21 128 10 149 10C176 10 197 29 201 55C211 44 226 39 241 43C259 47 271 61 273 77C290 76 304 84 310 96C316 108 309 120 296 125C267 132 230 128 203 131C164 135 116 130 78 133C56 134 38 130 29 126C15 126 6 124 4 118Z'
]

// ── 云：按深度分五层，层内用同一个 u 贯穿全部派生量 ────────────────
const CLOUD_LAYERS = [
  { d: 0.94, n: [1, 1], sx: [1.14, 1.30], top: [-7, 4], dur: [22, 26], soft: 0, sh: '0 12px 14px rgba(23,66,102,.22)', z: 94 },
  { d: 0.78, n: [2, 1], sx: [0.80, 1.02], top: [4, 22], dur: [28, 34], soft: 0, sh: '0 8px 10px rgba(23,66,102,.15)', z: 78 },
  { d: 0.52, n: [3, 2], sx: [0.50, 0.70], top: [22, 44], dur: [36, 44], soft: 0, sh: '', z: 46 },
  { d: 0.26, n: [4, 2], sx: [0.30, 0.44], top: [42, 62], dur: [48, 58], soft: 0.6, sh: '', z: 24 },
  { d: 0.08, n: [3, 1], sx: [0.17, 0.26], top: [60, 80], dur: [62, 76], soft: 1.1, sh: '', z: 10 }
]

const clouds = CLOUD_LAYERS.flatMap((L) =>
  Array.from({ length: L.n[NARROW ? 1 : 0] }, () => {
    const u = Math.random()                       // 同一个 u，绝不各摇一次
    const d = Math.min(1, Math.max(0, L.d + (u - 0.5) * 0.06))
    const sx = lerp(L.sx[0], L.sx[1], u)          // u↑ = 更近 = 更大
    const top = lerp(L.top[1], L.top[0], u)       // u↑ = 更近 = 更靠上
    const dur = lerp(L.dur[1], L.dur[0], u)       // u↑ = 更近 = 更快（视差方向）
    const sy = sx * flat(d)
    const p = Math.min(1, Math.max(0, top / 100 + 0.06))
    // 起伏周期取 0.809 倍横移周期：每横穿一屏约起伏 1.24 次。
    // 禁止取 1.0 或 0.5，那会与横移共振成固定的斜向直线运动。
    const bobDur = dur * (0.809 + (Math.random() - 0.5) * 0.16)
    const rest = Math.random()
    return {
      path: CLOUD_PATHS[Math.floor(Math.random() * CLOUD_PATHS.length)],
      track: {
        top: top.toFixed(2) + '%', zIndex: L.z,
        '--s': sx.toFixed(3), '--sy': sy.toFixed(3),
        animationDuration: dur.toFixed(1) + 's',
        animationDelay: (-rest * dur).toFixed(1) + 's',
        '--rest': `calc((100% + ${Math.round(350 * sx)}px) * ${rest.toFixed(4)})`
      },
      bob: {
        opacity: (0.82 + 0.18 * d).toFixed(2),
        // drop-shadow 必须挂在 bob 上：track 的子元素在动，父容器带 filter
        // 会强制整棵子树逐帧重栅格；bob 的子元素静止，滤镜只栅格化一次。
        filter: L.sh && !NARROW ? `drop-shadow(${L.sh})` : 'none',
        '--bob': (3 + 9 * d).toFixed(1) + 'px',
        animationDuration: bobDur.toFixed(1) + 's',
        animationDelay: (-Math.random() * bobDur).toFixed(1) + 's'
      },
      shape: {
        '--body': rgbs(cloudBody(p, d)),
        '--belly': rgbs(cloudBelly(p, d)),
        '--soft': (NARROW ? 0 : L.soft).toFixed(2) + 'px',
        '--highlight': (0.16 + 0.16 * d).toFixed(2)
      }
    }
  })
)

const domainTotal = computed(() => (settingStore.domainList || []).length)

// 过屏时间占比的阶梯。占比决定同屏数量（同屏 ≈ 总数 × 占比）；
// 具体档位不再按总数查表，而是由几何反解得出，见下方 tier。
const CHIP_LADDER = [
  { cls: 'rate-a', ratio: 0.75 }, { cls: 'rate-b', ratio: 0.60 },
  { cls: 'rate-c', ratio: 0.45 }, { cls: 'rate-d', ratio: 0.34 },
  { cls: 'rate-e', ratio: 0.26 }, { cls: 'rate-f', ratio: 0.18 },
  { cls: 'rate-g', ratio: 0.13 }, { cls: 'rate-h', ratio: 0.09 }
]

function buildDrifts(list) {
  const total = list.length
  if (!total) return []
  let rndState = driftSeed
  const rand = () => {
    rndState = (rndState * 9301 + 49297) % 233280
    return rndState / 233280
  }
  // 移动端登录卡占满宽度，只有顶部一条窄带可用，必须减量
  const render = NARROW ? Math.min(total, 16) : total
  const lanes = NARROW
    ? Math.min(4, Math.max(2, Math.ceil(render / 5)))
    : Math.min(11, Math.max(3, Math.ceil(total / 5)))

  // 先洗牌域名本身：配置里相邻的域名若依次落到相邻轨道，飘起来像一份斜排的列表
  const pool = [...list]
  for (let k = pool.length - 1; k > 0; k--) {
    const j = Math.floor(rand() * (k + 1))
    const t = pool[k]; pool[k] = pool[j]; pool[j] = t
  }
  const items = pool.slice(0, render)

  const members = Array.from({ length: lanes }, () => [])
  for (let i = 0; i < render; i++) members[i % lanes].push(i)

  // ── 深度挂在轨道上，而不是挂在单个胶囊上 ──
  // 硬约束要求"同轨道同速"，即速度是轨道的属性；而视差要求速度与大小同向，
  // 所以大小也必须是轨道的属性。一条轨道 = 一个景深平面。这样不重叠的保证
  // 不但没被削弱，反而更强：同轨道内字号完全一致，胶囊宽度只剩文本长度一个变量。
  const U = l => lanes > 1 ? 1 - l / (lanes - 1) : 0.6   // 轨道 0 在最上 = 最近
  const laneSpeed = Array.from({ length: lanes }, (_, l) =>
    1.34 - 0.52 * U(l) + (rand() - 0.5) * 0.05)          // lane 的纯函数 → 同轨道同速
  const EM_LO = NARROW ? 0.94 : 0.84, EM_HI = NARROW ? 1.16 : 1.30
  const laneEm = l => EM_LO + (EM_HI - EM_LO) * U(l)
  const chipFill = (u) =>
    `rgba(${Math.round(lerp(26, 9, u))},${Math.round(lerp(72, 42, u))},${Math.round(lerp(112, 72, u))},${(0.80 + 0.14 * u).toFixed(3)})`

  // ── 档位几何反解：让"不重叠"由参数结构保证，而不是靠一次性扫描验证 ──
  const vw = typeof window !== 'undefined' ? window.innerWidth : 1440
  const vh = typeof window !== 'undefined' ? window.innerHeight : 900
  const cardW = NARROW ? 0 : (vw <= 1024 ? 402 : 450)
  const skyW = Math.max(240, vw - cardW)
  const basePx = NARROW ? 12.5 : Math.min(15.5, Math.max(14, vw * 0.0102))
  const maxChars = items.reduce((m, t) => Math.max(m, t.length), 1)   // 元素已含 @
  // 0.55em/字 是这套圆体字栈的实测均宽；1.84em = padding .92×2 + 描边
  const maxChipPx = (maxChars * 0.55 + 1.84) * basePx * EM_HI * 1.08
  const travel = skyW + 280
  const perLane = Math.ceil(render / lanes)
  const safe = travel / (perLane * (maxChipPx + 48))     // 48px 最小净空
  const dense = 15 / render                              // 同屏目标 15 个
  const cap = Math.min(safe, dense)
  const tier = CHIP_LADDER.find(r => r.ratio <= cap) || CHIP_LADDER[CHIP_LADDER.length - 1]
  const CROSS = 20                                       // 横穿屏幕锁定约 20 秒
  const base = CROSS / tier.ratio

  // 每条轨道一个独立随机相位。不能用"轨道序号 × 常数"：那是固定增量，
  // 进入页面时后缀会连成一条等差斜线。纯随机偶尔也会凑出斜的一串，
  // 所以再筛一道：算出进场那一刻每条轨道最靠左后缀的横向位置，与轨道
  // 序号求相关性，太像一条直线就重摇。
  const leadSlope = (offs) => {
    const pts = []
    for (let l = 0; l < lanes; l++) {
      const n = members[l].length
      let first = Infinity
      for (let k = 0; k < n; k++) {
        const ph = (k / n + offs[l]) % 1
        if (ph <= tier.ratio && ph < first) first = ph
      }
      if (first < Infinity) pts.push([l, first])
    }
    if (pts.length < 3) return 0
    const n = pts.length
    const mx = pts.reduce((a, q) => a + q[0], 0) / n
    const my = pts.reduce((a, q) => a + q[1], 0) / n
    let num = 0, dx = 0, dy = 0
    for (const [a, b] of pts) { num += (a - mx) * (b - my); dx += (a - mx) ** 2; dy += (b - my) ** 2 }
    return dx && dy ? Math.abs(num / Math.sqrt(dx * dy)) : 0
  }
  let laneOffset = Array.from({ length: lanes }, () => rand())
  for (let tries = 0; tries < 40 && leadSlope(laneOffset) > 0.32; tries++) {
    laneOffset = Array.from({ length: lanes }, () => rand())
  }
  const phase = new Array(render)
  members.forEach((group, lane) => {
    group.forEach((idx, k) => {
      phase[idx] = ((k / group.length) + laneOffset[lane]) % 1
    })
  })

  // 轨道高度：恒定间距会读成表格行，加 ±25% 抖动打破。
  // 这只影响纵向，与"同轨道横向等间隔"的不重叠保证互不相干。
  const TOP_FROM = NARROW ? 2 : 4, TOP_TO = NARROW ? 14 : 68
  const gap = lanes > 1 ? (TOP_TO - TOP_FROM) / (lanes - 1) : 0
  const laneJit = Array.from({ length: lanes }, () => rand())
  const tops = Array.from({ length: lanes }, (_, l) =>
    TOP_FROM + l * gap + (laneJit[l] - 0.5) * gap * 0.50)

  // 垂直起伏幅度由实际轨道间距反解；轨道密集时会自动算成 0（自动关闭）
  let minGapPx = Infinity
  for (let l = 1; l < lanes; l++)
    minGapPx = Math.min(minGapPx, (tops[l] - tops[l - 1]) / 100 * vh)
  const chipH = basePx * EM_HI * 1.45 + 2
  const bobMax = lanes > 1
    ? Math.min(6, Math.max(0, (minGapPx - chipH) / 2 - 4)) : 4

  return items.map((text, i) => {
    const lane = i % lanes
    const u = U(lane)
    const duration = Math.round(base * laneSpeed[lane])
    const dly = (-phase[i] * duration).toFixed(2) + 's'
    // reduced-motion 下的静止位置：正好取动画的 t=0 帧
    const pr = phase[i] / tier.ratio
    return {
      key: `${text}#${i}`,
      text,   // domainList 的元素后端已带 @ 前缀（setting-service 里加的）
      cls: tier.cls,
      near: lane < Math.ceil(lanes / 2),
      off: pr > 1,
      track: {
        top: tops[lane].toFixed(2) + '%',
        animationDuration: duration + 's',
        animationDelay: dly,
        '--op': (0.94 + 0.06 * u).toFixed(2),
        '--rest': pr <= 1
          ? `calc((var(--sky-w) + 280px) * ${pr.toFixed(4)} - 280px)` : '0px'
      },
      chip: {
        fontSize: laneEm(lane).toFixed(3) + 'em',
        '--fill': chipFill(u),
        '--bd': `rgba(255,255,255,${(0.16 + 0.14 * u).toFixed(2)})`,
        '--shA': (0.06 + 0.14 * u).toFixed(3),
        '--bob': (bobMax * (0.5 + 0.5 * u)).toFixed(1) + 'px',
        '--rot': (0.4 * u).toFixed(2) + 'deg'
      }
    }
  })
}

// 用 shallowRef + watch 而不是 computed：domainList 是接口回来后才填充的，
// computed 重算会改写全部 animationDuration/Delay，浏览器把这些动画全部重启，
// 用户会在页面刚可交互那一刻看到整片后缀"啪"地跳回起始队形。
const suffixDrifts = shallowRef([])
watch(() => settingStore.domainList, (l) => {
  if (l && l.length && !suffixDrifts.value.length) suffixDrifts.value = buildDrifts(l)
}, { immediate: true })
const suffixNear = computed(() => suffixDrifts.value.filter(x => x.near))
const suffixFar = computed(() => suffixDrifts.value.filter(x => !x.near))
const registerLoading = ref(false)
suffix.value = domainList[0]
const verifyShow = ref(false)
let verifyToken = ''
let turnstileId = null
let botJsError = ref(false)
let verifyErrorCount = 0

window.onTurnstileSuccess = (token) => {
  verifyToken = token;
};

window.onTurnstileError = (e) => {
  if (verifyErrorCount >= 4) {
    return
  }
  verifyErrorCount++
  console.warn('人机验加载失败', e)
  setTimeout(() => {
    nextTick(() => {
      if (!turnstileId) {
        turnstileId = window.turnstile.render('.register-turnstile')
      } else {
        window.turnstile.reset(turnstileId);
      }
    })
  }, 1500)
};

window.loadAfter = (e) => {
  console.log('loadAfter')
}

window.loadBefore = (e) => {
  console.log('loadBefore')
}

const loginOpacity = computed(() => {
  const opacity = settingStore.settings.loginOpacity
  return uiStore.dark ? `rgba(0, 0, 0, ${opacity})` : `rgba(255, 255, 255, ${opacity})`
})

const hideLoginDomain = computed(() => settingStore.settings.loginDomain === 1)

const background = computed(() => {

  return settingStore.settings.background ? {
    'background-image': `url(${cvtR2Url(settingStore.settings.background)})`,
    'background-repeat': 'no-repeat',
    'background-size': 'cover',
    'background-position': 'center'
  } : ''
})

const openSelect = () => {
  mySelect.value.toggleMenu()
}

const getFullEmail = (email) => {
  return hideLoginDomain.value ? email : email + suffix.value
}

const getEmailName = (email) => {
  return email.split('@')[0]
}

function oauthLogin(provider) {
  const clientId = settingStore.settings[provider + 'ClientId']
  const redirectUri = encodeURIComponent(window.location.origin + '/login')
  sessionStorage.setItem('oauthProvider', provider)
  const authorizeUrls = {
    linuxdo: `https://connect.linux.do/oauth2/authorize?client_id=${clientId}&redirect_uri=${redirectUri}&response_type=code&scope=openid+profile+email&state=${provider}`,
    github: `https://github.com/login/oauth/authorize?client_id=${clientId}&redirect_uri=${redirectUri}&scope=user:email&state=${provider}`,
    google: `https://accounts.google.com/o/oauth2/v2/auth?client_id=${clientId}&redirect_uri=${redirectUri}&response_type=code&scope=openid+profile+email&state=${provider}`,
  }
  window.location.href = authorizeUrls[provider]
}

const loginFns = {
  linuxdo: oauthLinuxDoLogin,
  github: oauthGithubLogin,
  google: oauthGoogleLogin,
}

oauthGetUser();

async function oauthGetUser() {

  const params = new URLSearchParams(window.location.search)
  const code = params.get('code')
  if (!code || !oauthProvider.value) return

  const provider = oauthProvider.value
  oauthLoading.value = true
  sessionStorage.removeItem('oauthProvider')
  window.history.replaceState({}, '', window.location.origin + window.location.pathname)

  loginFns[provider](code, window.location.origin + '/login').then(data => {

    bindForm.oauthUserId = data.userInfo.oauthUserId;

    if (!data.token) {
      showBindForm.value = true
      oauthLoading.value = false
      ElMessage({
        message: '请注册绑定一个邮箱',
        type: 'warning',
        duration: 4000,
        plain: true,
      })
      return;
    }

    saveToken(data.token);
  }).catch(() => {
    oauthLoading.value = false
  })
}

function bind() {

  if (bindLoading.value) return

  if (!bindForm.email) {
    ElMessage({
      message: t('emptyEmailMsg'),
      type: 'error',
      plain: true,
    })
    return
  }


  if (getEmailName(bindForm.email).length < settingStore.settings.minEmailPrefix) {
    ElMessage({
      message: t('minEmailPrefix', {msg: settingStore.settings.minEmailPrefix}),
      type: 'error',
      plain: true,
    })
    return
  }

  let email = getFullEmail(bindForm.email);


  if (!isEmail(email)) {
    ElMessage({
      message: t('notEmailMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  if (settingStore.settings.regKey === 0) {

    if (!bindForm.code) {

      ElMessage({
        message: t('emptyRegKeyMsg'),
        type: 'error',
        plain: true,
      })
      return
    }

  }

  const form = {email, oauthUserId: bindForm.oauthUserId, code: bindForm.code}

  bindLoading.value = true
  oauthBindUser(form).then(data => {
    saveToken(data.token)
  }).catch(() => {
    bindLoading.value = false
  })
}

const submit = () => {

  if (loginLoading.value) return

  if (!form.email) {
    ElMessage({
      message: t('emptyEmailMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  let email = getFullEmail(form.email);

  if (!isEmail(email)) {
    ElMessage({
      message: t('notEmailMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  if (!form.password) {
    ElMessage({
      message: t('emptyPwdMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  loginLoading.value = true
  login(email, form.password).then(async data => {
    await saveToken(data.token)
  }).finally(() => {
    loginLoading.value = false
  })
}

async function saveToken(token) {
  localStorage.setItem('token', token)
  refreshWebsiteConfig()
  const user = await loginUserInfo();
  accountStore.currentAccountId = user.account.accountId;
  accountStore.currentAccount = user.account;
  userStore.user = user;
  const routers = permsToRouter(user.permKeys);
  routers.forEach(routerData => {
    router.addRoute('layout', routerData);
  });
  await router.replace({name: 'layout'})
  uiStore.showNotice()
  oauthLoading.value = false;
  bindLoading.value = false;
}

function refreshWebsiteConfig() {
  websiteConfig().then(setting => {
    settingStore.settings = setting
    settingStore.domainList = setting.domainList
    if (!suffix.value && setting.domainList.length > 0) {
      suffix.value = setting.domainList[0]
    }
    document.title = setting.title
  }).catch(e => {
    console.error(e)
  })
}


function submitRegister() {

  if (registerLoading.value) return

  if (!registerForm.email) {
    ElMessage({
      message: t('emptyEmailMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  console.log(registerForm.email)

  if (getEmailName(registerForm.email).length < settingStore.settings.minEmailPrefix) {
    ElMessage({
      message: t('minEmailPrefix', {msg: settingStore.settings.minEmailPrefix}),
      type: 'error',
      plain: true,
    })
    return
  }

  const email = getFullEmail(registerForm.email);

  if (!isEmail(email)) {
    ElMessage({
      message: t('notEmailMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  if (!registerForm.password) {
    ElMessage({
      message: t('emptyPwdMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  if (registerForm.password.length < 6) {
    ElMessage({
      message: t('pwdLengthMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  if (registerForm.password !== registerForm.confirmPassword) {

    ElMessage({
      message: t('confirmPwdFailMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  if (settingStore.settings.regKey === 0) {

    if (!registerForm.code) {

      ElMessage({
        message: t('emptyRegKeyMsg'),
        type: 'error',
        plain: true,
      })
      return
    }

  }

  if (!verifyToken && (settingStore.settings.registerVerify === 0 || (settingStore.settings.registerVerify === 2 && settingStore.settings.regVerifyOpen))) {
    if (!verifyShow.value) {
      verifyShow.value = true
      nextTick(() => {
        if (!turnstileId) {
          try {
            turnstileId = window.turnstile.render('.register-turnstile')
          } catch (e) {
            botJsError.value = true
            console.log('人机验证js加载失败')
          }
        } else {
          window.turnstile.reset('.register-turnstile')
        }
      })
    } else if (!botJsError.value) {
      ElMessage({
        message: t('botVerifyMsg'),
        type: "error",
        plain: true
      })
    }
    return;
  }

  registerLoading.value = true

  const form = {
    email,
    password: registerForm.password,
    token: verifyToken,
    code: registerForm.code
  }

  register(form).then(({regVerifyOpen}) => {
    show.value = 'login'
    registerForm.email = ''
    registerForm.password = ''
    registerForm.confirmPassword = ''
    registerForm.code = ''
    registerLoading.value = false
    verifyToken = ''
    settingStore.settings.regVerifyOpen = regVerifyOpen
    verifyShow.value = false
    ElMessage({
      message: t('regSuccessMsg'),
      type: 'success',
      plain: true,
    })
  }).catch(res => {

    registerLoading.value = false

    if (res.code === 400) {
      verifyToken = ''
      settingStore.settings.regVerifyOpen = true
      if (turnstileId) {
        window.turnstile.reset(turnstileId)
      } else {
        nextTick(() => {
          turnstileId = window.turnstile.render('.register-turnstile')
        })
      }
      verifyShow.value = true

    }
  });
}

</script>


<style>
.el-select-dropdown__item {
  padding: 0 15px;
}

.no-autofill-pwd {
  .el-input__inner {
    -webkit-text-security: disc !important;
  }
}
</style>

<style lang="scss" scoped>

.form-wrapper {
  position: fixed;
  right: 0;
  height: 100%;
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: center;
  @media (max-width: 767px) {
    width: 100%;
  }
}

.container {
  background: v-bind(loginOpacity);
  /* 让后缀的淡出读成有意为之，而不是"被切掉" */
  box-shadow: -18px 0 40px -20px rgba(12, 53, 87, .18);
  padding-left: 40px;
  padding-right: 40px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  width: var(--card-w);
  height: 100%;
  border-left: 1px solid var(--login-border);
  @media (max-width: 1024px) {
    padding: 20px 18px;
    width: 384px;
    margin-left: 18px;
  }
  @media (max-width: 767px) {
    border: 1px solid var(--login-border);
    padding: 20px 18px;
    border-radius: 6px;
    height: fit-content;
    width: 100%;
    margin-right: 18px;
    margin-left: 18px;
  }

  .btn {
    height: 36px;
    width: 100%;
    border-radius: 6px;
  }

  .form-desc {
    margin-top: 5px;
    margin-bottom: 18px;
    color: var(--form-desc-color);
  }

  .form-title {
    font-weight: bold;
    font-size: 22px !important;
  }

  .switch {
    margin-top: 20px;
    text-align: center;

    span {
      color: var(--login-switch-color);
      cursor: pointer;
    }
  }

  :deep(.el-input__wrapper) {
    border-radius: 6px;
    background: var(--el-bg-color);
  }

  .email-input :deep(.el-input__wrapper) {
    border-radius: 6px 0 0 6px;
    background: var(--el-bg-color);
  }

  .el-input {
    height: 38px;
    width: 100%;
    margin-bottom: 18px;

    :deep(.el-input__inner) {
      height: 36px;
    }
  }
}

:deep(.el-select-dropdown__item) {
  padding: 0 10px;
}

:deep(.bind-dialog) {
  width: 400px !important;
  @media (max-width: 440px) {
    width: calc(100% - 40px) !important;
    margin-right: 20px !important;
    margin-left: 20px !important;
  }
}

.bind-container {
  display: grid;
  grid-template-columns: 1fr;
  gap: 15px;
}

.setting-icon {
  position: relative;
  top: 6px;
}

.github {
  position: fixed;
  width: 35px;
  height: 35px;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 50%;
  background: var(--el-bg-color);
  bottom: 10px;
  right: 10px;
  z-index: 1000;
  border: 1px solid var(--el-border-color-light);
  box-shadow: var(--el-box-shadow-light);
  cursor: pointer;
}

:deep(.el-input-group__append) {
  padding: 0 !important;
  padding-left: 8px !important;
  padding-right: 4px !important;
  background: var(--el-bg-color);
  border-radius: 0 8px 8px 0;
}

:deep(.el-button+.el-button) {
  margin: 0;
}

.register-turnstile {
  margin-bottom: 18px;
}

.select {
  position: absolute;
  right: 30px;
  width: 100px;
  opacity: 0;
  pointer-events: none;
  visibility: hidden;
}

.custom-style {
  margin-bottom: 10px;
}

.custom-style .el-segmented {
  --el-border-radius-base: 6px;
  width: 180px;
}


#login-box {
  --card-w: 450px;
  background: linear-gradient(to bottom, #2980b9, #6dd5fa, #fff);
  font: 100% Arial, sans-serif;
  height: 100%;
  margin: 0;
  padding: 0;
  overflow-x: hidden;
  display: grid;
  grid-template-columns: 1fr;
}


/* --card-w = 登录卡实际占用的横向宽度（含 margin），--sky-w 靠它算后缀终点。
   窄屏下卡片不占满高度，天空仍是全宽，所以取 0。 */
@media (max-width: 1024px) {
  #login-box { --card-w: 402px; }
}

@media (max-width: 767px) {
  #login-box { --card-w: 0px; }
}

#background-wrap {
  height: 100%;
  z-index: 0;
  position: relative;
  /* #login-box 只写了 overflow-x:hidden。按规范一轴 hidden、另一轴 visible 时，
     visible 会被计算成 auto —— 于是绝对定位的云在矮视口上能撑出一条纵向滚动条，
     页面可被拖动、把居中的登录卡拖歪。装饰层自己关掉溢出，不动 #login-box
     （后者里还有小屏注册态可能超过一屏的表单）。 */
  overflow: hidden;
  --sky-w: calc(100vw - var(--card-w, 450px));
}

/* 雾罩：把最远的三样东西（L0/L1 云、远后缀）统一压到同一层"空气"后面。
   z-index 定在 38 而不是更高，near 侧全部保持锐利，边界干净。 */
#background-wrap::after {
  content: "";
  position: absolute;
  inset: 0;
  pointer-events: none;
  z-index: 38;
  background: linear-gradient(to bottom,
    rgba(238, 248, 253, 0) 0%,
    rgba(238, 248, 253, 0) 45%,
    rgba(238, 248, 253, .06) 62%,
    rgba(238, 248, 253, .18) 76%,
    rgba(238, 248, 253, .40) 90%,
    rgba(238, 248, 253, .52) 100%);
}

/* ── 云：三层嵌套。track 管水平漂移，bob 管垂直起伏，cloud 管缩放与造型。
   必须分三层：单个元素的 transform 会互相覆盖，而 animation-composition:add
   在老 WebView 上不支持时会让第二条动画直接顶掉第一条 —— 云会完全停住。 ── */
.cloud-track {
  position: absolute;
  left: 0;
  width: 100%;
  height: 0;
  animation-name: cloudDrift;
  animation-timing-function: linear;
  animation-iteration-count: infinite;
}

@keyframes cloudDrift {
  from { transform: translate3d(0, 0, 0); }
  to   { transform: translate3d(calc(100% + 350px * var(--s)), 0, 0); }
}

.cloud-bob {
  position: absolute;
  left: 0;
  top: 0;
  width: 0;
  height: 0;
  animation-name: cloudBob;
  animation-timing-function: cubic-bezier(.37, 0, .63, 1);
  animation-iteration-count: infinite;
}

@keyframes cloudBob {
  0%   { transform: translate3d(0, calc(var(--bob) * -1), 0); }
  50%  { transform: translate3d(0, var(--bob), 0); }
  100% { transform: translate3d(0, calc(var(--bob) * -1), 0); }
}

.cloud {
  position: absolute;
  left: 0;
  top: 0;
  width: 350px;
  height: 140px;
  transform-origin: 0 50%;
  /* scale 必须在 translateX 之前：写成 translateX(-100%) scale(s) 会在未缩放的
     坐标系里位移 350px，小云被多藏 350×(1−s) px，重新引入入场死区。 */
  transform: scale(var(--s), var(--sy)) translateX(-100%);
  /* filter 在 transform 之前生效，所以模糊半径要预先除掉 scale，
     否则 blur(1.1px) 配 scale(0.20) 在屏幕上只剩 0.22px。 */
  filter: blur(calc(var(--soft, 0px) / var(--s)));
  pointer-events: none;
}

/* 一条连续的 SVG 轮廓避免多个圆球叠成“煎饼”。底部腹部和左上高光
   只负责表现体积，不改变云的整体轮廓。 */
.cloud-svg {
  display: block;
  width: 100%;
  height: 100%;
  overflow: visible;
}

.cloud-silhouette {
  fill: var(--body);
}

.cloud-belly {
  fill: var(--belly);
  opacity: .54;
}

.cloud-highlight {
  fill: #fff;
  opacity: var(--highlight, .26);
}

/* ── 邮箱后缀 ── */
.suffix-drift {
  position: absolute;
  inset: 0;
  pointer-events: none;
  /* 基准字号必须抬上来，否则拉大轨道间极差后远端会跌破可读下限 */
  font-size: clamp(14px, 1.02vw, 15.5px);
}

.suffix-drift.far  { z-index: 30; }
.suffix-drift.near { z-index: 70; }

.chip-track {
  position: absolute;
  left: 0;
  top: 0;
  /* 承约束：非线性缓动会让同轨道的后缀瞬时速度不同、间距在周期内出现最小值，
     "等间隔 + 同速 ⇒ 永不重叠"的推导立刻失效。勿改。 */
  animation-timing-function: linear;
  animation-iteration-count: infinite;
}

.suffix-chip {
  display: inline-block;
  white-space: nowrap;
  /* 所有内部几何量都用 em：原来字号用 em 而 padding/box-shadow 写死 px，
     等于在同一个盒子里塞不同大小的字，尺寸线索被自我抵消掉一半 ——
     这才是"极差拉大了却感觉不明显"的真因。border 是唯一保留 px 的
     （亚像素描边会整条消失）。 */
  padding: .34em .92em;
  border-radius: 99em;
  border: 1px solid var(--bd);
  background: var(--fill);
  color: #fff;
  text-shadow: 0 .5px 0 rgba(0, 0, 0, .10);
  box-shadow: 0 .12em .30em rgba(6, 30, 52, var(--shA));
  line-height: 1.45;
  /* 全程 700 不递减：小字需要更多而不是更少的笔画重量 */
  font-weight: 700;
  letter-spacing: .02em;
  font-family: ui-rounded, "SF Pro Rounded", "PingFang SC", "Hiragino Maru Gothic ProN",
  "Segoe UI Variable Display", "Segoe UI", system-ui, -apple-system, "Helvetica Neue", sans-serif;
}

/* 八档过屏占比。淡入淡出的百分比写成 ratio 的倍数：因为 base = CROSS/ratio
   已把过屏时间锁死在 20 秒，所以各档的淡入淡出在屏幕上走过的距离和墙钟时长
   完全一致（淡入 1.2s / 淡出 2.4s）。写成固定百分比会让最低档的胶囊在屏幕
   正中间就开始淡出。
   绝不能把 visibility 写进这组关键帧：Blink 对关键帧动画是整条判定能否上
   合成器，加一行 visibility 会让 220 个胶囊从 120fps 掉到 48.6fps。
   屏外驻留一律用 opacity:0。 */
$rates: (a: .75, b: .60, c: .45, d: .34, e: .26, f: .18, g: .13, h: .09);

@each $k, $r in $rates {
  @keyframes driftSuffix-#{$k} {
    0%                  { transform: translate3d(-280px, 0, 0); opacity: 0; }
    #{$r * 6}%          { opacity: var(--op, .96); }
    #{$r * 88}%         { opacity: var(--op, .96); }
    #{$r * 100}%, 100%  { transform: translate3d(var(--sky-w), 0, 0); opacity: 0; }
  }
  .chip-track.rate-#{$k} { animation-name: driftSuffix-#{$k}; }
}

/* 左下角后缀总数。原来它跟飘动的后缀用完全相同的配方（同背景/圆角/padding/
   边框/阴影），会被读成"一个卡住不动的后缀"；而且它坐在 top≈98% 的纯白上，
   只剩描边撑着约 1.10:1。改成墨底白字后 14.3:1，且明确"贴在玻璃上"。
   z-index 从 11 降到 9，防止小屏上盖住 z-index:10 的登录卡。 */
.domain-count {
  position: fixed;
  left: 20px;
  bottom: 20px;
  z-index: 9;
  padding: 6px 14px;
  border-radius: 999px;
  background: #0A2C4A;
  color: #fff;
  font-family: ui-rounded, "SF Pro Rounded", "PingFang SC", "Hiragino Maru Gothic ProN",
  "Segoe UI Variable Display", "Segoe UI", system-ui, -apple-system, "Helvetica Neue", sans-serif;
  font-size: 12.5px;
  font-weight: 700;
  letter-spacing: .01em;
  font-variant-numeric: tabular-nums;
  border: 1px solid rgba(255, 255, 255, .22);
  box-shadow: 0 2px 4px rgba(23, 66, 102, .12), 0 8px 20px -6px rgba(23, 66, 102, .22);
  pointer-events: none;
  user-select: none;
}

/* 移动端：登录卡占满宽度，可用天空只剩顶部一条窄带。轨道压到卡片上沿之上，
   后缀减量到 16 个 / 4 条轨道，云减到 7 朵，并全局关掉 filter。
   render / lanes / 轨道带这三个数是配套的，不能单独调其中一个。 */
@media (max-width: 767px) {
  .suffix-drift {
    font-size: clamp(11.5px, 3.2vw, 13px);
  }

  .domain-count {
    left: 12px;
    bottom: 12px;
    font-size: 11.5px;
    padding: 5px 12px;
  }
}

/* reduced-motion 的语义是"去掉运动"，不是"去掉内容"。旧实现把 55 个后缀整层
   抹掉、却把 16 朵大面积慢速平移的云一朵都没关 —— 而后者对前庭敏感用户的
   刺激更强。改成冻结：静态构图恰好是动画的 t=0 帧，而防斜线的 leadSlope
   逻辑已经保证那一帧是散开的。 */
@media (prefers-reduced-motion: reduce) {
  .cloud-track,
  .cloud-bob,
  .chip-track {
    animation: none !important;
  }

  .cloud-track { transform: translate3d(var(--rest), 0, 0); }
  .cloud-bob   { transform: none; }

  .chip-track {
    transform: translate3d(var(--rest), 0, 0);
    opacity: var(--op, .96);
  }

  .chip-track.drift-off { display: none; }
}

</style>

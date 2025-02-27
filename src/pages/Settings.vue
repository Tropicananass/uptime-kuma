<template>
    <transition name="slide-fade" appear>
        <div>
            <h1 v-show="show" class="mb-3">
                {{ $t("Settings") }}
            </h1>

            <div class="shadow-box">
                <div class="row">
                    <div class="col-md-6">
                        <h2 class="mb-2">{{ $t("Appearance") }}</h2>

                        <div class="mb-3">
                            <label for="language" class="form-label">{{ $t("Language") }}</label>
                            <select id="language" v-model="$i18n.locale" class="form-select">
                                <option v-for="(lang, i) in $i18n.availableLocales" :key="`Lang${i}`" :value="lang">
                                    {{ $i18n.messages[lang].languageName }}
                                </option>
                            </select>
                        </div>

                        <div class="mb-3">
                            <label for="timezone" class="form-label">{{ $t("Theme") }}</label>

                            <div>
                                <div class="btn-group" role="group" aria-label="Basic checkbox toggle button group">
                                    <input id="btncheck1" v-model="$root.userTheme" type="radio" class="btn-check" name="theme" autocomplete="off" value="light">
                                    <label class="btn btn-outline-primary" for="btncheck1">{{ $t("Light") }}</label>

                                    <input id="btncheck2" v-model="$root.userTheme" type="radio" class="btn-check" name="theme" autocomplete="off" value="dark">
                                    <label class="btn btn-outline-primary" for="btncheck2">{{ $t("Dark") }}</label>

                                    <input id="btncheck3" v-model="$root.userTheme" type="radio" class="btn-check" name="theme" autocomplete="off" value="auto">
                                    <label class="btn btn-outline-primary" for="btncheck3">{{ $t("Auto") }}</label>
                                </div>
                            </div>
                        </div>

                        <div class="mb-3">
                            <label class="form-label">{{ $t("Theme - Heartbeat Bar") }}</label>
                            <div>
                                <div class="btn-group" role="group" aria-label="Basic checkbox toggle button group">
                                    <input id="btncheck4" v-model="$root.userHeartbeatBar" type="radio" class="btn-check" name="heartbeatBarTheme" autocomplete="off" value="normal">
                                    <label class="btn btn-outline-primary" for="btncheck4">{{ $t("Normal") }}</label>

                                    <input id="btncheck5" v-model="$root.userHeartbeatBar" type="radio" class="btn-check" name="heartbeatBarTheme" autocomplete="off" value="bottom">
                                    <label class="btn btn-outline-primary" for="btncheck5">{{ $t("Bottom") }}</label>

                                    <input id="btncheck6" v-model="$root.userHeartbeatBar" type="radio" class="btn-check" name="heartbeatBarTheme" autocomplete="off" value="none">
                                    <label class="btn btn-outline-primary" for="btncheck6">{{ $t("None") }}</label>
                                </div>
                            </div>
                        </div>

                        <h2 class="mt-5 mb-2">{{ $t("General") }}</h2>
                        <form class="mb-3" @submit.prevent="saveGeneral">
                            <div class="mb-3">
                                <label for="timezone" class="form-label">{{ $t("Timezone") }}</label>
                                <select id="timezone" v-model="$root.userTimezone" class="form-select">
                                    <option value="auto">
                                        {{ $t("Auto") }}: {{ guessTimezone }}
                                    </option>
                                    <option v-for="(timezone, index) in timezoneList" :key="index" :value="timezone.value">
                                        {{ timezone.name }}
                                    </option>
                                </select>
                            </div>

                            <div class="mb-3">
                                <label class="form-label">{{ $t("Search Engine Visibility") }}</label>

                                <div class="form-check">
                                    <input id="searchEngineIndexYes" v-model="settings.searchEngineIndex" class="form-check-input" type="radio" name="flexRadioDefault" :value="true" required>
                                    <label class="form-check-label" for="searchEngineIndexYes">
                                        {{ $t("Allow indexing") }}
                                    </label>
                                </div>
                                <div class="form-check">
                                    <input id="searchEngineIndexNo" v-model="settings.searchEngineIndex" class="form-check-input" type="radio" name="flexRadioDefault" :value="false" required>
                                    <label class="form-check-label" for="searchEngineIndexNo">
                                        {{ $t("Discourage search engines from indexing site") }}
                                    </label>
                                </div>
                            </div>

                            <div>
                                <button class="btn btn-primary" type="submit">
                                    {{ $t("Save") }}
                                </button>
                            </div>
                        </form>

                        <template v-if="loaded">
                            <template v-if="! settings.disableAuth">
                                <h2 class="mt-5 mb-2">{{ $t("Change Password") }}</h2>
                                <form class="mb-3" @submit.prevent="savePassword">
                                    <div class="mb-3">
                                        <label for="current-password" class="form-label">{{ $t("Current Password") }}</label>
                                        <input id="current-password" v-model="password.currentPassword" type="password" class="form-control" required>
                                    </div>

                                    <div class="mb-3">
                                        <label for="new-password" class="form-label">{{ $t("New Password") }}</label>
                                        <input id="new-password" v-model="password.newPassword" type="password" class="form-control" required>
                                    </div>

                                    <div class="mb-3">
                                        <label for="repeat-new-password" class="form-label">{{ $t("Repeat New Password") }}</label>
                                        <input id="repeat-new-password" v-model="password.repeatNewPassword" type="password" class="form-control" :class="{ 'is-invalid' : invalidPassword }" required>
                                        <div class="invalid-feedback">
                                            {{ $t("passwordNotMatchMsg") }}
                                        </div>
                                    </div>

                                    <div>
                                        <button class="btn btn-primary" type="submit">
                                            {{ $t("Update Password") }}
                                        </button>
                                    </div>
                                </form>
                            </template>

                            <h2 class="mt-5 mb-2">{{ $t("Advanced") }}</h2>

                            <div class="mb-3">
                                <button v-if="settings.disableAuth" class="btn btn-outline-primary me-1" @click="enableAuth">{{ $t("Enable Auth") }}</button>
                                <button v-if="! settings.disableAuth" class="btn btn-primary me-1" @click="confirmDisableAuth">{{ $t("Disable Auth") }}</button>
                                <button v-if="! settings.disableAuth" class="btn btn-danger me-1" @click="$root.logout">{{ $t("Logout") }}</button>
                            </div>
                        </template>
                    </div>

                    <div class="notification-list col-md-6">
                        <div v-if="$root.isMobile" class="mt-3" />

                        <h2>{{ $t("Notifications") }}</h2>
                        <p v-if="$root.notificationList.length === 0">
                            {{ $t("Not available, please setup.") }}
                        </p>
                        <p v-else>
                            {{ $t("notificationDescription") }}
                        </p>

                        <ul class="list-group mb-3" style="border-radius: 1rem;">
                            <li v-for="(notification, index) in $root.notificationList" :key="index" class="list-group-item">
                                {{ notification.name }}<br>
                                <a href="#" @click="$refs.notificationDialog.show(notification.id)">{{ $t("Edit") }}</a>
                            </li>
                        </ul>

                        <button class="btn btn-primary me-2" type="button" @click="$refs.notificationDialog.show()">
                            {{ $t("Setup Notification") }}
                        </button>
                    </div>
                </div>
            </div>

            <footer>
                <div class="container-fluid">
                    Uptime Kuma -
                    {{ $t("Version") }}: {{ $root.info.version }} -
                    <a href="https://github.com/louislam/uptime-kuma/releases" target="_blank" rel="noopener">{{ $t("Check Update On GitHub") }}</a>
                </div>
            </footer>

            <NotificationDialog ref="notificationDialog" />

            <Confirm ref="confirmDisableAuth" btn-style="btn-danger" :yes-text="$t('I understand, please disable')" :no-text="$t('Leave')" @yes="disableAuth">
                <template v-if="$i18n.locale === 'en' ">
                    <p>Are you sure want to <strong>disable auth</strong>?</p>
                    <p>It is for <strong>someone who have 3rd-party auth</strong> in front of Uptime Kuma such as Cloudflare Access.</p>
                    <p>Please use it carefully.</p>
                </template>

                <template v-if="$i18n.locale === 'es-ES' ">
                    <p>Seguro que deseas <strong>deshabilitar la autenticación</strong>?</p>
                    <p>Es para <strong>quien implementa autenticación de terceros</strong> ante Uptime Kuma como por ejemplo Cloudflare Access.</p>
                    <p>Por favor usar con cuidado.</p>
                </template>

                <template v-if="$i18n.locale === 'zh-HK' ">
                    <p>你是否確認<strong>取消登入認証</strong>？</p>
                    <p>這個功能是設計給已有<strong>第三方認証</strong>的用家，例如 Cloudflare Access。</p>
                    <p>請小心使用。</p>
                </template>

                <template v-if="$i18n.locale === 'zh-CN' ">
                    <p>是否确定 <strong>取消登录验证</strong>？</p>
                    <p>这是为 <strong>有第三方认证</strong> 的用户提供的功能，如 Cloudflare Access</p>
                    <p>请谨慎使用！</p>
                </template>

                <template v-if="$i18n.locale === 'de-DE' ">
                    <p>Bist du sicher das du die <strong>Authentifizierung deaktivieren</strong> möchtest?</p>
                    <p>Es ist für <strong>jemanden der eine externe Authentifizierung</strong> vor Uptime Kuma geschaltet hat, wie z.B. Cloudflare Access.</p>
                    <p>Bitte mit Vorsicht nutzen.</p>
                </template>

                <template v-if="$i18n.locale === 'sr' ">
                    <p>Да ли сте сигурни да желите да <strong>искључите аутентификацију</strong>?</p>
                    <p>То је за <strong>оне који имају додату аутентификацију</strong> испред Uptime Kuma као на пример Cloudflare Access.</p>
                    <p>Молим Вас користите ово са пажњом.</p>
                </template>

                <template v-if="$i18n.locale === 'sr-latn' ">
                    <p>Da li ste sigurni da želite da <strong>isključite autentifikaciju</strong>?</p>
                    <p>To je za <strong>one koji imaju dodatu autentifikaciju</strong> ispred Uptime Kuma kao na primer Cloudflare Access.</p>
                    <p>Molim Vas koristite ovo sa pažnjom.</p>
                </template>
            </Confirm>
        </div>
    </transition>
</template>

<script>
import Confirm from "../components/Confirm.vue";
import dayjs from "dayjs";
import utc from "dayjs/plugin/utc"
import timezone from "dayjs/plugin/timezone"
import NotificationDialog from "../components/NotificationDialog.vue";
dayjs.extend(utc)
dayjs.extend(timezone)

import { timezoneList } from "../util-frontend";
import { useToast } from "vue-toastification"
const toast = useToast()

export default {
    components: {
        NotificationDialog,
        Confirm,
    },
    data() {
        return {
            timezoneList: timezoneList(),
            guessTimezone: dayjs.tz.guess(),
            show: true,
            invalidPassword: false,
            password: {
                currentPassword: "",
                newPassword: "",
                repeatNewPassword: "",
            },
            settings: {

            },
            loaded: false,
        }
    },
    watch: {
        "password.repeatNewPassword"() {
            this.invalidPassword = false;
        },

        "$i18n.locale"() {
            localStorage.locale = this.$i18n.locale;
        },
    },

    mounted() {
        this.loadSettings();
    },

    methods: {

        saveGeneral() {
            localStorage.timezone = this.$root.userTimezone;
            this.saveSettings();
        },

        savePassword() {
            if (this.password.newPassword !== this.password.repeatNewPassword) {
                this.invalidPassword = true;
            } else {
                this.$root.getSocket().emit("changePassword", this.password, (res) => {
                    this.$root.toastRes(res)
                    if (res.ok) {
                        this.password.currentPassword = ""
                        this.password.newPassword = ""
                        this.password.repeatNewPassword = ""
                    }
                })
            }
        },

        loadSettings() {
            this.$root.getSocket().emit("getSettings", (res) => {
                this.settings = res.data;

                if (this.settings.searchEngineIndex === undefined) {
                    this.settings.searchEngineIndex = false;
                }

                this.loaded = true;
            })
        },

        saveSettings() {
            this.$root.getSocket().emit("setSettings", this.settings, (res) => {
                this.$root.toastRes(res);
                this.loadSettings();
            })
        },

        confirmDisableAuth() {
            this.$refs.confirmDisableAuth.show();
        },

        disableAuth() {
            this.settings.disableAuth = true;
            this.saveSettings();
        },

        enableAuth() {
            this.settings.disableAuth = false;
            this.saveSettings();
            this.$root.storage().removeItem("token");
        },

    },
}
</script>

<style lang="scss" scoped>
@import "../assets/vars.scss";

.shadow-box {
    padding: 20px;
}

.btn-check:active + .btn-outline-primary,
.btn-check:checked + .btn-outline-primary,
.btn-check:hover + .btn-outline-primary {
    color: #fff;
}

.dark {
    .list-group-item {
        background-color: $dark-bg2;
        color: $dark-font-color;
    }

    .btn-check:active + .btn-outline-primary,
    .btn-check:checked + .btn-outline-primary,
    .btn-check:hover + .btn-outline-primary {
        color: #000;
    }
}

footer {
    color: #aaa;
    font-size: 13px;
    margin-top: 20px;
    padding-bottom: 30px;
    text-align: center;
}
</style>

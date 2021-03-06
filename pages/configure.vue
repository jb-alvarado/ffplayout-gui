<template>
    <div>
        <Menu />
        <b-card no-body>
            <b-tabs pills card vertical>
                <b-tab title="GUI" active @click="resetAlert()">
                    <b-container class="config-container">
                        <b-form v-if="configGui" @submit="onSubmitGui">
                            <b-form-group
                                label-cols-lg="2"
                                label="GUI Configuration"
                                label-size="lg"
                                label-class="font-weight-bold pt-0"
                                class="config-group"
                            >
                                <div style="width: 100%; height: 43px;">
                                    <div class="float-right">
                                        <b-button v-if="multiChannel" size="sm" variant="primary" class="m-md-2" @click="addChannel()">
                                            Add new Channel
                                        </b-button>
                                    </div>
                                </div>
                                <div v-for="(prop, name, idx) in configGui[configID]" :key="idx">
                                    <b-form-group
                                        v-if="idx >= 1 && name !== 'engine_service'"
                                        label-cols-sm="2"
                                        :label="name"
                                        label-align-sm="right"
                                        :label-for="name"
                                    >
                                        <b-form-tags
                                            v-if="name === 'extra_extensions'"
                                            v-model="configGui[configID][name]"
                                            :input-id="name"
                                            separator=" ,;"
                                            :placeholder="`add ${name}...`"
                                            class="mb-2 tags-list"
                                        />
                                        <b-form-text v-if="name === 'extra_extensions'">
                                            Visible extensions only for the GUI and not the playout
                                        </b-form-text>
                                        <b-form-select v-else-if="name === 'net_interface'" :id="name" v-model="configGui[configID][name]" :options="netChoices" :value="prop" />
                                        <b-form-input
                                            v-else
                                            :id="name"
                                            v-model="configGui[configID][name]"
                                            :value="prop"
                                            :disabled="(name === 'playout_config') ? true : false"
                                        />
                                    </b-form-group>
                                </div>
                            </b-form-group>
                            <b-row>
                                <b-col cols="1" style="min-width:158px">
                                    <b-button-group>
                                        <b-button type="submit" variant="primary">
                                            Save
                                        </b-button>
                                        <b-button v-if="multiChannel && configGui[configID].id > 1" variant="danger" @click="deleteChannel()">
                                            Delete
                                        </b-button>
                                    </b-button-group>
                                </b-col>
                                <b-col>
                                    <b-alert v-model="showAlert" :variant="alertVariant" dismissible>
                                        {{ alertMsg }}
                                    </b-alert>
                                </b-col>
                            </b-row>
                        </b-form>
                    </b-container>
                </b-tab>
                <b-tab title="Playout" @click="resetAlert()">
                    <b-container class="config-container">
                        <b-form v-if="configPlayout" @submit="onSubmitPlayout">
                            <b-form-group
                                v-for="(item, key, index) in configPlayout"
                                :key="index"
                                label-cols-lg="2"
                                :label="key"
                                label-size="lg"
                                label-class="font-weight-bold pt-0"
                                class="config-group"
                            >
                                <b-form-group
                                    v-for="(prop, name, idx) in item"
                                    :key="idx"
                                    label-cols-sm="2"
                                    :label="(typeof prop === 'boolean' || name === 'helptext') ? '' : name"
                                    label-align-sm="right"
                                    :label-for="name"
                                >
                                    <b-form-textarea
                                        v-if="name === 'helptext'"
                                        id="textarea-plaintext"
                                        plaintext
                                        :value="prop"
                                        rows="2"
                                        max-rows="8"
                                        class="text-area"
                                    />
                                    <b-form-checkbox
                                        v-else-if="typeof prop === 'boolean'"
                                        :id="name"
                                        v-model="configPlayout[key][name]"
                                        :name="name"
                                    >
                                        {{ name }}
                                    </b-form-checkbox>
                                    <b-form-input
                                        v-else-if="prop && prop.toString().match(/^-?\d+[.,]\d+$/)"
                                        :id="name"
                                        v-model.number="configPlayout[key][name]"
                                        type="number"
                                        step="0.001"
                                        class="input-field"
                                    />
                                    <b-form-input
                                        v-else-if="prop && !isNaN(prop)"
                                        :id="name"
                                        v-model.number="configPlayout[key][name]"
                                        type="number"
                                        step="1"
                                        class="input-field"
                                    />
                                    <b-form-tags
                                        v-else-if="Array.isArray(prop)"
                                        v-model="configPlayout[key][name]"
                                        :input-id="name"
                                        separator=" ,;"
                                        :placeholder="`add ${name}...`"
                                        class="mb-2 tags-list"
                                    />
                                    <b-form-input
                                        v-else-if="name.includes('pass')"
                                        :id="name"
                                        v-model="configPlayout[key][name]"
                                        type="password"
                                        :value="prop"
                                    />
                                    <b-form-input v-else :id="name" v-model="configPlayout[key][name]" :value="prop" />
                                </b-form-group>
                            </b-form-group>

                            <b-row>
                                <b-col cols="1" style="min-width: 85px">
                                    <b-button type="submit" variant="primary">
                                        Save
                                    </b-button>
                                </b-col>
                                <b-col>
                                    <b-alert v-model="showAlert" :variant="alertVariant" dismissible>
                                        {{ alertMsg }}
                                    </b-alert>
                                </b-col>
                            </b-row>
                        </b-form>
                    </b-container>
                </b-tab>
                <b-tab title="User" @click="resetAlert()">
                    <b-card-text>
                        <b-container class="config-container">
                            <b-form v-if="configUser" @submit="onSubmitUser">
                                <b-form-group
                                    label-cols-lg="2"
                                    label="User Configuration"
                                    label-size="lg"
                                    label-class="font-weight-bold pt-0"
                                    class="config-group"
                                >
                                    <b-form-group
                                        label-cols-sm="2"
                                        :label="'username'"
                                        label-align-sm="right"
                                        :label-for="'username'"
                                    >
                                        <b-form-input id="username" v-model="configUser['username']" :value="configUser['username']" disabled />
                                    </b-form-group>
                                    <b-form-group
                                        label-cols-sm="2"
                                        :label="'email'"
                                        label-align-sm="right"
                                        :label-for="'email'"
                                    >
                                        <b-form-input id="email" v-model="configUser['email']" :value="configUser['email']" />
                                    </b-form-group>
                                    <b-form-group
                                        label-cols-sm="2"
                                        label="old password"
                                        label-align-sm="right"
                                        label-for="oldPass"
                                    >
                                        <b-form-input id="oldPass" v-model="oldPass" type="password" />
                                    </b-form-group>
                                    <b-form-group
                                        label-cols-sm="2"
                                        label="new password"
                                        label-align-sm="right"
                                        label-for="newPass"
                                    >
                                        <b-form-input id="newPass" v-model="newPass" type="password" />
                                    </b-form-group>
                                    <b-form-group
                                        label-cols-sm="2"
                                        label="confirm password"
                                        label-align-sm="right"
                                        label-for="confirmPass"
                                    >
                                        <b-form-input id="confirmPass" v-model="confirmPass" type="password" />
                                    </b-form-group>
                                </b-form-group>
                                <b-row>
                                    <b-col cols="1" style="min-width: 85px">
                                        <b-button type="submit" variant="primary">
                                            Save
                                        </b-button>
                                    </b-col>
                                    <b-col>
                                        <b-alert v-model="showAlert" :variant="alertVariant" dismissible>
                                            {{ alertMsg }}
                                        </b-alert>
                                    </b-col>
                                </b-row>
                            </b-form>
                        </b-container>
                    </b-card-text>
                </b-tab>
            </b-tabs>
        </b-card>
    </div>
</template>

<script>
import { mapState } from 'vuex'
import Menu from '@/components/Menu.vue'

export default {
    name: 'Configure',

    components: {
        Menu
    },

    middleware: 'auth',

    data () {
        return {
            oldPass: null,
            newPass: null,
            confirmPass: null,
            showAlert: false,
            alertVariant: 'success',
            alertMsg: ''
        }
    },

    computed: {
        ...mapState('config', ['configID', 'netChoices', 'multiChannel']),
        configGui: {
            get () {
                return this.$store.state.config.configGui
            },
            set (config) {
                this.$store.commit('config/UPDATE_GUI_CONFIG', config)
            }
        },
        configPlayout: {
            get () {
                return this.$store.state.config.configPlayout
            },
            set (config) {
                this.$store.commit('config/UPDATE_PLAYLOUT_CONFIG', config)
            }
        },
        configUser: {
            get () {
                return this.$store.state.config.configUser
            },
            set (config) {
                this.$store.commit('config/UPDATE_USER_CONFIG', config)
            }
        }
    },

    methods: {
        addChannel () {
            const config = this.$_.cloneDeep(this.configGui)
            const newConf = this.$_.cloneDeep(this.configGui[this.configGui.length - 1])

            const playoutConfigPath = newConf.playout_config.match(/.*\//)
            const playoutConfigFile = newConf.playout_config.replace(/(.*\/|\.yml)/g, '').split('-')

            const engineService = newConf.engine_service.split('-')

            newConf.id = config.length + 1
            newConf.channel = `Channel ${Math.random().toString(36).substring(7)}`
            newConf.playout_config = `${playoutConfigPath}${playoutConfigFile[0]}-${String(parseInt(playoutConfigFile[1]) + 1).padStart(3, '0')}.yml`
            newConf.engine_service = `${engineService[0]}-${String(parseInt(engineService[1]) + 1).padStart(3, '0')}`

            config.push(newConf)

            this.$store.commit('config/UPDATE_GUI_CONFIG', config)
            this.$store.commit('config/UPDATE_CONFIG_ID', this.configGui.length - 1)
        },
        async onSubmitGui (evt) {
            evt.preventDefault()
            await this.$store.dispatch('auth/inspectToken')
            const update = await this.$store.dispatch('config/setGuiConfig', this.configGui[this.configID])

            if (update.status === 200 || update.status === 201) {
                this.alertVariant = 'success'
                this.alertMsg = 'Update GUI config success!'
            } else {
                this.alertVariant = 'danger'
                this.alertMsg = 'Update GUI config failed!'
            }

            this.showAlert = true
        },
        async deleteChannel () {
            const config = this.$_.cloneDeep(this.configGui)
            const id = config[this.configID].id

            if (id === 1) {
                this.alertVariant = 'warning'
                this.alertMsg = 'First channel can not be deleted!'
                this.showAlert = true
                return
            }
            const response = await this.$axios.delete(`api/player/guisettings/${id}/`)

            config.splice(this.configID, 1)

            this.$store.commit('config/UPDATE_GUI_CONFIG', config)
            this.$store.commit('config/UPDATE_CONFIG_ID', this.configGui.length - 1)
            await this.$store.dispatch('config/getPlayoutConfig')

            if (response.status === 204) {
                this.alertVariant = 'success'
                this.alertMsg = 'Delete GUI config success!'
            } else {
                this.alertVariant = 'danger'
                this.alertMsg = 'Delete GUI config failed!'
            }

            this.showAlert = true
        },
        async onSubmitPlayout (evt) {
            evt.preventDefault()
            await this.$store.dispatch('auth/inspectToken')
            const update = await this.$store.dispatch('config/setPlayoutConfig', this.configPlayout)

            if (update.status === 200) {
                this.alertVariant = 'success'
                this.alertMsg = 'Update playout config success!'
            } else {
                this.alertVariant = 'danger'
                this.alertMsg = 'Update playout config failed!'
            }

            this.showAlert = true
        },
        async onSubmitUser (evt) {
            evt.preventDefault()
            if (this.oldPass && this.newPass && this.newPass === this.confirmPass) {
                this.configUser.old_password = this.oldPass
                this.configUser.new_password = this.newPass
            }
            await this.$store.dispatch('auth/inspectToken')
            const update = await this.$store.dispatch('config/setUserConfig', this.configUser)

            if (update.status === 200) {
                this.alertVariant = 'success'
                this.alertMsg = 'Update user profil success!'
            } else {
                this.alertVariant = 'danger'
                this.alertMsg = 'Update user profil failed!'
            }

            this.showAlert = true

            this.oldPass = null
            this.newPass = null
            this.confirmPass = null
        },

        resetAlert () {
            this.showAlert = false
            this.alertVariant = 'success'
            this.alertMsg = ''
        }
    }
}
</script>

<style lang="scss">
.config-container {
    margin: 2em auto 2em auto;
    padding: 0;
}

.config-group {
    margin-bottom: 2em;
}

.input-field {
    max-width: 200px;
}

.text-area {
    overflow-y: hidden !important;
}
</style>

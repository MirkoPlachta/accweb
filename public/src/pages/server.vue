<template>
    <layout>
        <div class="title">
            <h1>{{servername}}</h1>
            <div class="menu">
                <button class="primary" v-on:click="save" v-if="is_admin"><i class="fas fa-save"></i> {{$t("save")}}</button>
                <button v-on:click="$router.push('/')" v-if="is_admin"><i class="fas fa-ban"></i> {{$t("cancel")}}</button>
                <button class="primary" v-on:click="$router.push('/')" v-if="!is_admin"><i class="fas fa-arrow-left"></i> {{$t("back")}}</button>
            </div>
        </div>
        <div class="tabs">
            <div v-bind:class="{tab: true, 'tab-active': activeTab === 0}" v-on:click="activeTab = 0">{{$t("server_config")}}</div>
            <div v-bind:class="{tab: true, 'tab-active': activeTab === 1}" v-on:click="activeTab = 1" v-if="is_admin && !id">{{$t("import_server")}}</div>
        </div>
        <div v-show="activeTab === 0">
            <basic ref="basic"></basic>
            <settings ref="settings"></settings>
            <event ref="event"></event>
            <eventrules ref="eventrules"></eventrules>
            <entrylist ref="entrylist"></entrylist>
        </div>
        <div v-show="activeTab === 1">
            <p>{{$t("upload_hint")}}</p>
            <form v-on:submit.prevent="importServer">
                <label>configuration.json</label>
                <input type="file" name="configuration.json" v-on:change="configurationJsonListener" />
                <label>settings.json</label>
                <input type="file" name="settings.json" v-on:change="settingsJsonListener" />
                <label>event.json</label>
                <input type="file" name="event.json" v-on:change="eventJsonListener" />
                <label>eventRules.json</label>
                <input type="file" name="eventRules.json" v-on:change="eventRulesJsonListener" />
                <label>entrylist.json</label>
                <input type="file" name="entrylist.json" v-on:change="entrylistJsonListener" />
                <input class="primary" type="submit" :value="$t('import_button')" />
            </form>
        </div>
    </layout>
</template>

<script>
import axios from "axios";
import {layout, end, basic, settings, event, eventrules, entrylist} from "../components";

export default {
    components: {layout, end, basic, settings, event, eventrules, entrylist},
    data() {
        return {
            activeTab: 0,
            id: 0,
            servername: "New Server",
            configurationJson: null,
            settingsJson: null,
            eventJson: null,
            eventRulesJson: null,
            entrylistJson: null
        };
    },
    mounted() {
        this.id = this.$route.query.id;

        if(this.id) {
            this.loadServer();
        }
    },
    methods: {
        loadServer() {
            axios.get("/api/server", {params: {id: this.id}})
            .then(r => {
                this.servername = r.data.settings.serverName;
                this.$refs.basic.setData(r.data.basic);
                this.$refs.settings.setData(r.data.settings);
                this.$refs.event.setData(r.data.event);
                this.$refs.eventrules.setData(r.data.eventRules);
                this.$refs.entrylist.setData(r.data.entrylist);
            });
        },
        save() {
            let basic = this.$refs.basic.getData();
            let settings = this.$refs.settings.getData();
            let event = this.$refs.event.getData();
            let eventRules = this.$refs.eventrules.getData();
            let entrylist = this.$refs.entrylist.getData();
            let data = {
                id: parseInt(this.id),
                basic,
                settings,
                event,
                eventRules,
                entrylist
            };

            axios.post("/api/server", data)
            .then(() => {
                this.$router.push("/");
            })
            .catch(e => {
                this.$store.commit("toast", this.$t("save_error"))
            });
        },
        configurationJsonListener(e) {
            this.configurationJson = e.target.files[0];
        },
        settingsJsonListener(e) {
            this.settingsJson = e.target.files[0];
        },
        eventJsonListener(e) {
            this.eventJson = e.target.files[0];
        },
        eventRulesJsonListener(e) {
            this.eventRulesJson = e.target.files[0];
        },
        entrylistJsonListener(e) {
            this.entrylistJson = e.target.files[0];
        },
        importServer() {
            let data = new FormData();
            data.append("configuration", this.configurationJson);
            data.append("settings", this.settingsJson);
            data.append("event", this.eventJson);
            data.append("eventRules", this.eventRulesJson);
            data.append("entrylist", this.entrylistJson);

            let headers = {headers: {"Content-Type": "multipart/form-data"}};

            axios.post("/api/server/import", data, headers)
            .then(() => {
                this.$router.push("/");
            })
            .catch(e => {
                this.$store.commit("toast", this.$t("import_error"))
            });
        }
    }
}
</script>

<i18n>
{
    "en": {
        "save": "Save",
        "cancel": "Cancel",
        "back": "Back",
        "server_config": "Configure server",
        "import_server": "Import server",
        "upload_hint": "You can import an existing server by uploading its configuration files. Select the files of your server configuration and press import.",
        "import_button": "Import",
        "save_error": "Error saving configuration, please check your input.",
        "import_error": "Error importing configuration files, please check your input."
    }
}
</i18n>

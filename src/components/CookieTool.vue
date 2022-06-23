<template>
    <div class="centered">
        <table>
            <tr>
                <th>Domain attribute</th>
                <th>Domain who set the cookie</th>
                <th>Path</th>
                <th>Secure</th>
                <th class="plus">HttpOnly</th>
                <th class="plus">SameSite</th>
            </tr>
            <tr>
                <td>
                    <UrlInput placeholder="not set" @update="(x) => domain = x" :requirements="{optional: true}"/>
                </td>
                <td>
                    <UrlInput placeholder="set origin domain" @update="(x) => origin = x"/>
                </td>
                <td>
                    <input type="text" v-model.trim="path" placeholder="/">
                </td>
                <td>
                    <input type="checkbox" v-model="secure" />
                </td>
                <td class="plus">
                    <input type="checkbox" v-model="http" />
                </td>
                <td class="plus">
                    <select v-model="sameSite">
                        <option v-for="(type,id) in types" :key="id" :disabled="type === 'None' && !secure">{{type}}</option>
                    </select>
                </td>
            </tr>
        </table>        
    </div>
    <div class="centered">
        <table>
            <tr>
                <th>Request sent from</th>
                <th>Request sent to</th>
                <th></th>
            </tr>
            <tr>
                <td>
                    <UrlInput placeholder="set source" @update="(x) => source = x"/>
                </td>
                <td>
                    <UrlInput placeholder="set destination" @update="(x) => destination = x" :requirements="{protocol: true}"/>
                </td>
                <td>
                    <span :class="{send : send, keep : !send}">{{send ? 'sent' : 'not sent' }}</span>
                </td>
            </tr>
        </table>
    </div>
</template>

<script>
import UrlInput from "./UrlInput.vue";
export default {
    name: "CookieTool",
    components: { UrlInput },
    data() {
        return {
            domain: {valid: true},
            origin: {},
            path: "",
            secure: false,
            http: false,
            sameSite: "",
            types: ["Lax", "Strict", "None"],
            source: {},
            destination: {}
        };
    },
    computed: {
        send() {
            if (!this.domain.valid || !this.origin.valid || !this.source.valid || !this.destination.valid) return false;
            let send = true;
            if (this.domain.data?.domain) {
                send &&= this.destination.data.domain.endsWith(this.domain.data.domain);//TODO check domain-suffix
            }
            else {
                send &&= this.destination.data.domain === this.origin.data.domain;
            }
            send &&= this.destination.data.path.startsWith(this.path);//TODO check whole path
            send &&= this.secure ? this.destination.data.protocol === 'https://' : true;
            send &&= this.sameSite === 'Strict' ? !this.isCrossSite : true;
            return send;
        },
        isCrossSite(){
            return this.extractTLD(this.source.data.domain) !== this.extractTLD(this.destination.data.domain);
        }
    },
    methods: {
        extractTLD(str) {
            const regex = /[a-z]+\.[a-z]+$/;
            return str.match(regex)[0];
        }
    },
    mounted() {
        this.sameSite = this.types[0];
    }    
}
</script>

<style scoped>
.centered {
    display: grid;
    justify-content: center;
    margin-top: 2em;
}

td{
    padding: 1em;
}

.plus{
    padding-left: 2em;
}

.send{
    padding: 5px;
    font-weight: bold;
    border: solid seagreen 5px;
}

.keep{
    padding: 5px;
    font-weight: bold;
    border: solid red 2px;
}
</style>
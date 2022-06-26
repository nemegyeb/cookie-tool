<template>
    <tr>
        <td>
            <input type="text" placeholder="set cookie name" v-model="name">
        </td>
        <td>
            <url-input placeholder="not set" v-model="domain" :requirements="{optional: true}"/>
        </td>
        <td>
            <url-input placeholder="set origin domain" v-model="origin"/>
        </td>
        <td>
            <input type="text" v-model.trim="path" placeholder="/" />
        </td>
        <td>
            <input type="checkbox" v-model="secure" />
        </td>
        <td>
            <input type="checkbox" v-model="http" />
        </td>
        <td>
            <select v-model="sameSite">
                <option v-for="(type,id) in types" :key="id" :disabled="type === 'None' && !secure">{{type}}</option>
            </select>
        </td>
        <td :class="{yes: send, no: !send}">
            {{send ? 'Yes' : 'No'}}
        </td>
        <td>
            <button type="button" @click="remove">X</button>
        </td>
    </tr>
</template>

<script>
import UrlInput from "./UrlInput.vue";
export default {
    name: "CookieAttributeSetter",
    props: {
        source: Object,
        destination: Object
    },
    components: { 
        UrlInput 
    },
    data() {
        return {
            name: '',
            domain: {valid: true},
            origin: {},
            path: '',
            secure: false,
            http: false,
            sameSite: '',
            types: ['Lax', 'Strict', 'None'],
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
        },
        remove(){
            this.name = '';
            this.domain = {reset: true};
            this.origin = {reset: true};
            this.path = "";
            this.secure = false;
            this.http = false;
            this.sameSite = 'Lax';
            this.$emit('remove');
        }
    },
    mounted() {
        this.sameSite = this.types[0];
    }
}
</script>

<style scoped>
.yes {
    font-weight: bold;
    color: green;
}

.no{
    font-weight: bold;
    color: red;
}

td{
    padding: 1em;
}
</style>

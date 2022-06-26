<template>
    <div class="centered">
        <table>
            <tr>
                <th>Request sent from</th>
                <th>Request sent to</th>
            </tr>
            <tr>
                <td>
                    <UrlInput placeholder="set source" v-model="source"/>
                </td>
                <td>
                    <UrlInput placeholder="set destination" v-model="destination" :requirements="{protocol: true}"/>
                </td>
            </tr>
        </table>
    </div>
    <div class="centered info" v-if="reset">
        Removed last row, added a new one.
    </div>
    <div class="centered">
        <table>
            <tr>
                <th>Name</th>
                <th>Domain attribute</th>
                <th>Domain who set the cookie</th>
                <th>Path</th>
                <th>Secure</th>
                <th>HttpOnly</th>
                <th>SameSite</th>
                <th>Attached</th>
                <th></th>
            </tr>
            <cookie-attribute-setter v-for="cookie in cookies" :key="cookie" :source="source" :destination="destination" @remove="removeConfig(cookie)" />
        </table>
    </div>
    <button type="button" @click="addConfig">+</button>
</template>

<script>
import UrlInput from "./UrlInput.vue";
import CookieAttributeSetter from "./CookieAttributeSetter.vue";
export default {
    name: "CookieTool",
    components: { 
        UrlInput,
        CookieAttributeSetter 
    },
    data() {
        return {
            source: {},
            destination: {},
            cookies: [0],
            reset: false
        };
    },
    methods: {
        addConfig(){
            this.cookies.push(this.cookies[this.cookies.length -1] + 1);
        },
        removeConfig(id){
            if(this.reset) return;            
            if(this.cookies.length === 1){
                this.cookies = [0];
                this.reset = true;
                setTimeout(() => {
                    this.reset = false;
                }, 3000);
            } else {
                this.cookies = this.cookies.filter((cookie) => cookie !== id);
            }            
        }
    }
}
</script>

<style scoped>
.centered {
    display: grid;
    justify-content: center;
    margin-top: 2em;
}

td, th {
    padding: 1em;
}

.info {
    color: #00529B;
    background-color: #BDE5F8;
    padding: 1em 2em;
    margin: 10px 20px;
    font-size: 1.25em;    
    display:inline-block;
    vertical-align:middle;
}
</style>
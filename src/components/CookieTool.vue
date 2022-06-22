<template>
    <div>
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
                    <input type="text" v-model.trim="domain" placeholder="not set">
                </td>
                <td>
                    <input type="text" v-model.trim="origin" placeholder="set origin domain" :class="{'error': !validOrigin}">
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
                        <option v-for="(type,id) in types" :key="id">{{type}}</option>
                    </select>
                </td>
            </tr>
        </table>        
    </div>
    <div>
        <table>
            <tr>
                <th>Request sent from</th>
                <th>Request sent to</th>
                <th></th>
            </tr>
            <tr>
                <td>
                    <input type="text" v-model.trim="source" placeholder="set source" :class="{'error': !validSource}">
                </td>
                <td>
                    <input type="text" v-model.trim="destination" placeholder="set destination" :class="{'error': !validDestination}">
                </td>
                <td>
                    <span :class="{'send' : sent}">{{toSend}}</span>
                </td>
            </tr>
        </table>
    </div>
</template>

<script>
export default {
    data(){
        return {
            domain: "",
            origin: "",
            path: "",
            secure: false,
            http: false,
            sameSite: null,
            types: ["Lax", "Strict", "None"],
            source: "",
            destination: ""
        }
    },
    computed: {
        sent(){
            if(!this.validOrigin || !this.validDestination || !this.validSource || !this.validDestination) return false;
            const pOrigin = this.parseSet(this.origin);
            const pDest = this.parseReq(this.destination);
            const pSource = this.parseSet(this.source);
            let send = true;

            if(this.domain){
                const pDomain = this.parseSet(this.domain);
                send &= this.isSuffix(pDest.domain, pDomain.domain);
            } else{
                send &= pOrigin.domain === pDest.domain;
            }

            send &= pDest.path.startsWith(this.path);

            send &= this.secure ? pDest.protocol === "https" : true;

            switch(this.sameSite){
                case "Lax":                    
                    break;
                case "Strict":
                    send &= !this.isCrossSite(pSource.domain, pDest.domain);
                    break;
                case "None":
                    send &= !this.secure;
                    break;
            }

            return send;
        },
        toSend(){
            return this.sent ? "sent" : "not sent";
        },
        validOrigin(){
            return this.origin && this.validateSet(this.parseSet(this.origin));
        },
        validSource(){
            return this.source && this.validateSet(this.parseSet(this.source));
        },
        validDestination(){
            return this.destination && this.validateReq(this.parseReq(this.destination));
        }
    },
    methods: {
        parseReq(str){
            const prot = str.indexOf('://');
            let dom = str.indexOf('/', prot + 3);
            dom = dom == -1 ? str.length : dom
            return {
                protocol: prot == -1 ? "" : str.substring(0,prot),
                domain: str.substring(prot + 3, dom),
                path: str.substring(dom) || "/"
            }
        },
        parseSet(str){
            let start;
            if(str.startsWith('http://')){
                start = 'http://'.length;
            } else if(str.startsWith('https://')){
                start = 'https://'.length;
            }
            let dom = str.indexOf('/', start);
            dom = dom == -1 ? str.length : dom
            return {
                domain: str.substring(start, dom),
                path: str.substring(dom) || "/"
            }
        },
        validateReq(url){
            return url.protocol && validateSet(url.domain);
        },
        validateSet(url){
            const regex = /.+\.[a-z]{1,3}/;
            return url.domain.match(regex) != null;
        },
        isSuffix(str, suffix){
            return str.indexOf(suffix, str.length - suffix.length) !== -1;
        },
        extractTLD(str){
            const regex = /[^\.]*\..{1,3}$/;
            return (str.match(regex) || [""])[0];
        },
        isCrossSite(source, dest){
            return this.extractTLD(source) !== this.extractTLD(dest);
        }
    },
    mounted(){
        this.sameSite = this.types[0];
    }
}
</script>

<style scoped>
div{
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

.error{
    border: solid orangered 2px;
}
</style>
<template>
    <div>
        <input type="text" v-model.trim="input" :placeholder="placeholder" :class="{ error: !valid }" />
    </div>
</template>

<script>
export default {
    name: 'UrlInput',
    props: {
        placeholder: String,
        requirements: Object
    },
    emits: ['update'],
    data() {
        return {
            text: ""
        }
    },
    computed: {
        data() {
            const regex = /(https?:\/\/)?((?:[a-z]+\.)+[a-z]+)((?:\/[a-z]+)*\/?)/;
            const matches = this.text.match(regex);
            const valid = matches !== null && matches[0] === this.text;
            return {
                valid,
                data: !valid ? undefined : {
                    protocol: matches[1],
                    domain: matches[2],
                    path: matches[3] || '/'
                }
            };
        },
        url() {
            return {
                valid: this.valid,
                data: !this.valid ? undefined : this.data.data
            };
        },
        valid() {
            let valid = this.data.valid;
            if (this.requirements) {
                if(this.requirements.optional && this.text === '') return true;
                valid &&= this.requirements.protocol ? this.data.data.protocol != null : true;
                valid &&= this.requirements.domain ? this.data.data.domain != null : true;
                valid &&= this.requirements.path ? this.data.data.path != null : true;
            }
            return valid;
        },
        input: {
            get(){
                return this.text;
            },
            set(val){
                this.text = val;
                this.$emit('update', this.url);
            }
        }
    }
}
</script>

<style scoped>
.error {
    border: solid orangered 2px;
}
</style>

<template>
    <step>
        <template slot="header">
            <div class="btn-left"><a @click="$emit('back')">{{ "Back"|t('app') }}</a></div>
            <h1>Identity</h1>
        </template>
        <template slot="main">
            <form @submit.prevent="save()">
                <p><label><input type="radio" value="craftid" v-model="identityMode" /> {{ "Use your Craft ID"|t('app') }}</label></p>

                <template v-if="identityMode === 'craftid'">
                    <p>
                        <btn kind="primary" @click="connectCraftId">{{ "Connect to your Craft ID"|t('app') }}</btn>
                    </p>
                </template>

                <p><label><input type="radio" value="guest" v-model="identityMode" /> {{ "Continue as guest"|t('app') }}</label></p>

                <template v-if="identityMode === 'guest'">
                    <textbox id="email" placeholder="Email" v-model="guestEmail" :errors="guestEmailError" />
                    <btn kind="primary" type="submit" :disabled="(!validates || loading)" :loading="loading">{{ "Continue"|t('app') }}</btn>
                </template>
            </form>
        </template>
    </step>
</template>

<script>
    /* global Craft */

    import {mapState} from 'vuex'
    import Step from '../Step'

    export default {
        components: {
            Step,
        },

        data() {
            return {
                guestEmail: '',
                guestEmailError: false,
                loading: false,
            }
        },

        computed: {
            ...mapState({
                cart: state => state.cart.cart,
                craftId: state => state.craft.craftId,
            }),

            identityMode: {
                get() {
                    return this.$store.state.cart.identityMode
                },

                set(value) {
                    this.$store.commit('cart/changeIdentityMode', value)
                }
            },

            validates() {
                if (this.identityMode === 'guest' && !this.guestEmail) {
                    return false
                }

                return true
            }
        },

        methods: {
            connectCraftId() {
                let width = 800
                let height = 600

                let winWidth = window.innerWidth ? window.innerWidth : document.documentElement.clientWidth ? document.documentElement.clientWidth : screen.width
                let winHeight = window.innerHeight ? window.innerHeight : document.documentElement.clientHeight ? document.documentElement.clientHeight : screen.height

                let left = ((winWidth / 2) - (width / 2))
                let top = ((winHeight / 2) - (height / 2))

                let url = Craft.getActionUrl('plugin-store/connect', {redirectUrl: Craft.getActionUrl('plugin-store/modal-callback')})
                let name = 'ConnectWithOauth'
                let specs = 'location=0,status=0,width=' + width + ',height=' + height + ',left=' + left + ',top=' + top

                window.open(url, name, specs)
            },

            save() {
                this.loading = true

                if (this.identityMode === 'guest') {
                    let data = {
                        email: this.guestEmail,
                    }

                    this.$store.dispatch('cart/saveCart', data)
                        .then(() => {
                            this.loading = false
                            this.$root.openModal('payment')
                        })
                        .catch((error) => {
                            this.loading = false
                            this.$root.displayError("Couldn???t save identity.")
                            throw error
                        })
                } else {
                    this.loading = false
                    this.$root.openModal('payment')
                }
            }
        },

        mounted() {
            this.$root.$on('craftIdUpdated', () => {
                if (this.craftId) {
                    this.$root.openModal('payment')
                }
            });

            this.guestEmail = this.cart.email
        }
    }
</script>

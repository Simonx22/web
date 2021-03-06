<template>
    <div v-if="room" class="room-wrapper">
        <div class="room">
            <Player :loaded-scripts="loadedScripts" />
            <PlayerFooter />
            <Chat />
        </div>
        <Loading v-if="!wsHeartbeat" />
    </div>
</template>
<script>
    import { mapGetters } from 'vuex'

    import Chat from '~/components/Chat'
    import Player from '~/components/Player'
    import PlayerFooter from '~/components/Player/Footer'
    import Loading from '~/components/Loading'

    export default {
        middleware: 'authenticated',
        components: {
            Chat,
            Player,
            PlayerFooter,
            Loading
        },
        async asyncData(context) {
            await context.store.dispatch('fetchRoom')
        },
        data() {
            return {
                loadedScripts: []
            }
        },
        computed: {
            ...mapGetters(['room', 'wsHeartbeat'])
        },
        mounted() {
            this.$store.commit('setupWebSocket')

            this.unsubscribe = this.$store.subscribe(({ type }, { stream }) => {
                if (type === 'setupWebSocket')
                    this.$store.dispatch('fetchRoom')
                else if (type === 'handleRoom')
                    if (!this.room)
                        this.$router.push('/home')
            })
        },
        beforeDestroy() {
            this.unsubscribe()
            this.$store.commit('disconnectWebSocket')
        },
        head() {
            // ToDo: rework JSMpeg/Janus imports
            return {
                title: this.room ? this.room.name : 'Room Not Found',
                link: process.env.ENABLE_JANUS ? [
                    { rel: 'preload', href: '/assets/js/janus.min.js?v=1', as: 'script'}
                ] : [
                    { rel: 'preload', href: '/assets/js/jsmpeg.min.js?v=1', as: 'script'}
                ],
                script: process.env.ENABLE_JANUS ? [
                    { charset: 'utf-8', src: '/assets/js/janus.min.js?v=1', callback: () => this.loadedScripts.push('janus')}
                ] : [
                    { charset: 'utf-8', src: '/assets/js/jsmpeg.min.js?v=1', callback: () => this.loadedScripts.push('jsmpeg')}
                ]
            }
        }
    }
</script>

<template>
    <v-slide-y-reverse-transition>
        <div v-if="state.active" class="pa-2 tw-absolute tw-z-50">
                <v-alert closable density="compact" variant="flat" :type="state.alert.type" position="fixed" location="bottom right" class="ma-2">
                    <div class="text-body-1">{{ state.alert.title }}</div>
                </v-alert>
        </div>
    </v-slide-y-reverse-transition>
</template>

<script>
export default {

    computed: {
        state() {
            return this.$store.getters['alert/alert']
        }
    },
}
</script>

<style>

</style>
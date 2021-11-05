<template>
    <div class="container">
        <div><span v-show="player.statPoints > 0" @click="$emit('upgradeStat', 'hp')">+</span><strong>HP</strong>: {{ player.hp }} / {{ player.maxHP }}</div>
        <div><strong>Сытость</strong>: {{ player.hunger }}</div>
        <div><span v-show="player.statPoints > 0" @click="$emit('upgradeStat', 'strength')">+</span><strong>DMG</strong>: {{ player.strength + ((weapon) ? weapon.damage : 0) }}</div>
        <div><span v-show="player.statPoints > 0" @click="$emit('upgradeStat', 'defense')">+</span><strong>DEF</strong>: {{ player.defense + ((armor) ? armor.protection : 0) }}</div>
    </div>
</template>

<script>
export default {
    props: {
        armor: {
            type: Object
        },
        player: {
            type: Object,
            required: true
        },
        weapon: {
            type: Object
        }
    }
}
</script>

<style scoped>
.container {
    width: 40%;
    height: 70%;
    padding: 5% 1.5%;
    margin: 0 3.5%;
    border: 2px solid hsla(150, 60%, 30%, 0.9);
    display: inline-block;
}
</style>

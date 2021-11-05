<template>
    <div class="container">
        <button 
            class="btn" 
            @click="$emit('explore')"
            >Исследовать</button>
        <button 
            class="btn"
            @click="$emit('eat')"
            >Перекусить</button>
        <button 
            class="btn"
            @click="$emit('potion')"
            >Выпить Зелье</button>
    </div>
</template>

<script>
</script>

<style scoped>
.btn {
    width: 55%;
    display: block;
    height: 25%;
    background: none;

    color: hsla(150, 60%, 30%, 0.9);
    font-size: 16px;
    font-family: Verdana, Geneva, Tahoma, sans-serif;

    border: 1px solid hsla(150, 60%, 30%, 0.9);
    margin: 4% auto;
}

.container {
    width: 40%;
    height: 70%;
    padding: 2% 1.5%;
    margin: 0 3.5%;
    border: 2px solid hsla(150, 60%, 30%, 0.9);
    display: inline-block;
}
</style>

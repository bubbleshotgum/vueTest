<template>
<div class="top-level">
    <player-info
        :player="player"
        :armor="(player.inventory.armor > -1) ? armor[player.inventory.armor] : null"
        :weapon="(player.inventory.weapon > -1) ? weapons[player.inventory.weapon] : null"
        @upgrade-stat="upgradeStat"
    />
    <control-panel
        @explore="explore"
        @eat="openFood"
        @potion="openPotions"
    />
</div>
<div class="bottom-level">
    <div v-if="status === 'idle'">
        <h4>Вы сидите на большом валуне, свесив ноги и думая, что бы сейчас сделать…</h4>
    </div>
    <div v-else-if="status === 'exploring'">
        <h4>Исследование начинается! (-3 сытости)</h4>
        <p>{{ explorationMessage.content }}</p>
    </div>
    <div v-else-if="status === 'looting'">
        <h4>Ура! Сундук с сокровищами!</h4>
        <button @click="openTreasureChest">Открыть</button>
    </div>
    <div v-else-if="status === 'trapped'">
        <h4>Вы угодили в ловушку… Потеряна часть здоровья</h4>
        <button @click="getFree">Освободиться</button>
    </div>
    <div v-else-if="status === 'eating'">
        <div v-for="item in player.inventory.food" :key="item">
            <p>{{ food[item].name }} — {{ food[item].saturation }} сытости</p>
            <button @click="eat(food[item])">Съесть</button>
        </div>
        <button @click="status = 'idle'">Назад</button>
    </div>
    <div v-else-if="status === 'potion'">
        <div v-for="item in player.inventory.potions" :key="item">
            <p>{{ potions[item].name }} — {{ potions[item].heal }} здоровья</p>
            <button @click="usePotion(potions[item])">Выпить</button>
        </div>
        <button @click="status = 'idle'">Назад</button>
    </div>
    <div v-show="status === 'fighting'">
        <h4>Путь перегородила злобная тварь!</h4>
        <div v-if="!monster">Вы пытаетесь опознать противника…</div>
        <div class="enemy" v-else-if="monster">
            <h4>Противник</h4>
            <div>Здоровье: <span>{{ monster.hp }}</span> / <span>{{ monster.maxHP }}</span></div>
            <div>Атака: <span>{{ monster.dmg }}</span></div> 
            <div>Броня: <span>{{ monster.def }}</span></div>
        </div>
        <div v-show="monster">
            <p ref="fightLog">Ваш ход:</p>
            <button @click="hit">Удар</button>
            <button @click="flee">Сбежать</button>
        </div>
    </div>
</div>
<div 
    class="lossScreen"
    v-if='player.hp <= 0'>
    <div>
        <h1>Вы умерли</h1>
        <button onclick="window.location.reload()">Обрести Покой</button>
    </div>
</div>
</template>

<script>
import ControlPanel from '@/components/ControlPanel'
import PlayerInfo from '@/components/PlayerInfo'
export default {
    components: {
        ControlPanel,
        PlayerInfo
    },
    data: () => ({
        armor: [
            {id: 1, name: 'Кожаная Туника', protection: 1},
            {id: 2, name: 'Ржавые Латы', protection: 2},
            {id: 3, name: 'Легкая Кольчуга', protection: 4},
            {id: 4, name: 'Стальные Латы', protection: 5}
        ],
        event: null,
        events: [
            {id: 1, type: 'fighting', weight: 3},
            {id: 2, type: 'looting', weight: 1},
            {id: 3, type: 'trapped', weight: 2}
        ],
        explorationMessage: '',
        food: [
            {id: 1, name: "Черствый Хлеб", saturation: 1},
            {id: 2, name: "Вяленая Рыба", saturation: 2},
            {id: 3, name: "Яйцо", saturation: 3},
            {id: 4, name: "Бекон", saturation: 6},
            {id: 5, name: "Чечевичная Похлебка", saturation: 15},
            {id: 6, name: "Хачапури по-гурийски", saturation: 40}
        ],
        messages: [
            {id: 1, content: 'Проходя по опушке, Вы вспоминаете, как отец учил Вас выживанию в лесу. Жаль, что воспоминания обрываются на том моменте, когда Вы услышали чей-то вой…'},
            {id: 2, content: 'Вы блуждаете в пещерных лабиринтах, пытаясь найти выход по источнику ветра. В одном из коридоров при свете факела виднеется обглоданный детский скелет.'},
            {id: 3, content: 'Вам на пути повстречался заброшенный особняк. В голову закрадываются мысли о драгоценностях, которые там могут находиться. Поддавшись соблазну, Вы заходите внутрь, но, увы, всё здесь было разорено ещё задолго до Вашего визита.'},
            {id: 4, content: 'Жизнь в городе — настолько скучное и неблагодарное занятие, что Вы просто не могли позволить себе заниматься такой ерундой. Причина Вашего отшельничества кроется явно в этом, а не в том, что в городе нужно работать.'},
            {id: 5, content: 'Вы идёте вниз по реке и видите следующую картину: какая-то девочка следует за белым кроликом. С моноклем. В костюме пажа. Каждый миг то и дело посматривающим на часы…'},
            {id: 6, content: 'Во время привала Вы замечаете, как вдалеке яростно сражаются воин в белом халате и деревянных сандалиях и чёрное зловещее нечто. Доблестный воин собирался было поставить точку в этом поединке с помощью своего меча, однако противник в последний момент сумел отправить его в будущее и тем самым избежать гибели.'},
            {id: 7, content: 'Проходя по опушке, Вы вспоминаете, как отец учил Вас выживанию в лесу. Внезапно голова начала раскалываться; Вы слышите детские крики, а вой всё усиливается. Что же произошло в тот день?'},
            {id: 8, content: 'Проходя по опушке, Вы вспоминаете, как отец учил Вас выживанию в лесу. В голове начали складываться чёткие образы: за деревьями показался гигантский черношерстный волк. Отец кричит Вам бежать на всех парах.'},
            {id: 9, content: 'Вы блуждаете в пещерных лабиринтах, пытаясь найти выход по источнику ветра. От сильного дуновения ветра факел погас. Вы слышите рёв зверя за спиной…'}
        ],
        monster: null,
        player: {
            defense: 1,
            hp: 12,
            hunger: 100,
            inventory: {
                armor: -1,
                food: [],
                potions: [],
                weapon: -1
            },
            maxHP: 12,
            statPoints: 0,
            strength: 1
        },
        potions: [
            {id: 1, name: "Малое зелье здоровья", heal: 1},
            {id: 2, name: "Слабое зелье здоровья", heal: 3},
            {id: 3, name: "Большое зелье здоровья", heal: 7},
            {id: 4, name: "Сильное зелье здоровья", heal: 12}
        ],
        status: 'idle',
        weapons: [
            {id: 1, name: 'Дубина', damage: 1},
            {id: 2, name: 'Ржавый Меч', damage: 2},
            {id: 3, name: 'Кинжал Разбойника', damage: 4},
            {id: 4, name: 'Рыцарский Меч', damage: 5}
        ]
    }),
    methods: {
        eat(item)
        {
            this.player.hunger = (this.player.hunger + item.saturation > 100) ? 100 : this.player.hunger + item.saturation
            this.player.inventory.food.splice(this.player.inventory.food.indexOf(item.id), 1)
            this.status = 'idle'
        },
        explore()
        {
            if(this.status === 'idle')
            {
                this.status = 'exploring'
                if(this.player.hunger > 10) 
                    this.player.hunger -= 3
                this.explorationMessage = this.messages[Math.floor(Math.random() * (this.messages.length - 1))]
                setTimeout(() => {
                    let _events = [],
                        i = 0
                    for(; i < this.events.length; i++)
                    {
                        let _event = {
                            id: this.events[i].id,
                            type: this.events[i].type,
                            weight: this.events[i].weight
                        }
                        _events.push(_event)
                    }
                    for(i = 1; i < _events.length; i++)
                        _events[i].weight += _events[i - 1].weight
                    _events[i - 1].weight--
                    let random = Math.random() * _events[i - 1].weight
                    for(i = 0; i < _events.length; i++)
                        if(_events[i].weight >= random)
                            break
                    this.event = _events[i]
                    if(this.event.type === 'fighting')
                    {
                        setTimeout(() => {
                            this.monster = {
                                maxHP: Math.floor(Math.max(12, (Math.random() + 2) * this.player.maxHP + 7 * (Math.round(Math.random()) * 2 - 1))),
                                def: Math.floor(Math.max(0, (Math.random() + .5) * (this.player.defense + ((this.player.inventory.armor > -1) ? this.armor[this.player.inventory.armor].protection : 0)) + 2 * (Math.round(Math.random()) * 2 - 1))),
                                dmg: Math.floor(Math.max(2, (Math.random() + .5) * (this.player.strength + ((this.player.inventory.weapon > -1) ? this.weapons[this.player.inventory.armor].damage : 0)) + (Math.round(Math.random()) * 2 - 1)))
                            }
                            this.monster.hp = this.monster.maxHP
                        }, 3000)
                    }
                    this.explorationMessage = ''
                    this.status = this.event.type
                }, 10 * 100 * 1000 / this.player.hunger)
            }
            else
                alert('Вы заняты и не можете исследовать локацию')
        },
        flee()
        {
            let chance = Math.random()
            this.player.hp -= chance < 0.5 ? Math.max(0, this.monster.dmg - (this.player.defense + ((this.player.inventory.armor > -1) ? this.armor[this.player.inventory.armor].protection : 0))) : 0
            this.$refs.fightLog.textContent = 
                chance < 0.5 ? 
                "При попытке сбежать Вы получили урон в сумме " 
                + Math.max(0, this.monster.dmg - (this.player.defense + ((this.player.inventory.armor > -1) ? this.armor[this.player.inventory.armor].protection : 0)))
                + " ед. здоровья" 
                : "Вы успешно сбежали, не получив удара в спину!"
            setTimeout(() => {
                this.monster = null
                this.event = null
                this.$refs.fightLog.textContent = ''
                this.status = 'idle'
            }, 3000)
        },
        getFree()
        {
            this.player.hp -= Math.floor(this.player.maxHP / 3)
            this.event = null
            this.status = 'idle'
        },
        hit()
        {
            let chance = Math.random()
            this.monster.hp -= chance > 0.5 ?
                Math.max(0,
                    (this.player.strength 
                    + ((this.player.inventory.weapon > -1) ? this.weapons[this.player.inventory.weapon].damage : 0))
                    - this.monster.def)
                : 0
            this.$refs.fightLog.textContent = 
                chance > 0.5 ? 
                "Вы нанесли " 
                + Math.max(0, this.player.strength + ((this.player.inventory.weapon > -1) ? this.weapons[this.player.inventory.weapon].damage : 0) - this.monster.def)
                + " ед. урона противнику" 
                : "Вы промахнулись!"
            
            if(this.monster.hp <= 0)
            {
                this.monster = null
                this.event = null
                this.player.statPoints++
                this.$refs.fightLog.textContent = ''
                this.status = 'idle'
            }
            else
            {
                chance = Math.random()
                this.player.hp -= chance < 0.5 ? Math.max(0, this.monster.dmg - (this.player.defense + ((this.player.inventory.armor > -1) ? this.armor[this.player.inventory.armor].protection : 0))) : 0
                this.$refs.fightLog.textContent += 
                    chance < 0.5 ? 
                    " Противник нанёс " 
                    + Math.max(0, this.monster.dmg - (this.player.defense + ((this.player.inventory.armor > -1) ? this.armor[this.player.inventory.armor].protection : 0)))
                    + " ед. урона в ответ" 
                    : " Вы успешно увернулись!"
            }
        },
        openFood()
        {
            if(this.status === 'idle')
                this.status = 'eating'
            else
                alert("Вы сейчас слишком заняты, чтобы есть")
        },
        openPotions()
        {
            if(this.status === 'idle')
                this.status = 'potion'
            else
                alert("Вы сейчас слишком заняты для использования зелья")
        },
        openTreasureChest()
        {
            let randArmor = Math.floor(Math.random() * (this.armor.length - 1)),
                randWeapon = Math.floor(Math.random() * (this.weapons.length - 1))
            if(this.player.inventory.armor < randArmor)
                this.player.inventory.armor = randArmor
            if(this.player.inventory.weapon < randWeapon)
                this.player.inventory.weapon = randWeapon
            this.player.inventory.food.push(this.food[Math.floor(Math.random() * (this.food.length - 1))].id)
            this.player.inventory.potions.push(this.potions[Math.floor(Math.random() * (this.potions.length - 1))].id)
            this.event = null
            this.status = 'idle'
        },
        upgradeStat(stat)
        {
            if(this.player.statPoints == 0)
                return null
            switch(stat)
            {
                case 'hp':
                    this.player.maxHP++
                    break
                case 'strength':
                    this.player.strength++
                    break
                case 'defense':
                    this.player.defense++
                    break
                default:
                    console.warn('Unrecognized parameter…')
                    break
            }
            this.player.statPoints--
        },
        usePotion(item)
        {
            this.player.hp = (this.player.hp + item.heal > this.player.maxHP) ? this.player.maxHP : this.player.hp + item.heal
            this.player.inventory.potions.splice(this.player.inventory.potions.indexOf(item.id), 1)
            this.status = 'idle'
        }
    }
}
</script>

<style>
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}
#app {
    padding: 15px;
    width: 100vw;
    height: 90vh;
}
.top-level {
    width: 100%;
    height: 50%;
}
.lossScreen
{
    position: fixed;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    background: rgba(0, 0, 0, .8);
}
.lossScreen div 
{
    width: 40%;
    height: 30%;
    margin: auto;
    background: rgb(60, 60, 60);
}
</style>

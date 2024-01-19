<template>
    <main>
        <h1 class="title">Kalkulacka</h1>
        <div class="display">{{ (state.eq || 0) }}</div>
        <div class="grid">
            <div v-for="n in grid" @click="input(n)" class="button" :key="n">
                {{ n }}
            </div>
        </div>
    </main>
</template>

<script setup>
import { ref, reactive } from 'vue';

const grid = [
    '(', ')', 'π', '<-', 'C',
    '7', '8', '9', 'x²', 'xʸ',
    '4', '5', '6', ' * ',' / ',
    '1', '2', '3', ' + ', ' - ',
    '0', '.', 'EXP', 'Ans', '='
];
const ops = ['.', ' * ', ' / ', ' + ', ' - '];
/*
let num1 = ref(0);
let num2 = ref('bbb');
let op = ref('');*/
let state = reactive({ eq: '', result: '' });

function input(num) {
    let sl = state.eq.slice(-1);
    let isNumeric = (!isNaN(num) && num && num !== ' ');
    let isNumber = isNumeric || num === 'π' || num === ')' || num === '²';
    let isLastNumeric = (!isNaN(sl) && sl && sl !== ' ');
    let isLastNumber = isLastNumeric || sl === 'π' || sl === ')' || sl === '²';
    let isLastUnallowed = sl === ' ' || sl === '.' || sl === '(' || sl === '^' || sl === 'e';

    if(num === 'C') {
        state.eq = '';
        return;
    }
    else if (num === '<-') {
        state.eq = state.eq.slice(0, (state.eq.slice(-1) === ' ' ? -3 : -1));
        return;
    }
    else if(num === '=') {
        state.eq = state.eq.replaceAll('π', Math.PI.toString());
        state.eq = state.eq.replaceAll('²', '^2');
        let sp = state.eq.split(' ');
        for(let i=0; i < sp.length; i++) {
            if(sp[i].includes('^')) {
                let sp2 = sp[i].split('^');
                sp[i] = 'Math.pow(' + sp2[0] + ',' + sp2[1] + ')';
            }
        }
        state.eq = sp.join(' ');
        state.eq = eval(state.eq).toString();
        state.result = state.eq;
        if(state.eq === '0') state.eq = '';
        return;
    }
    else if(ops.includes(num)) {
        if(isLastUnallowed) return;
        if((num === '.' && state.eq === '')) state.eq = '0';
        let sp = state.eq.split(' ');
        sp = sp[sp.length-1];
        if((!sp || sp.includes(num))) return;
    }
    else if(num === 'x²' || num === 'xʸ') {
        let sp = state.eq.split(' ');
        sp = sp[sp.length-1];
        if(sp.includes('²') || sp.includes('^')) return;
        num = num === 'x²' ? '²' : '^';
    } else {
        switch(num) {
            case 'EXP':
                if(!isLastNumeric) return;
                num = 'e';
                break;
            case 'Ans':
                if(isLastNumber && state.result) num = ' * ' + state.result;
                else num = state.result;
                break;
            case 'π':
                if(isLastNumber) num = ' * π';
                break;
            case '(':
                if(isLastNumber) num = ' * (';
                break;
            case ')':
                if(sl === '(') return;
                let LParenN = (state.eq.match(/\(/g)||[]).length;
                let RParenN = (state.eq.match(/\)/g)||[]).length;
                if(RParenN >= LParenN) return;
                break;
            default: break;
        }
    }
    if(isLastNumber && !isLastNumeric && isNumeric) num = ' + ' + num;
    state.eq += num;
    
    /*if(op === '') num1 = num;
    else num2 = num;*/
}

</script>

<style scoped>
main {
    position: fixed;
    text-align: center;
    color: #ccc;
    padding: 0 30vw;
    //background-image: linear-gradient(to top, #a18cd1 0%, #fbc2eb 100%);
    background: #03001e; /* fallback for old browsers */
    background: -webkit-linear-gradient(45deg, #03001e, #7303c0, #ec38bc, #fdeff9); /* Chrome 10-25, Safari 5.1-6 */
    background: linear-gradient(45deg, #03001e, #7303c0, #ec38bc, #fdeff9); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
    height: 100%;
    width: 100%;
}

.title {
    font-size: 56px;
    filter: drop-shadow(0px 3px 2px rgba(0, 0, 0, .5));
}

.display {
    background-color: #171717;
    font-size: 32px;
    filter: drop-shadow(0px 3px 2px rgba(0, 0, 0, 1));
    padding: 20px 0;
    margin-top: 10vh;
    margin-bottom: 20px;
    border-radius: 24px;
    user-select: none;
}

.grid {
    display: grid;
    gap: 10px;
    grid-template-columns: repeat(5, 1fr);
}

.button {
    user-select: none;
    cursor: pointer;
    font-size: 32px;
    filter: drop-shadow(0px 3px 2px rgba(0, 0, 0, 1));
    padding: 30px;
    background-color: #222;
    border-radius: 10px;
} .button:hover {
    background-color: #171717;
} .button:active {
    background-color: #000;
}

</style>

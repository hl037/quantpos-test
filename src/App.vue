
<style scoped>
.controller{
  position:fixed;
  top: 0px;
  right: 0px;
  width: 50vw;
}

#content{
  position:relative;
}

.content div{
  width: 20px;
  height: 20px;
}

#a {
  background-color: #FF0000AA;
}

#b {
  background-color: #00FF00AA;
}

#c {
  background-color: #0000FFAA;
}

#ball {
  position:absolute;
  top: 0px;
  left:0px;
}

#ball>div {
  width: 10px;
  height: 10px;
  border-radius: 5px;
  background-color: #940071;
  position: absolute;
  top: -5px;
  left: -5px;
}

#areaA {
  width: 100px;
  height: 100px;
  background:#4efff9 ;
}

#areaB {
  width: 100px;
  height: 100px;
  background: #19c4de ;
}




</style>

<template>
<div>
  <div id="content" class="content">
    <div id="a" :ref="(e) => {elements[0].value=e;}">
      a
      <div id="b" :ref="(e) => { elements[1].value=e; }">
        b
        <div id="c" :ref="(e) => { elements[2].value=e; }">
          c
        </div>
      </div>
    </div>
    <div id="a2" :ref="(e) => {elements[3].value=e;}">
      a2
      <div id="b2" :ref="(e) => {elements[4].value=e;}">
        b2
        <div id="c2" :ref="(e) => {elements[5].value=e;}">
          c2
        </div>
      </div>
    </div>
    <div id="areaA" :ref="(e) => {elements[6].value=e;}">
      <div id="areaB" :ref="(e) => {elements[7].value=e;}" @click="placeball">
      </div>
    </div>
    <div id="ballContainer" :ref="(e) => {elements[8].value=e;}">
      <div id="ball" :ref="(e) => {elements[9].value=e;}">
        <div></div>
      </div>
    </div>

  </div>
  <div class="controller">
    <codemirror
      v-model="proxies.style"
      placeholder=""
      :style="{ height: '400px' }"
      :autofocus="true"
      :indent-with-tab="true"
      :tabSize="2"
      :extensions="extensions"
      mode="css"
      @change="update"
    />

    <button type="button" @mousedown="toggle">Toggle</button>

    <table>
      <tr>
        <th>el</th>
        <th v-for="v, k in watched">{{k}}</th>
      </tr>
      <tr v-for="e, i in elements">
        <th>{{e?.value?.id}}</th>
        <td v-for="v in values">{{v[i]}}</td>
      </tr>
    </table>

  </div>
</div>
  
  
</template>

<script setup>
/* global defineProps, defineEmits, defineExpose */

import { ref, onMounted, watch, toRefs, computed } from 'vue'
import { Codemirror } from 'vue-codemirror'
import { css as CMcss} from '@codemirror/lang-css'
import { vim } from "@replit/codemirror-vim"
import QP from 'quantpos'
const code = ref("")
const extensions = [ vim(), CMcss() ]

const style = document.createElement('style')
document.head.appendChild(style)

const proxies = {
  get style(){
    return style.innerHTML;
  },
  set style(v){
    style.innerHTML = v;
    localStorage.setItem('style', v)
  }
}

proxies.style = localStorage.getItem('style') || `
#a{
  position: relative;
}

#b{
  position: absolute;
}

#c{
  position: relative;
}

#a2{
  position: relative;
}

#b2{
  position: absolute;
}

#c2{
  position: relative;
}

`

const watched = {
  op(e){
    return e?.offsetParent.id;
  },
  ot(e){
    return e?.offsetTop;
  },
  ol(e){
    return e?.offsetLeft;
  },
  ow(e){
    return e?.offsetTop;
  },
  oh(e){
    return e?.offsetHeight;
  },
  tro(e){
    return e?.transformOrigin;
  }
}

const elements = [ref(), ref(), ref(), ref(), ref(), ref(), ref(), ref(), ref(), ref()]

const values = {}
Object.keys(watched).forEach( (k) => {
  values[k] = elements.map(() => {
    return ref()
  })
})

function update() {
  for(const k in watched){
    for(let i=0 ; i < elements.length ; ++i){
      values[k][i].value = watched[k](elements[i].value);
    }
  }
}

update()
onMounted(update)
onMounted( () => {
  window.a = elements[0].value
  window.b = elements[1].value
  window.c = elements[2].value
  window.sa = () => getComputedStyle(a)
  window.sb = () => getComputedStyle(b)
  window.sc = () => getComputedStyle(c)
  window.a2 = elements[3].value
  window.b2 = elements[4].value
  window.c2 = elements[5].value
  window.sa2 = () => getComputedStyle(a2)
  window.sb2 = () => getComputedStyle(b2)
  window.sc2 = () => getComputedStyle(c2)
  window.ball = elements[9].value
  window.sball = () => getComputedStyle(ball)
})

let match = false
function toggle(){
  if(match) {
    QP.setMat(c2, null)
    match = false
  }
  else {
    QP.setGlobalMat(c2, QP.getGlobalMat(c))
    match = true
  }
}

function placeball(ev){
  const pos = [ev.offsetX, ev.offsetY]
  const ball = elements[9].value
  let src
  let dst
  let trans = new QP.CoordTranslator(
    src=QP.getGlobalMat(ev.target),
    dst=QP.getGlobalMat(ball, true)
  )
  const ntrans = QP.translate(trans.fromAtoB(pos))
  console.log({
    'ev.target' : ev.target,
    'pos' : pos,
    'ntrans' : ntrans,
    'src' : src,
    'dst' : dst,
  });
  QP.setMat(ball, ntrans)
}




</script>


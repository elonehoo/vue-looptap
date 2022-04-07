
<script setup lang="ts">
  const arc =ref([180, 270])
  const taps = ref(0)
  const score = ref(0)
  const best = ref(window.localStorage.best || 0)
  const state = ref("init")
  const prevTapTime = ref(0)
  const colors = ref([
    '#F26D22',
    '#F1B21D',
    '#ECE82F',
    '#AED136',
    '#7EC242',
    '#63BC46',
    '#65BD5D',
    '#64C29A',
    '#61C8D2',
    '#2DAAE1',
    '#456EB5',
    '#4451A3',
    '#6151A2'
  ])

  function arcDValue():string{
    return describeArc(50, 50, 40, arc.value[0], arc.value[1]);
  }

  function polarToCartesian(centerX:number, centerY:number, radius:number, angleInDegrees:number){
      const angleInRadians = ((angleInDegrees - 90) * Math.PI) / 180.0;
      return {
        x: centerX + radius * Math.cos(angleInRadians),
        y: centerY + radius * Math.sin(angleInRadians),
      }
  }

  function describeArc(x:number, y:number, radius:number, startAngle:number, endAngle:number):string{
    const start = polarToCartesian(x, y, radius, endAngle);
    const end = polarToCartesian(x, y, radius, startAngle);
    const arcFlag = endAngle - startAngle <= 180 ? "0" : "1";
    const d = "M"+ " " + start.x + " " + start.y + " " + "A" + " " + radius + " " + radius + " " + 0 + " " + arcFlag + "" + 0 + " " + end.x + " "+ end.y
    return d;
  }

  function getAngle(cx:any, cy:any, ex:any, ey:any) {
    const dy = ey - cy;
    const dx = ex - cx;
    let theta = Math.atan2(dx, -dy);
    theta *= 180 / Math.PI;
    theta = theta < 0 ? theta + 360 : theta;
    return theta
  }

  function getBallAngle() {
    const bg = (document.getElementById("bg")as HTMLInputElement).getBoundingClientRect();
    const bgCenter = { x: bg.left + bg.width / 2, y: bg.top + bg.height / 2 };
    const ball = (document.getElementById("ball")as HTMLInputElement).getBoundingClientRect();
    const ballCenter = { x: ball.left + ball.width / 2, y: ball.top + ball.height / 2 };
    return getAngle(bgCenter.x, bgCenter.y, ballCenter.x, ballCenter.y);
  }

  function setArc() {
    const random = (i:any, j:any) => Math.floor(Math.random() * (j - i)) + i;
    let arc1 = [];
    arc1.push(random(0, 300));
    arc1.push(random(arc1[0] + 10, arc1[0] + 110));
    arc1[1] = arc1[1] > 360 ? 360 : arc1[1];
    arc.value = arc1;
  }

  function startPlay() {
    state.value = "started";
    taps.value = 0;
    score.value = 0;
    prevTapTime.value = Date.now();
  }

  function stopPlay() {
    if (state.value === "started") {
      state.value = "stopped";
      if (score.value > best.value) window.localStorage.best = best.value = score.value
    }
  }

  function tap(e:any) {
    e.preventDefault();
    e.stopPropagation();
    if (state.value === "started") {
      const ballAngle = getBallAngle();
      // adding a 6 for better accuracy as the arc stroke extends beyond the angle.
      if (ballAngle + 6 > arc.value[0] && ballAngle - 6 < arc.value[1]) {
        const currentTapTime = Date.now();
        const tapInterval = currentTapTime - prevTapTime.value;
        taps.value++;
        score.value = score.value + (tapInterval < 500 ? 5 : tapInterval < 1000 ? 2 : 1);
        prevTapTime.value = currentTapTime;
        setArc();
      } else {
        stopPlay()
      }
    }
  }

  if ("ontouchstart" in window) {
        window.addEventListener("touchstart", tap);
    } else {
        window.addEventListener("mousedown", tap);
        window.onkeypress = (e) => {
            if (e.keyCode == 32) {
                if (state.value === "stopped") {
                    startPlay();
                } else {
                    tap(e);
                }
            }
        };
    }
</script>

<template>
  <section id="canvas" v-bind:style="'display:flex;'">
        <svg id="looptap" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
            <rect id="bg" x="0" cy="0" height="100" width="100" fill="none" />
            <text x="50" y="50" dominant-baseline="middle" text-anchor="middle" class="score" id="score"
                v-if="state === 'started'" v-html="score"></text>
            <text x="50" y="32" text-anchor="middle" class="score" id="finalscore" v-if="state === 'stopped'"
                v-html="score"></text>
            <text x="50" y="70" text-anchor="middle" class="score" id="best" v-if="state === 'stopped'"
                v-html="'Best: '+best"></text>
            <g id="tip" v-if="state === 'init'">
                <text x="50" y="68" text-anchor="middle" class="tip">
                    Tap anywhere / press spacebar when
                </text>
                <text x="50" y="74" text-anchor="middle" class="tip">
                    the ball is on the colored area.
                </text>
            </g>
            <path id="arc" fill="none"
                v-bind:stroke="colors[Math.floor(score / 10)] || colors[Math.floor((score - 270) / 10)] || '#bdc3c7'"
                stroke-width="10" stroke-linejoin="round" stroke-linecap="round" v-bind:d="arcDValue()" />
            <circle id="ball" cx="50" cy="50" r="4" fill="#2C3D51" v-bind:class="state"
                v-bind:style="'animation-duration: '+(2000 - taps * 40) + 'ms'" />
            <polygon id="play" points="45,45 55,50 45,55" fill="#2C3D51" stroke="#2C3D51" stroke-width="5"
                stroke-linejoin="round" stroke-linecap="round" v-if="state !== 'started'" v-on:click="startPlay()" />
        </svg>
    </section>
</template>

<style>
html,
        body {
            overscroll-behavior: none;
        }

        body {
            font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Ubuntu, "Helvetica Neue", sans-serif;
            margin: 0;
            line-height: 1.5;
            -webkit-font-smoothing: antialiased;
            -moz-osx-font-smoothing: grayscale;
            -webkit-text-size-adjust: 100%;
            background: #fbf9f6;
        }

        a {
            color: inherit;
        }

        #canvas {
            display: none;
            height: 100vh;
            width: 100%;
            max-width: 640px;
            overflow: hidden;
            box-sizing: border-box;
            margin: auto;
            flex-direction: column;
            justify-content: center;
        }

        #canvas svg {
            width: 100%;
        }

        #about {
            text-align: center;
            font-size: 1.2rem;
            line-height: 1.75;
            padding: 0rem 2rem 1rem;
            color: #2c3d51;
        }

        #about a {
            font-weight: 500;
            font-style: italic;
        }

        #ball {
            transform: translateZ(0);
            animation: rot 2000ms infinite linear;
            animation-play-state: paused;
        }

        #ball.started {
            animation-play-state: running;
        }

        #play {
            cursor: pointer;
        }

        #finalscore,
        #best {
            font-size: 70%;
            fill: #34495e;
        }

        #best {
            font-style: italic;
            font-size: 50%;
        }

        #tip {
            font-weight: 300;
            font-size: 18%;
            font-style: italic;
            padding: 0.5rem 2rem 0rem;
        }

        #shareBtn {
            background: #1d9bf0;
            color: #fff;
            width: 150px;
            text-align: center;
            text-decoration: none;
            border-radius: 2rem;
            padding: 0.3rem 1.5rem;
            margin: 1rem auto 0px;
            font-size: 1.25rem;
        }

        #shareBtn.hide {
            visibility: hidden;
        }

        @keyframes rot {
            from {
                transform: rotate(0deg) translate(40%) rotate(0deg);
            }

            to {
                transform: rotate(360deg) translate(40%) rotate(-360deg);
            }
        }
</style>

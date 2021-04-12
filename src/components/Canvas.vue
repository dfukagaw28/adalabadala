<template>
  <div class="canvas">
    <canvas :width="width" :height="height"></canvas>
  </div>
</template>

<script>

const FRICTION = 0.99;  // ball vs wall
const FRICTION2 = 0.99;  // ball vs ball
const SPRINGRATE = 0.001;
const AIRRESIST = 0.99;

const SPEED = 1.0;
const RADIUS = 20;
const TWO_PI = Math.PI * 2;

const COLORS = [
  '#ff2800', '#faf500', '#35a16b', '#0041ff', '#66ccff',
  '#ff99a0', '#ff9900', '#9a0079', '#663300',
  '#ffd1d1', '#ffff99', '#cbf266', '#b4ebfa', '#edc58f',
  '#87e7b0', '#c7b2de', '#c8c8cb', '#7f878f',
];

const GCOLOR = [
  '#cc0000', '#00cc00', '#0000cc', '#cccc00', '#cc00cc', '#00cccc',
];

function getRandom(n) {
  return Math.random() * n;
}

function getRandomInt(n) {
  return Math.floor(getRandom(n));
}

function sample(array) {
  return array[getRandomInt(array.length)];
}

function bound(x, vx, lb, ub) {
  const w = ub - lb;
  let sgn = vx < 0 ? -1 : +1;
  let x2 = x + vx * SPEED - lb;
  if (x2 < 0) {
    x2 = -x2;
    sgn = +1;
  }
  if (x2 > 2 * w) {
    x2 = x2 % (2 * w);
  }
  if (x2 > w) {
    x2 = 2 * w - x2;
    sgn = -1;
  }
  return [lb + x2, sgn * Math.abs(vx) * FRICTION];
}

class Ball {
  constructor(canvas, x, y, radius, color) {
    this.x = x || getRandom(canvas.width);
    this.y = y || getRandom(canvas.height);
    this.r = radius || RADIUS;
    this.color = color || sample(COLORS);
    this.vx = -50 + Math.random() * 100;
    this.vy = -50 + Math.random() * 100;
  }
}

class Line {
  constructor(x1, y1, x2, y2, color) {
    this.x1 = x1;
    this.y1 = y1;
    this.x2 = x2;
    this.y2 = y2;
    this.color = color;
  }
}

function step(canvas, ball){
  const W = canvas.width;
  const H = canvas.height;

  // ball.vy += ball.ac;
  ball.y += ball.vy * SPEED;
  ball.x += ball.vx * SPEED;

  if (ball.y > H - ball.r) {
    ball.vy = - ball.vy * FRICTION;
    ball.y = H - ball.r;
  }

  if (ball.x < ball.r) {
    ball.vx = -ball.vx * FRICTION;
    ball.x = ball.r;
  }

  if (ball.x > W - ball.r) {
    ball.vx = -ball.vx * FRICTION;
    ball.x = W - ball.r;
  }

  if (ball.y < ball.r) {
    ball.vy = -ball.vy * FRICTION;
    ball.y = ball.r;
  }

  ball.vy *= AIRRESIST;
  ball.vx *= AIRRESIST;
}


function rebound(ball1, ball2) {
  let AB_X = ball2.x - ball1.x;
  let AB_Y = ball2.y - ball1.y;
  const len2 = AB_X * AB_X + AB_Y * AB_Y;
  const R = ball1.r + ball2.r;
  if (R * R < len2) return len2;

  let len = Math.sqrt(len2);
  let distance = R - len;

  if (len > 0) len = 1 / len;

  AB_X *= len;
  AB_Y *= len;

  distance /= 2;

  ball1.x -= AB_X * distance;
  ball1.y -= AB_Y * distance;
  ball2.x += AB_X * distance;
  ball2.y += AB_Y * distance;

  const ma = FRICTION2 * ((ball2.vx - ball1.vx) * AB_X + (ball2.vy - ball1.vy) * AB_Y);
  const mb = FRICTION2 * ((ball1.vx - ball2.vx) * AB_X + (ball1.vy - ball2.vy) * AB_Y);

  ball1.vx += ma * AB_X
  ball1.vy += ma * AB_Y
  ball2.vx += mb * AB_X
  ball2.vy += mb * AB_Y

  if (Math.abs(ball1.vx) <1) ball1.vx = 0;
  if (Math.abs(ball1.vy) <1) ball1.vy = 0;
  if (Math.abs(ball2.vx) <1) ball2.vx = 0;
  if (Math.abs(ball2.vy) <1) ball2.vy = 0;
}

function calcDist2(ball1, ball2) {
  const dx = ball2.x - ball1.x;
  const dy = ball2.y - ball1.y;
  return dx * dx + dy * dy;
}

function calcCenter(balls, indice) {
  let cx = 0;
  let cy = 0;
  for (const index of indice) {
    cx += balls[index].x;
    cy += balls[index].y;
  }
  cx /= indice.length;
  cy /= indice.length;
  return { x: cx, y: cy };
}

function calcCenters(balls, groups) {
  const centers = new Array(groups.length);
  for (let i = 0; i < groups.length; i++) {
    centers[i] = calcCenter(balls, groups[i]);
  }
  return centers;
}

function swap(array, i, j) {
  const temp = array[i];
  array[i] = array[j];
  array[j] = temp;
}

function replaceArray(array, newArray) {
  array.splice(0, array.length, ...newArray);
}

function updateCanvas() {
  if (!window.theCanvas.isPaused) {
    window.theCanvas.update();
    window.theCanvas.draw();
    requestAnimationFrame(updateCanvas);
  }
}

export default {
  props: {
    width: {
      type: Number,
      required: true,
    },
    height: {
      type: Number,
      required: true,
    },
    balls: {
      type: Array,
      required: true,
    },
    quotas: {
      type: Array,
      required: true,
    },
    groups: {
      type: Array,
      required: true,
    },
    ingroup: {
      type: Array,
      required: true,
    },
    onUpdateGroups: {
      type: Function,
      required: true,
    }
  },

  data() {
    return {
      centers: [],
      lines: [],
    };
  },

  computed: {
    canvas() {
      return this.$el.querySelector('canvas');
    },
    isPaused() {
      console.log( 'isPaused()' );
      try {
        return this.$parent.$refs.controller.isPaused;
      } catch (error) {
        // pass
      }
      return true;
    },
    textBalls() {
      try {
        const settings = this.$root.$children[0].$refs.settings;
        console.log( settings.items );
        return settings.items;
      } catch (error) {
        return [];
      }
    },
  },

  methods: {
    resize() {
      this.reset();
    },

    draw() {
      const canvas = this.canvas;

      this.ctx.clearRect(0, 0, canvas.width, canvas.height);

      // balls
      for (const ball of this.ballObjs) {
        this.ctx.beginPath();
        // circle
        this.ctx.fillStyle = ball.color;
        this.ctx.arc(ball.x, ball.y, ball.r, 0, TWO_PI);
        this.ctx.fill();
        // text
        this.ctx.fillStyle = "black";
        this.ctx.font = "bold 22px 'ＭＳ Ｐゴシック'";
        this.ctx.textAlign = "center";
        this.ctx.textBaseline = "middle";
        this.ctx.fillText(ball.name, ball.x, ball.y);
      }

      // lines
      for (const line of this.lines) {
        this.ctx.beginPath();
        this.ctx.strokeStyle = line.color;
        this.ctx.moveTo(line.x1, line.y1);
        this.ctx.lineTo(line.x2, line.y2);
        this.ctx.closePath();
        this.ctx.stroke();
      }
    },

    update() {
      const W = this.canvas.width;
      const H = this.canvas.height;
      const BALLS = this.ballObjs;

      // step
      /*
      for (const ball of this.ballObjs) {
        [ball.x, ball.vx] = bound(ball.x, ball.vx, ball.r, W - ball.r);
        [ball.y, ball.vy] = bound(ball.y, ball.vy, ball.r, H - ball.r);
      }
      */
     bound, H, W;

      for (let i = BALLS.length - 1; i >= 0; i--) {
        for (let j = 0; j < this.centers.length; j++) {
          const c = this.centers[j];
          const dx = c.x - BALLS[i].x;
          const dy = c.y - BALLS[i].y;
          if (j == this.ingroup[i]) {
            BALLS[i].vx += dx * SPRINGRATE;
            BALLS[i].vy += dy * SPRINGRATE;
          } else {
            BALLS[i].vx += - 0.3 / (dx * dx * this.centers.length);
            BALLS[i].vy += - 0.3 / (dy * dy * this.centers.length);
          }
          const vmax = BALLS[i].r;
          if (BALLS[i].vx > +vmax) { BALLS[i].vx = +vmax; }
          if (BALLS[i].vx < -vmax) { BALLS[i].vx = -vmax; }
          if (BALLS[i].vy > +vmax) { BALLS[i].vy = +vmax; }
          if (BALLS[i].vy < -vmax) { BALLS[i].vy = -vmax; }

          step(this.canvas, BALLS[i]);
        }
      }

      // collision between balls
      for (let i = 0; i < this.ballObjs.length; i++) {
        const b1 = this.ballObjs[i];
        for (let j = i + 1; j < this.ballObjs.length; j++) {
          const b2 = this.ballObjs[j];
          rebound(b1, b2);
        }
      }

      // clustering
      this.updateGroups();
    },

    initGroupArrays(ingroup) {
      const N = ingroup.length;
      const M = this.quotas.length;
      const groups = new Array(M);
      for (let j = 0; j < M; j++) {
        groups[j] = [];
        groups[j].index = j;
        groups[j].color = GCOLOR[j % GCOLOR.length];
      }
      for (let i = 0; i < N; i++) {
        groups[ingroup[i]].push(i);
      }
      // Renumber
      groups.sort((a,b) => - (a.length - b.length) * N * 2 + (a[0] - b[0]));
      for (let j = 0; j < M; j++) {
        for (const gjk of groups[j]) {
          ingroup[gjk] = j;
        }
      }
      return groups;
    },

    updateGroups() {
      const N = this.balls.length;
      const M = this.groups.length;
      //const W = this.canvas.width;
      //const H = this.canvas.height;
      const BALLS = this.ballObjs;

      // clear lines
      this.lines.splice(0);

      // distance matrix
      const dist2 = new Array(N);
      for (let i = 0; i < N; i++) {
        dist2[i] = new Array(N);
      }
      //const INFTY = (W + H) * (W + H);
      for (let i = 0; i < N; i++) {
        const b1 = BALLS[i];
        for (let j = 0; j < N; j++) {
          const b2 = BALLS[j];
          dist2[i][j] = i == j ? Infinity : calcDist2(b1, b2);
        }
      }

      let groups, ingroup, centers;

      for (let repeat = 0; repeat < 100; repeat++) {
        //console.log({repeat});
        if (M < 2) { break; }

        centers = calcCenters(BALLS, this.groups);

        // Compute the nearest
        const nearest = new Array(N);
        for (let i = 0; i < N; i++) {
          const jbegin = i == 0 ? 1 : 0;
          nearest[i] = jbegin;
          let dmin = calcDist2(BALLS[i], centers[jbegin]);
          for (let j = jbegin + 1; j < M; j++) {
            if (j == i) { continue; }
            const dj = calcDist2(BALLS[i], centers[j]);
            if (dj < dmin) {
              nearest[i] = j;
              dmin = dj;
            }
          }
        }

        // Update groups, ingroup, centers
        ingroup = nearest.slice();
        groups = this.initGroupArrays(ingroup);
        centers = calcCenters(BALLS, groups);

        // Adjust
        const excess = [];
        const insuff = [];
        for (let j = 0; j < M; j++) {
          let gj = groups[j];
          const Qj = this.quotas[j];
          // If the cluster is too large
          if (gj.length > Qj) {
            const cj = centers[j];
            // Sort the members in ascending order of the distance from cj
            const temp = gj.map((gjk) => {
              const d = calcDist2(BALLS[gjk], cj);
              return [gjk, d];
            });
            temp.sort(function(a,b){return a[1]-b[1];});
            gj = [];
            for (let k = 0; k < Qj; k++) {
              gj.push(temp[k][0]);
            }
            for (let k = Qj; k < temp.length; k++) {
              excess.push(temp[k][0]);
            }
            groups[j] = gj;
          } else if (gj.length < this.quotas[j]) {
            insuff.push(j);
          }
        }

        while (excess.length > 0) {
          //console.log(`#excess=${excess.length}, #insuff=${insuff.length}`);

          const pairs = [];
          let k1, k2;
          if (insuff.length > 1) {
            excess.forEach((i, k1) => {
              const bi = BALLS[i];
              insuff.forEach((j, k2) => {
                const cj = centers[j];
                const d = calcDist2(bi, cj);
                pairs.push(new Array(k1, k2, d));
              });
            });
            pairs.sort(function(a,b){return a[2]-b[2]});
            k1 = pairs[0][0];
            k2 = pairs[0][1];
          } else {
            k1 = excess.length - 1;
            k2 = 0;
          }
          const i = excess[k1];
          const j = insuff[k2];
          ingroup[i] = j;
          groups[j].push(i);
          centers[j] = calcCenter(BALLS, groups[j]);
          
          if (groups[j].length == this.quotas[j]) {
            swap(insuff, k2, insuff.length - 1);
            insuff.pop();
          }
          
          swap(excess, k1, excess.length - 1);
          excess.pop();
        }
      }

      this.onUpdateGroups(ingroup);

      // Replace arrays
      replaceArray(this.groups, groups);
      replaceArray(this.ingroup, ingroup);
      replaceArray(this.centers, centers);

      // Compute lines
      const lines = [];
      groups.forEach((gj, j) => {
        const cj = GCOLOR[j % GCOLOR.length];
        for (let k1 = 0; k1 < gj.length; k1++) {
          const i1 = gj[k1];
          const b1 = BALLS[i1];
          for (let k2 = k1 + 1; k2 < gj.length; k2++) {
            const i2 = gj[k2];
            const b2 = BALLS[i2];
            const line = new Line(b1.x, b1.y, b2.x, b2.y, cj);
            lines.push(line);
          }
        }
      });
      replaceArray(this.lines, lines);
    },

    reset() {
      //console.log('canvas reset()');

      const canvas = this.canvas;

      // Stop the timer event
      this.$parent.$refs.controller.isPaused = true;

      // Ball Objects
      const _balls = [];
      for (let i = 0; i < this.balls.length; i++) {
        const ball = new Ball(canvas);
        //const hue = Math.floor(360 / this.balls.length * i);
        ball.name = this.balls[i].name;
        //ball.color = `hsl(${hue},100%,50%)`;
        ball.color = COLORS[i % COLORS.length];
        ball.index = i;
        _balls.push(ball);
      }
      //console.log(_balls);
      this.ballObjs = _balls;

      // Groups
      const ingroup = new Array(this.balls.length);
      for (let i = 0; i < ingroup.length; i++) {
        ingroup[i] = i % this.quotas.length;
      }
      const groups = this.initGroupArrays(ingroup);
      const centers = calcCenters(this.ballObjs, groups);

      replaceArray(this.groups, groups);
      replaceArray(this.ingroup, ingroup);
      replaceArray(this.centers, centers);

      replaceArray(this.lines, []);

      this.draw();
    },

    start() {
      console.log('start()')
      window.theCanvas = this;
      this.$parent.$refs.controller.isPaused = false;
      requestAnimationFrame(updateCanvas);
    },

    pause() {
      this.$parent.$refs.controller.isPaused = true;
    },

    fitToWindow() {
      //this.$el.width = w;
      //this.$el.height = h;
    },
  },

  mounted() {
    this.ctx = this.canvas.getContext('2d');
    this.reset();
  },
}
</script>

<style scoped>
canvas {
  border: 1px solid #333333;
  max-width: 100%;
}
</style>

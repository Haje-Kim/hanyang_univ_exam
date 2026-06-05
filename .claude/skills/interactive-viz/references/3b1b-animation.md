# 3B1B 애니메이션 씬 — 자체 경량 엔진 + 패턴

> interactive-viz-engineer 전용. 3Blue1Brown식 "움직이는 시각"을 **CDN 의존 0**의 vanilla JS로 구현한다.
> 오프라인 우선 원칙 때문에 manim·p5.js CDN에 의존하지 않는다. requestAnimationFrame + easing만으로 충분하다.

## 목차
1. 설계 원칙
2. `sceneEngine` — 공유 엔진 (과목당 1회 삽입)
3. 씬 작성 패턴 (단계 시퀀스 + 캡션 + 컨트롤)
4. 워크드 예제 3종 (정렬 모핑 · 경사하강 · attention 흐름)
5. 테마·반응형·접근성
6. QA 체크리스트

---

## 1. 설계 원칙

- **씬 = 상태(step)의 시퀀스.** 각 step은 그릴 데이터의 스냅샷. step 사이는 **보간(interpolate)**으로 모핑.
- **그리기는 순수 함수.** `draw(ctx, state)` — 주어진 state를 그릴 뿐, 시간을 모른다. 엔진이 state를 보간해 넘긴다.
- **사용자 주도.** 자동재생 금지. play/pause + 단계 스크럽(◀ ▶) + 진행 슬라이더.
- **점진 등장.** step 데이터에 `appear: 0→1` 같은 필드를 두고 보간해 페이드/슬라이드 인.
- **일관 색.** 의미별 색을 토큰에서 읽어 고정(`getColor('accent')`).

## 2. sceneEngine — 공유 엔진

과목 HTML에 **한 번만** 삽입(여러 씬이 공유). `<script>` 블록:

```html
<script>
/* ===== sceneEngine: CDN-free 3B1B-style animation (offline-first) ===== */
window.sceneEngine = (function () {
  const easings = {
    linear: t => t,
    smooth: t => t * t * (3 - 2 * t),            // smoothstep (3b1b 기본 느낌)
    easeOut: t => 1 - Math.pow(1 - t, 3),
    easeInOut: t => t < .5 ? 4*t*t*t : 1 - Math.pow(-2*t+2,3)/2,
  };
  const lerp = (a, b, t) => a + (b - a) * t;
  // 숫자/배열/객체(얕은 수치 필드) 보간
  function interp(a, b, t) {
    if (typeof a === 'number') return lerp(a, b, t);
    if (Array.isArray(a)) return a.map((x, i) => interp(x, b[i], t));
    if (a && typeof a === 'object') {
      const o = {}; for (const k in a) o[k] = interp(a[k], b[k], t); return o;
    }
    return t < 1 ? a : b;  // 비수치는 끝에서 스냅
  }
  // 테마 토큰 색 읽기 (다크모드 전환 대응)
  // ⚠ CSS 변수는 테마가 걸린 요소(html 또는 body)의 "하위"에서만 오버라이드된다.
  //   data-theme를 body에 거는 프로젝트(예: 학습허브)에서 documentElement(=html)를
  //   읽으면 항상 :root(기본 테마) 값만 나온다. 그래서 색은 "씬 캔버스 element"에서 읽는다
  //   (캔버스는 body 하위라 테마가 정상 cascade된다). create(cv,...) 안에서 cv를 클로저로 받는다.
  function makeColor(el) {
    return (name, fallback) => {
      const v = getComputedStyle(el).getPropertyValue('--' + name).trim();
      return v || fallback;
    };
  }
  const reduced = matchMedia('(prefers-reduced-motion: reduce)').matches;

  // 캔버스 HiDPI 셋업
  function fitCanvas(cv) {
    const dpr = window.devicePixelRatio || 1;
    const r = cv.getBoundingClientRect();
    cv.width = r.width * dpr; cv.height = r.height * dpr;
    const ctx = cv.getContext('2d');
    ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
    return { ctx, w: r.width, h: r.height };
  }

  /* Scene: steps=상태 배열, draw=(ctx,state,helpers)=>void, caption=(i)=>string */
  function create(cv, { steps, draw, caption, duration = 700, easing = 'smooth' }) {
    let i = 0, raf = 0;
    const color = makeColor(cv);                       // 테마 변수는 캔버스에서 읽는다
    const capEl = cv.parentElement.querySelector('[data-caption]');
    function render(state) {
      const { ctx, w, h } = fitCanvas(cv);
      ctx.clearRect(0, 0, w, h);
      draw(ctx, state, { w, h, color });
    }
    function show(idx) {        // 모핑 없이 즉시
      i = idx; render(steps[i]); if (capEl && caption) capEl.textContent = caption(i);
    }
    function go(idx) {          // idx로 모핑 전환
      idx = Math.max(0, Math.min(steps.length - 1, idx));
      if (reduced || idx === i) return show(idx);
      cancelAnimationFrame(raf);
      const from = steps[i], to = steps[idx], t0 = performance.now();
      if (capEl && caption) capEl.textContent = caption(idx);
      (function frame(now) {
        const p = Math.min(1, (now - t0) / duration);
        render(interp(from, to, easings[easing](p)));
        if (p < 1) raf = requestAnimationFrame(frame); else i = idx;
      })(t0);
    }
    function next() { go(i + 1); } function prev() { go(i - 1); }
    // 테마/리사이즈 시 현재 프레임 재그리기. data-theme를 html에 거는 프로젝트도,
    // body에 거는 프로젝트도 있으므로 둘 다 관찰한다.
    const repaint = () => show(i);
    window.addEventListener('resize', repaint);
    const mo = new MutationObserver(repaint);
    mo.observe(document.documentElement, { attributes: true, attributeFilter: ['data-theme', 'class'] });
    mo.observe(document.body, { attributes: true, attributeFilter: ['data-theme', 'class'] });
    show(0);
    return { next, prev, go, show, get index() { return i; }, len: steps.length };
  }
  return { create, color, lerp, interp, easings };
})();
</script>
```

> performance.now()는 브라우저 런타임 API라 사용 가능(워크플로 스크립트의 Date.now 제약과 무관 — 여기는 HTML 산출물 내부 코드다).

## 3. 씬 작성 패턴

각 씬은 마크업 + 씬 정의로 구성. concept-explainer의 `<!-- VIZ: ... -->` 마커 자리에 삽입.

```html
<figure class="viz-scene">
  <canvas id="sceneSort" style="width:100%;height:280px"></canvas>
  <figcaption data-caption class="viz-caption"></figcaption>
  <div class="viz-controls">
    <button data-prev aria-label="이전 단계">◀</button>
    <button data-play>▶ 재생</button>
    <button data-next aria-label="다음 단계">▶</button>
    <input type="range" data-scrub min="0" step="1" aria-label="단계 이동">
  </div>
</figure>
```

컨트롤 배선은 작은 헬퍼로 반복 제거:

```js
function wireScene(scene, root) {
  const scrub = root.querySelector('[data-scrub]');
  scrub.max = scene.len - 1;
  root.querySelector('[data-next]').onclick = () => { scene.next(); scrub.value = scene.index; };
  root.querySelector('[data-prev]').onclick = () => { scene.prev(); scrub.value = scene.index; };
  scrub.oninput = e => scene.go(+e.target.value);
  const playBtn = root.querySelector('[data-play]');
  let timer = null;
  playBtn.onclick = () => {
    if (timer) { clearInterval(timer); timer = null; playBtn.textContent = '▶ 재생'; return; }
    playBtn.textContent = '⏸ 일시정지';
    timer = setInterval(() => {
      if (scene.index >= scene.len - 1) { clearInterval(timer); timer = null; playBtn.textContent = '▶ 재생'; return; }
      scene.next(); scrub.value = scene.index;
    }, 900);
  };
}
```

## 4. 워크드 예제 3종

### (A) 정렬 모핑 — 막대가 자리를 바꾸며 정렬 (type: morph)
각 step = 막대 높이 배열 + 강조 인덱스. 비교 중인 두 막대는 accent 색, 확정된 막대는 green. step 간 높이/위치를 보간하면 **막대가 미끄러져 자리를 바꾸는** 3B1B식 모핑이 된다.

```js
const data = [5,2,4,1,3];
const steps = buildInsertionSortSteps(data); // [{bars:[...], cmp:[i,j], sorted:k}, ...]
const scene = sceneEngine.create(document.getElementById('sceneSort'), {
  steps,
  caption: i => steps[i].note,   // "2와 5 비교 → 2가 작으므로 왼쪽으로 이동"
  draw(ctx, s, { w, h, color }) {
    const n = s.bars.length, bw = w / (n * 1.6), unit = (h - 40) / Math.max(...data);
    s.bars.forEach((v, k) => {
      const x = (k + .3) * (w / n), bh = v * unit;
      ctx.fillStyle = k <= s.sorted ? color('ok', '#22c55e')
                    : s.cmp && s.cmp.includes(k) ? color('accent', '#3b82f6')
                    : color('muted', '#94a3b8');
      ctx.fillRect(x, h - bh - 20, bw, bh);
    });
  }
});
```
`buildInsertionSortSteps`는 정렬을 돌리며 매 비교·이동마다 스냅샷을 push(라벨 note 포함). **알고리즘의 각 단계가 곧 한 프레임** — 시험에 나오는 "한 줄씩 추적"을 눈으로 본다.

### (B) 경사하강 — 손실면 위를 굴러 내려가는 점 (type: morph)
step = 점의 (x, y=loss(x)). 곡선은 고정으로 그리고 점만 보간 → **점이 곡선을 따라 내려가며 최소점으로 수렴**. 학습률을 슬라이더로 바꾸면(시뮬레이션 패턴 결합) 큰 학습률에서 **튕기며 발산**하는 것까지 보인다.

```js
const loss = x => 0.5*x*x + Math.sin(2*x)*0.6 + 2;   // 다봉형
let x = -3.2; const lr = 0.15, steps = [{x, y: loss(x), note:'시작점 — 기울기 큰 곳'}];
for (let k=0;k<14;k++){ const g=(loss(x+1e-3)-loss(x-1e-3))/2e-3; x-=lr*g;
  steps.push({x, y: loss(x), note:`step ${k+1}: 기울기 ${g.toFixed(2)} 반대로 이동`}); }
// draw: 곡선(고정) + 현재 점(accent) + 기울기 접선
```
캡션이 "기울기가 0에 가까워질수록 보폭이 줄어 멈춘다"를 단계마다 말해준다 → gradient descent(경사하강)의 **왜**를 본다.

### (C) attention 흐름 — 토큰을 잇는 가중 엣지 (type: structure→morph)
step = 쿼리 토큰에서 각 키 토큰으로 가는 엣지의 굵기(=attention weight). 한 step씩 가중치가 재분배되며 **굵은 선이 관련 토큰으로 모이는** 흐름. GAT/Transformer 설명용. 색=accent, 굵기=weight*k. softmax 정규화로 합=1을 캡션에 표시.

## 5. 테마·반응형·접근성

- **테마:** 색은 항상 `color('accent', fallback)`로 토큰에서 읽는다. 엔진이 `data-theme`/`class` 변경을 감지해 자동 repaint.
- **반응형:** Canvas는 `width:100%` + `fitCanvas`의 devicePixelRatio 스케일. resize 시 자동 repaint. 모바일에서 컨트롤 세로 스택(CSS).
- **접근성:** `reduced`(prefers-reduced-motion)면 모핑 대신 즉시 `show` — 단계는 그대로 보되 움직임만 끈다. 버튼에 aria-label, 슬라이더로 키보드 단계 이동. 자동재생은 사용자가 누를 때만.

## 6. QA 체크리스트 (harness-qa 전달)

- [ ] 네트워크 차단(오프라인)에서도 씬이 그려진다(CDN 0이므로 당연하지만 확인)
- [ ] 다크↔라이트 전환 시 씬 색이 토큰 따라 바뀐다(잔상·하드코드 색 없음)
- [ ] prefers-reduced-motion에서 모핑이 꺼지고 단계는 보인다
- [ ] 모바일 폭에서 캔버스·컨트롤이 깨지지 않는다(리사이즈 repaint)
- [ ] 콘솔 에러 0, 캡션이 단계와 동기
- [ ] 단계 스크럽(◀▶·슬라이더)이 양방향 동작, 재생/일시정지 토글

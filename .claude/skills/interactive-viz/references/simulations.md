# 개념 시뮬레이션 — 패턴 모음

단계가 있거나 파라미터에 반응하는 개념을 "조작 가능"하게 만드는 경량 패턴. 외부 라이브러리 없이 vanilla JS + SVG/Canvas로 구현(D3는 트리에만, 시뮬레이션은 의존 최소화 → 오프라인 안전).

## 목차
1. 공통 컨트롤 골격(스테퍼)
2. 정렬·DP 스테퍼 (컴퓨터 알고리즘)
3. 탐색·CSP 애니메이션 (인공지능개론)
4. GNN 메시지 패싱 (의생명 AI)
5. ROC·임계값 슬라이더 (의료진단 AI)
6. 체크리스트

## 1. 공통 컨트롤 골격 (스테퍼)

모든 단계형 시뮬레이션은 같은 골격: 상태 배열 + 현재 인덱스 + ◀ ▶ 버튼 + 단계 설명 텍스트.

```html
<div class="sim" data-sim="sort">
  <div class="sim-stage"><!-- SVG/Canvas 또는 막대 div --></div>
  <div class="sim-ctrl">
    <button class="sim-prev">◀ 이전</button>
    <span class="sim-step">단계 0 / N</span>
    <button class="sim-next">다음 ▶</button>
    <button class="sim-play">▶ 자동</button>
  </div>
  <p class="sim-desc">현재 단계 설명이 여기 갱신된다.</p>
</div>
```

```js
function makeStepper(root, frames){          // frames: [{render(stage), desc}]
  let i=0, timer=null;
  const stage=root.querySelector('.sim-stage'), step=root.querySelector('.sim-step'),
        desc=root.querySelector('.sim-desc');
  function show(){ frames[i].render(stage); desc.textContent=frames[i].desc;
    step.textContent=`단계 ${i} / ${frames.length-1}`; }
  root.querySelector('.sim-prev').onclick=()=>{ if(i>0){i--;show();} };
  root.querySelector('.sim-next').onclick=()=>{ if(i<frames.length-1){i++;show();} };
  root.querySelector('.sim-play').onclick=function(){
    if(timer){clearInterval(timer);timer=null;this.textContent='▶ 자동';return;}
    this.textContent='⏸ 정지';
    timer=setInterval(()=>{ if(i<frames.length-1){i++;show();}
      else{clearInterval(timer);timer=null;this.textContent='▶ 자동';} },700);
  };
  show();
}
```

각 개념은 `frames` 배열만 만들면 된다 — 단계별 상태 + 한 줄 설명(개념 라벨 필수).

## 2. 정렬·DP 스테퍼 (컴퓨터 알고리즘)

- **정렬**: 막대 높이 배열. 각 frame은 비교/스왑 후 상태 + "i와 j 비교, swap" 설명. 비교 중인 인덱스는 강조색.
- **DP**: 표(2D 배열) 셀을 단계별로 채운다. frame마다 채워지는 셀 하이라이트 + 점화식 설명("dp[i][j]=max(...)"). KaTeX로 점화식 렌더.

## 3. 탐색·CSP 애니메이션 (인공지능개론)

- **지역탐색/CSP**: 격자나 그래프에서 현재 상태·이웃 후보·선택을 색으로 구분. frame = 한 번의 이동, 설명 = "충돌 수 3 → 이웃 중 최소 1 선택". 가장 출제 우선순위 높은 토픽이므로 정성 들일 것.
- **A*/탐색 트리 확장**: open/closed 집합 색 구분, frame마다 확장 노드 + f=g+h 값 표시.

## 4. GNN 메시지 패싱 (의생명 AI)

- 작은 그래프(노드 5~7개) SVG. frame 진행에 따라 한 노드가 이웃의 메시지를 모으는 과정을 화살표 애니메이션 + 노드 임베딩 벡터 갱신 표시.
- 설명에 집계식(예: $h_v^{(k)}=\sigma(W\sum_{u\in N(v)} h_u^{(k-1)})$)을 KaTeX로. GAT면 attention 가중치를 화살표 굵기로.

## 5. ROC·임계값 슬라이더 (의료진단 AI)

- `<input type="range">` 슬라이더로 임계값 조절 → TP/FP/TN/FN 혼동행렬과 민감도·특이도·ROC 위 점이 즉시 갱신(파라미터 반응형, 스테퍼 아님).

```js
slider.addEventListener('input',e=>{
  const th=+e.target.value;
  const {tp,fp,tn,fn}=classify(scores,labels,th);
  sens.textContent=(tp/(tp+fn)).toFixed(2);
  spec.textContent=(tn/(tn+fp)).toFixed(2);
  drawConfusion(tp,fp,tn,fn); moveROCpoint(fp/(fp+tn), tp/(tp+fn));
});
```

## 6. 체크리스트

- [ ] 모든 시뮬레이션에 단계 설명/개념 라벨이 갱신된다
- [ ] 라이브러리 의존 없음(vanilla) → 오프라인에서도 동작
- [ ] 자동재생 타이머가 페이지 이탈/정지 시 clearInterval 됨(누수 방지)
- [ ] 다크모드 색 반영, 모바일에서 컨트롤 세로 스택
- [ ] 수식은 KaTeX 렌더(폴백 원문 유지)

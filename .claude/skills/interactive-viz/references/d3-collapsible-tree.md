# D3 Collapsible 탐색 트리 — 완성 패턴

기존 정적 트리맵(`.treemap` div)과 onclick 로드맵을 대체하는 D3 v7 collapsible tree. 접기/펼치기·줌·팬·노드 클릭 시 해당 본문 섹션으로 스크롤.

## 목차
1. CDN 로드 + 폴백
2. 트리 데이터 형식
3. SVG 트리 렌더 코드
4. 노드 클릭 → 섹션 드릴다운
5. 다크모드·반응형
6. 체크리스트

## 1. CDN 로드 + 폴백

`<head>`에 KaTeX와 같은 방식으로 추가:

```html
<script src="https://cdn.jsdelivr.net/npm/d3@7/dist/d3.min.js"></script>
```

폴백: 기존 `.treemap` div를 지우지 말고 `id="treemap-fallback"`로 유지한 채 숨겨둔다. 트리 컨테이너는 별도. 스크립트 끝에서:

```js
function initTree(){
  if (typeof d3 === 'undefined') {            // CDN 실패
    document.getElementById('tree-container').style.display='none';
    document.getElementById('treemap-fallback').style.display='block';  // 정적 트리맵 노출
    return;
  }
  document.getElementById('treemap-fallback').style.display='none';
  renderTree();                                // D3 성공 → 인터랙티브 트리
}
window.addEventListener('load', initTree);
```

## 2. 트리 데이터 형식

content-architect의 `_workspace/{과목}_toc.md`를 계층 JSON으로 변환. `id`는 본문 섹션 앵커와 1:1.

```js
const TREE = {
  name: "인공지능개론", id: null,
  children: [
    { name: "선행지식", id: "prereq", children: [
        { name: "탐색·최적화 기초", id: "prereq-search" },
        { name: "확률 기초", id: "prereq-prob" } ] },
    { name: "탐색", id: "grp-search", children: [
        { name: "CSP·지역탐색", id: "c5b" },
        { name: "A*", id: "c6" } ] }
  ]
};
```

## 3. SVG 트리 렌더 코드

```js
function renderTree(){
  const el = document.getElementById('tree-container');
  const width = el.clientWidth, dx = 28, dy = Math.min(220, width/4);
  const root = d3.hierarchy(TREE);
  root.x0 = 0; root.y0 = 0;
  root.descendants().forEach((d,i)=>{ d.id=i; d._children=d.children;
    if (d.depth >= 1) d.children = null;        // 1단계만 펼친 상태로 시작
  });
  const svg = d3.create("svg")
    .attr("viewBox",[-dy/2, -dx, width, dx])     // viewBox로 반응형
    .style("width","100%").style("height","auto").style("font","14px Pretendard,sans-serif")
    .style("user-select","none");
  const gLink = svg.append("g").attr("fill","none")
    .attr("stroke","var(--line,#ccc)").attr("stroke-width",1.5);
  const gNode = svg.append("g").attr("cursor","pointer").attr("pointer-events","all");

  const zoom = d3.zoom().scaleExtent([0.4,2.5]).on("zoom",e=>{
    gLink.attr("transform",e.transform); gNode.attr("transform",e.transform);
  });
  svg.call(zoom);

  function update(source){
    const nodes = root.descendants().reverse(), links = root.links();
    const tree = d3.tree().nodeSize([dx,dy]); tree(root);
    let l=root, r=root; root.eachBefore(n=>{ if(n.x<l.x)l=n; if(n.x>r.x)r=n; });
    const h = Math.max(dx, r.x-l.x+dx*2);
    const t = svg.transition().duration(250)
      .attr("viewBox",[-dy/2, l.x-dx, width, h]);

    const node = gNode.selectAll("g").data(nodes,d=>d.id);
    const nEnter = node.enter().append("g")
      .attr("transform",`translate(${source.y0},${source.x0})`)
      .attr("fill-opacity",0).attr("stroke-opacity",0)
      .on("click",(e,d)=>{ d.children=d.children?null:d._children; update(d);
        if(d.data.id) goSection(d.data.id);     // 잎 클릭 → 본문 이동
      });
    nEnter.append("circle").attr("r",4.5)
      .attr("fill",d=>d._children?"var(--accent,#0066FF)":"var(--muted,#888)")
      .attr("stroke-width",10).attr("stroke","transparent");
    nEnter.append("text").attr("dy","0.32em")
      .attr("x",d=>d._children?-9:9).attr("text-anchor",d=>d._children?"end":"start")
      .attr("fill","var(--text,#222)").text(d=>d.data.name)
      .clone(true).lower().attr("stroke","var(--panel,#fff)").attr("stroke-width",3);

    node.merge(nEnter).transition(t)
      .attr("transform",d=>`translate(${d.y},${d.x})`)
      .attr("fill-opacity",1).attr("stroke-opacity",1);
    node.exit().transition(t).remove()
      .attr("transform",`translate(${source.y},${source.x})`)
      .attr("fill-opacity",0);

    const link = gLink.selectAll("path").data(links,d=>d.target.id);
    const diag = d3.linkHorizontal().x(d=>d.y).y(d=>d.x);
    const lEnter = link.enter().append("path").attr("d",()=>{
      const o={x:source.x0,y:source.y0}; return diag({source:o,target:o}); });
    link.merge(lEnter).transition(t).attr("d",diag);
    link.exit().transition(t).remove().attr("d",()=>{
      const o={x:source.x,y:source.y}; return diag({source:o,target:o}); });

    root.eachBefore(d=>{ d.x0=d.x; d.y0=d.y; });
  }
  el.innerHTML=""; el.appendChild(svg.node());
  update(root);
}

function goSection(id){
  const t=document.getElementById(id);
  if(t) t.scrollIntoView({behavior:"smooth",block:"start"});
}
```

## 4. 노드 클릭 → 섹션 드릴다운

- 내부 노드 클릭: 자식 토글(접기/펼치기).
- 잎 노드 클릭: `goSection(id)`로 본문 앵커 이동. 기존 스크롤스파이가 있으면 그대로 동작.
- `data.id`가 본문 섹션 `id`와 정확히 일치해야 한다(harness-qa가 교차 검증).

## 5. 다크모드·반응형

- 색은 모두 CSS 변수(`var(--accent)`, `var(--line)`, `var(--text)`, `var(--panel)`)로 — 다크모드 토글 시 자동 반영.
- `viewBox` + `width:100%`로 폭 반응. 모바일은 `nodeSize` dy를 줄이고(위 `Math.min`), `svg.call(zoom)`으로 핀치/휠 줌 허용.
- 리사이즈 대응: `window.addEventListener('resize', debounce(renderTree, 200))` 선택.

## 6. 체크리스트

- [ ] D3 미로드 시 `treemap-fallback` 정적 트리맵이 보인다(오프라인 OK)
- [ ] 노드 접기/펼치기 동작, 잎 클릭 시 정확한 섹션으로 이동
- [ ] 모든 `data.id`가 실제 본문 섹션 id와 매칭(dead link 0)
- [ ] 다크모드에서 트리 색이 따라감
- [ ] 모바일 폭에서 트리가 잘리지 않고 줌 가능

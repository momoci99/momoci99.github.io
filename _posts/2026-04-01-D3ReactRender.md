---
layout: single
title: "react 및 d3의 렌더링 방법"
tag:
  - React
  - d3
---

d3는 데이터 시각화에 있어, 매우 강력하지만 저 수준(low-level) 라이브러리입니다. react와 결합했을때, 잘못사용하면 충돌이 발생하기 쉽습니다. 이 글에서는 충돌을 해결하는 두 가지 방식을 정의합니다.

# 방식 1 - d3 로 모든 dom을 처리하기

d3가 차트 렌더링을 위한 dom을 완전히 소유하는 방식입니다. React는 dom 과 연결된 ref(`<svg ref={ref} />`)만 제공하고, 이후 렌더링은 d3가 `useEffect` 안에서 처리합니다.

예시 코드

```tsx
const BarChartPureD3 = ({ data }: { data: DataItem[] }) => {
  const svgRef = useRef<SVGSVGElement>(null);

  useEffect(() => {
    const svg = d3.select(svgRef.current);
    svg.selectAll("*").remove();

    const g = svg.append("g").attr("transform", `translate(${MARGIN.left},${MARGIN.top})`);
    const xScale = d3.scaleBand()...
    const yScale = d3.scaleLinear()...

    // D3가 grid, bar, axis를 직접 생성
    g.selectAll(".grid-line").data(yScale.ticks()).join("line")...
    g.selectAll("rect").data(data).join("rect")...
    g.append("g").call(d3.axisBottom(xScale));
    g.append("g").call(d3.axisLeft(yScale));
  }, [data]);

  return <svg ref={svgRef} width={WIDTH} height={HEIGHT} />;
};
```

- d3의 `transition`, `zoom`, `brush` 같은 인터랙션을 그대로 사용 할 수 있습니다.
- `selectAll("*").remove()`로 매 렌더마다 초기화가 필요합니다. d3가 제공하는 `join` 으로 diff 요소만 처리할 수 있지만 react 의 컴포넌트 생명주기와 맞물려서 의도하지 않은 렌더링 문제가 발생할 수 있습니다.
- 컴포넌트 재사용이 어렵습니다. d3 계산 로직과 강한 결합도를 가집니다.
- React DevTools로 추적이 어렵습니다. dom이 d3에 의해 직접 조작되므로, React의 가상 DOM과 일치하지 않기 때문입니다.

<br>

# 방식 2 - d3는 계산만 렌더링은 react

d3의 `scale`, `path generator` 와 같은 수학적인 유틸리티만 사용하고(계산만), 실제 dom은 jsx로 렌더링하는 방식입니다.

예시 코드

```tsx
const BarChartReact = ({ data }: { data: DataItem[] }) => {
  const xScale = useMemo(() => d3.scaleBand()..., [data]);
  const yScale = useMemo(() => d3.scaleLinear()..., [data]);

  return (
    <svg width={WIDTH} height={HEIGHT}>
      <g transform={`translate(${MARGIN.left},${MARGIN.top})`}>
        {/* grid, bar, axis를 하나의 컴포넌트 안에서 직접 렌더링 */}
        {yScale.ticks().map((tick) => <line key={tick} ... />)}
        {data.map((d) => <rect key={d.label} ... />)}
        {xScale.domain().map((tick) => <text key={tick} ... />)}
      </g>
    </svg>
  );
};
```

- React 생태계와 완전히 통합하는 방식입니다. (상태 관리, 테스트, SSR 등)
- `useMemo` 를 적용하지 않으면 불필요한 재계산이 일어나니 주의해야합니다.

# 방식 2 심화 - 컴포넌트로 분리

재사용 가능한 UI 요소 (위에서 예시로 든 grid, axis)는 독립 컴포넌트로 분리하는 방식입니다.

예시 코드

```tsx
const Grid = ({ scale, width }: GridProps) => (
  <g>{scale.ticks().map((tick) => <line key={tick} ... />)}</g>
);
const XAxis = ({ scale, height }: XAxisProps) => (...);
const YAxis = ({ scale }: YAxisProps) => (...);

const BarChartReact = ({ data }: { data: DataItem[] }) => {
  const xScale = useMemo(() => d3.scaleBand()..., [data]);
  const yScale = useMemo(() => d3.scaleLinear()..., [data]);

  return (
    <svg width={WIDTH} height={HEIGHT}>
      <g transform={`translate(${MARGIN.left},${MARGIN.top})`}>
        <Grid scale={yScale} width={INNER_WIDTH} />
        {data.map((d) => <rect key={d.label} ... />)}
        <XAxis scale={xScale} height={INNER_HEIGHT} />
        <YAxis scale={yScale} />
      </g>
    </svg>
  );
};
```

- scale을 prop으로 받는 구조로 만들면 LineChart, AreaChart 등 어떤 차트에도 재사용할 수 있습니다.
- 컴포넌트의 재활용 부분 및 유지보수 측면에서 더 좋은 방식이라고 볼 수 있습니다.

---

참고한 글

- https://2019.wattenberger.com/blog/react-and-d3

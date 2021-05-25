# SVG

## Create SVG

You can create SVG using JSX or lit-HTML. For example, below is a reimplementation of <a href="https://github.com/snabbdom/snabbdom/tree/master/examples/carousel-svg">a snabbdom example</a> by Jon Kleiser (@jkleiser).

### SVG use JSX

```js
const style=`
--8<-- "svg.css"
`
const triangles = [
{id: "yellow", rot: 0},
{id: "green", rot: 60},
{id: "magenta", rot: 120},
{id: "red", rot: 180},
{id: "cyan", rot: 240},
{id: "blue", rot: 300}
];

class SvgComponent extends Component {
  state = 0;

  view = state => {
    return <div className="view">
    <style>{style}</style>
      <svg width="380" height="380" viewBox="-190,-190,380,380">
        <g id="carousel" transform={`rotate(${state})`}>
          {triangles.map(t =>
            <polygon id={t.id}
              points="-50,-88 0,-175 50,-88"
              transform={`rotate(${t.rot})`}
              stroke-width="3" />
          )}
        </g>
      </svg>
      <button $onclick="rot+60">Rotate Clockwise</button>
      <button $onclick="rot-60">Rotate Anticlockwise</button>
      <button $onclick="reset">Reset</button>
    </div>
  };

  update = {
    "rot+60": state => state + 60,
    "rot-60": state => state - 60,
    "reset": state => 0,
  };
}
new SvgComponent().start(document.body);
```
<apprun-play style="height:480px"></apprun-play>


### SVG use lit-HTML

```js
const style=`
--8<-- "svg.css"
`
const triangles = [
{id: "yellow", rot: 0},
{id: "green", rot: 60},
{id: "magenta", rot: 120},
{id: "red", rot: 180},
{id: "cyan", rot: 240},
{id: "blue", rot: 300}
];

class SvgComponent extends Component {
  state = 0;

  view = state => {
    const items = triangles.map(t =>
      svg`<polygon id="${t.id}"
        points="-50,-88 0,-175 50,-88"
        transform="rotate(${t.rot})"
        stroke-width="3" />`
    );
    return html`<div class="view">
      <style>${style}</style>
      <svg width="380" height="380" viewBox="-190,-190,380,380">
        <g id="carousel" style="transform: rotate(${state}deg);">
          ${items}
        </g>
      </svg>
      <button @click=${run("@rot+60")}>Rotate Clockwise</button>
      <button @click=${run("@rot-60")}>Rotate Anticlockwise</button>
      <button @click=${run("@reset")}>Reset</button>
    </div>`;
  };

  update = {
    "@rot+60": state => state + 60,
    "@rot-60": state => state - 60,
    "@reset": () => 0,
  };
}
new SvgComponent().start(document.body);
```
<apprun-play style="height:480px"></apprun-play>

## SVG - xlink:href

You can use _xlinkHref_ to define the _xlink:href_ attribute of SVG.

```js
// SVG - xlink
const view = () => <svg viewBox="0 0 150 20">
  <a xlinkHref="https://apprun.js.org/">
    <text x="10" y="10" font-size="5">Click Here</text></a>
</svg>

app.start(document.body, {}, view);
```
<apprun-play></apprun-play>


##  SVG Event Handlers

You can handle the _onclick_ event of SVG elements. Or use the _$onclick_ directive

```js
// SVG - $onclick
const view = state => <>
<div>click the buttons:</div>
<svg viewBox="0 0 520 520" xmlns="http://www.w3.org/2000/svg">
  <rect x="10" y="10" width="90" height="20" fill="#aaa"
    $onclick="test" id="$onclick"/>
  <rect x="110" y="10" width="90" height="20" fill="#bbb"
    onclick="app.run('test', event)" id="onclick"/>
</svg>
</>

const update = {
  test: (state, evt) => alert("You have used: " + evt.target.id)
}
app.start(document.body, '', view, update);
```
<apprun-play></apprun-play>

## SVG Animation

You can use the _animate_ element to add animation to SVG.

```js
// SVG - animation
const view = () => <>
  <svg height="60" width="160">
    <rect width="100%" height="100%" rx="10" ry="10" fill="lightgrey" />
    <circle cx="30" cy="30" r="20" fill='lime'>
    <animate
          attributeType="XML"
          attributeName="fill"
          values="lime;lightgrey;lime;lightgrey"
          dur="0.5s"
          repeatCount="indefinite"/>
    </circle>
    <circle cx="80" cy="30" r="20" fill='yellow' fill-opacity='0.2'/>
    <circle cx="130" cy="30" r="20" fill='orangered' fill-opacity='0.2' />
  </svg>
</>;
app.start(document.body, {}, view);
```
<apprun-play></apprun-play>

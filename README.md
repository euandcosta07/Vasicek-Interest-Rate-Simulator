# Vasicek-Interest-Rate-Simulator
Vasicek Interest Rate Simulator
<!DOCTYPE html>
<html lang="en-GB">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Vasicek interest rate simulator</title>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Newsreader:opsz,wght@6..72,500;6..72,600&family=Public+Sans:wght@400;500;600&display=swap">
<style>
:root{
  color-scheme:light;
  --bg:#f2f6f8;--panel:#ffffff;--ink:#13222c;--muted:#52656f;--line:#cfdbe1;
  --accent:#0f6a8a;--band:rgba(15,106,138,.14);--path:rgba(70,100,116,.28);--warn:#b4491f;
  box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px);
}
@media (prefers-color-scheme:dark){
  :root:not([data-theme="light"]){color-scheme:dark;--bg:#0e1a21;--panel:#14252e;--ink:#e5eef2;--muted:#9bb0ba;--line:#2a404b;
  --accent:#5cc2e4;--band:rgba(92,194,228,.16);--path:rgba(155,190,206,.26);--warn:#f09a6b}
}
:root[data-theme="dark"]{color-scheme:dark;--bg:#0e1a21;--panel:#14252e;--ink:#e5eef2;--muted:#9bb0ba;--line:#2a404b;
  --accent:#5cc2e4;--band:rgba(92,194,228,.16);--path:rgba(155,190,206,.26);--warn:#f09a6b}
html{scroll-padding-top:env(safe-area-inset-top,0px)}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--ink);font:16px/1.55 "Public Sans",system-ui,-apple-system,"Segoe UI",sans-serif}
main{max-width:980px;margin:0 auto;padding:28px 18px 56px}
h1{font:600 clamp(1.9rem,5vw,2.8rem)/1.1 "Newsreader",Georgia,serif;margin:0 0 10px;letter-spacing:-.01em}
h2{font:600 1.35rem/1.2 "Newsreader",Georgia,serif;margin:36px 0 8px}
.lede{max-width:62ch;color:var(--muted);margin:0 0 22px}
.eq{font-family:"Newsreader",Georgia,serif;font-size:1.15rem;color:var(--ink);background:var(--panel);border:1px solid var(--line);border-radius:6px;display:inline-block;padding:6px 12px;margin:0 0 22px}
.chart{background:var(--panel);border:1px solid var(--line);border-radius:8px;padding:10px}
canvas{display:block;width:100%;height:clamp(260px,46vw,420px)}
.legend{display:flex;flex-wrap:wrap;gap:6px 18px;font-size:.85rem;color:var(--muted);padding:6px 6px 2px}
.legend i{display:inline-block;width:18px;height:0;border-top:2px solid var(--accent);vertical-align:middle;margin-right:6px}
.legend i.p{border-top-color:var(--path);border-top-width:3px}
.legend i.d{border-top-style:dashed;border-top-color:var(--muted)}
.legend i.b{height:10px;border:0;background:var(--band);width:18px}
.grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(240px,1fr));gap:14px 26px;margin:20px 0}
label{display:flex;justify-content:space-between;font-weight:500;font-size:.95rem}
label output{font-variant-numeric:tabular-nums;color:var(--accent);font-weight:600}
input[type=range]{width:100%;accent-color:var(--accent);margin:6px 0 0}
small{display:block;color:var(--muted);font-size:.8rem;margin-top:2px}
.row{display:flex;flex-wrap:wrap;gap:10px;margin:6px 0 4px}
button{font:inherit;font-weight:500;border-radius:6px;border:1px solid var(--line);background:var(--panel);color:var(--ink);padding:9px 14px;cursor:pointer;min-height:44px}
button.go{background:var(--accent);border-color:var(--accent);color:#fff}
:root[data-theme="dark"] button.go{color:#08161d}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]) button.go{color:#08161d}}
button:focus-visible,input:focus-visible{outline:3px solid var(--accent);outline-offset:2px}
.stats{display:grid;grid-template-columns:repeat(auto-fit,minmax(200px,1fr));gap:12px;margin:18px 0 0}
.stat{background:var(--panel);border:1px solid var(--line);border-radius:8px;padding:12px 14px}
.stat b{display:block;font:600 1.5rem/1.2 "Newsreader",Georgia,serif;font-variant-numeric:tabular-nums}
.stat span{font-size:.85rem;color:var(--muted)}
.notes{max-width:68ch}
.notes p{margin:0 0 12px}
footer{margin-top:40px;color:var(--muted);font-size:.85rem;max-width:68ch}
</style>
</head>
<body>
<main>
  <h1>Vasicek interest rate simulator</h1>
  <p class="lede">Interest rates drift back towards a long-run level but get pushed around by random shocks. Change the three settings below and watch how simulated rate paths respond.</p>
  <div class="eq" aria-label="Vasicek model equation">dr = a(b − r)dt + σ dW</div>

  <div class="chart">
    <canvas id="c" role="img" aria-label="Chart of simulated interest rate paths over time with the average path and a 95% range"></canvas>
    <div class="legend">
      <span><i class="p"></i>Simulated paths</span>
      <span><i></i>Average path</span>
      <span><i class="b"></i>95% range</span>
      <span><i class="d"></i>Long-run level</span>
    </div>
  </div>

  <div class="grid">
    <div><label for="a">Reversion speed (a) <output id="oa"></output></label><input id="a" type="range" min="0.05" max="3" step="0.05" value="0.6"><small>How fast rates are pulled back to the long-run level.</small></div>
    <div><label for="b">Long-run rate (b) <output id="ob"></output></label><input id="b" type="range" min="0" max="8" step="0.1" value="3.5"><small>The level rates drift towards, in %.</small></div>
    <div><label for="s">Volatility (σ) <output id="os"></output></label><input id="s" type="range" min="0.1" max="3" step="0.1" value="1"><small>Size of random shocks, in % per √year.</small></div>
    <div><label for="r0">Starting rate (r₀) <output id="or0"></output></label><input id="r0" type="range" min="-1" max="10" step="0.1" value="5"><small>Where the rate begins, in %.</small></div>
    <div><label for="T">Horizon <output id="oT"></output></label><input id="T" type="range" min="1" max="20" step="1" value="10"><small>Years simulated.</small></div>
    <div><label for="N">Number of paths <output id="oN"></output></label><input id="N" type="range" min="5" max="150" step="5" value="40"><small>More paths show the spread of outcomes.</small></div>
  </div>
  <div class="row">
    <button class="go" id="new">Simulate new paths</button>
    <button data-p="0.6,3.5,1,5">Default</button>
    <button data-p="2.5,3.5,0.8,6">Fast reversion</button>
    <button data-p="0.1,3.5,1.2,6">Slow reversion</button>
    <button data-p="1,3,2.5,3">High volatility</button>
  </div>

  <div class="stats" aria-live="polite">
    <div class="stat"><b id="sHalf"></b><span>Half-life: time for the gap to the long-run level to halve</span></div>
    <div class="stat"><b id="sMean"></b><span>Expected rate at the horizon</span></div>
    <div class="stat"><b id="sRange"></b><span>95% range at the horizon</span></div>
    <div class="stat"><b id="sNeg"></b><span>Chance the rate is below zero at the horizon</span></div>
  </div>

  <h2>How to read it</h2>
  <div class="notes">
    <p><strong>Reversion speed.</strong> A high value pulls rates back to the long-run level quickly, so paths stay close together. A low value lets rates wander for years before they return.</p>
    <p><strong>Volatility.</strong> Larger shocks widen the 95% range. Over the long run the spread settles at σ divided by √(2a), so faster reversion also narrows it.</p>
    <p><strong>A known limitation.</strong> The model can produce negative rates, which was once seen as a flaw and is now a real feature of some markets. Later models such as Cox–Ingersoll–Ross avoid it.</p>
  </div>
  <footer>Built by Euan D’Costa as a companion to an Excel project that simulated Vasicek rate paths using Bank of England data. This is an educational illustration. It uses the model’s exact weekly update, and the numbers shown are not forecasts.</footer>
</main>
<script>
(function(){
  var $=function(id){return document.getElementById(id)};
  var ids=["a","b","s","r0","T","N"];
  var fmt={a:function(v){return v.toFixed(2)},b:function(v){return v.toFixed(1)+"%"},s:function(v){return v.toFixed(1)+"%"},r0:function(v){return v.toFixed(1)+"%"},T:function(v){return v+" yrs"},N:function(v){return v}};
  var seed=7,cv=$("c"),ctx=cv.getContext("2d"),paths=[];
  function rng(s){return function(){s|=0;s=s+0x6D2B79F5|0;var t=Math.imul(s^s>>>15,1|s);t=t+Math.imul(t^t>>>7,61|t)^t;return((t^t>>>14)>>>0)/4294967296}}
  function norm(r){var u=1-r(),v=r();return Math.sqrt(-2*Math.log(u))*Math.cos(2*Math.PI*v)}
  function cdf(x){var t=1/(1+.2316419*Math.abs(x)),d=.3989423*Math.exp(-x*x/2),p=d*t*(.3193815+t*(-.3565638+t*(1.781478+t*(-1.821256+t*1.330274))));return x>0?1-p:p}
  function val(id){return parseFloat($(id).value)}
  function css(n){return getComputedStyle(document.documentElement).getPropertyValue(n).trim()}
  function simulate(){
    var a=val("a"),b=val("b"),s=val("s"),r0=val("r0"),T=val("T"),N=val("N"),dt=1/52,n=Math.round(T*52),r=rng(seed);
    var e=Math.exp(-a*dt),sd=s*Math.sqrt((1-Math.exp(-2*a*dt))/(2*a));
    paths=[];
    for(var i=0;i<N;i++){var p=new Float32Array(n+1);p[0]=r0;for(var k=1;k<=n;k++)p[k]=p[k-1]*e+b*(1-e)+sd*norm(r);paths.push(p)}
  }
  function moments(t){var a=val("a"),b=val("b"),s=val("s"),r0=val("r0");return{m:b+(r0-b)*Math.exp(-a*t),sd:Math.sqrt(s*s/(2*a)*(1-Math.exp(-2*a*t)))}}
  function draw(){
    var w=cv.clientWidth,h=cv.clientHeight,dpr=window.devicePixelRatio||1;
    cv.width=w*dpr;cv.height=h*dpr;ctx.setTransform(dpr,0,0,dpr,0,0);ctx.clearRect(0,0,w,h);
    var T=val("T"),b=val("b"),n=Math.round(T*52),L=44,R=10,Tp=10,B=26,pw=w-L-R,ph=h-Tp-B;
    var lo=Infinity,hi=-Infinity,i,k,mm;
    for(i=0;i<paths.length;i++)for(k=0;k<=n;k+=4){var v=paths[i][k];if(v<lo)lo=v;if(v>hi)hi=v}
    for(k=0;k<=T;k+=.5){mm=moments(k);lo=Math.min(lo,mm.m-1.96*mm.sd);hi=Math.max(hi,mm.m+1.96*mm.sd)}
    lo=Math.min(lo,0);var pad=(hi-lo)*.06;lo-=pad;hi+=pad;
    var X=function(t){return L+t/T*pw},Y=function(v){return Tp+(hi-v)/(hi-lo)*ph};
    ctx.font="12px 'Public Sans',system-ui,sans-serif";ctx.fillStyle=css("--muted");ctx.strokeStyle=css("--line");ctx.lineWidth=1;
    var step=[.5,1,2,5,10].find(function(s){return (hi-lo)/s<=7})||10;
    ctx.textAlign="right";ctx.textBaseline="middle";
    for(var g=Math.ceil(lo/step)*step;g<=hi;g+=step){var y=Y(g);ctx.beginPath();ctx.moveTo(L,y);ctx.lineTo(w-R,y);ctx.stroke();ctx.fillText(g.toFixed(step<1?1:0)+"%",L-6,y)}
    ctx.textAlign="center";ctx.textBaseline="top";
    var ts=T<=5?1:T<=10?2:5;for(var t=0;t<=T;t+=ts)ctx.fillText(t+"y",X(t),h-B+8);
    ctx.strokeStyle=css("--muted");ctx.lineWidth=1.2;ctx.beginPath();ctx.moveTo(L,Y(0));ctx.lineTo(w-R,Y(0));ctx.stroke();
    ctx.fillStyle=css("--band");ctx.beginPath();
    for(k=0;k<=n;k+=2){mm=moments(k/52);ctx.lineTo(X(k/52),Y(mm.m+1.96*mm.sd))}
    for(k=n;k>=0;k-=2){mm=moments(k/52);ctx.lineTo(X(k/52),Y(mm.m-1.96*mm.sd))}
    ctx.closePath();ctx.fill();
    ctx.strokeStyle=css("--path");ctx.lineWidth=1;
    for(i=0;i<paths.length;i++){ctx.beginPath();for(k=0;k<=n;k++){var px=X(k/52),py=Y(paths[i][k]);k?ctx.lineTo(px,py):ctx.moveTo(px,py)}ctx.stroke()}
    ctx.setLineDash([6,5]);ctx.strokeStyle=css("--muted");ctx.beginPath();ctx.moveTo(L,Y(b));ctx.lineTo(w-R,Y(b));ctx.stroke();ctx.setLineDash([]);
    ctx.strokeStyle=css("--accent");ctx.lineWidth=2.5;ctx.beginPath();
    for(k=0;k<=n;k+=2){mm=moments(k/52);var qx=X(k/52),qy=Y(mm.m);k?ctx.lineTo(qx,qy):ctx.moveTo(qx,qy)}ctx.stroke();
  }
  function stats(){
    var a=val("a"),T=val("T"),mm=moments(T);
    var hl=Math.log(2)/a;
    $("sHalf").textContent=(hl<1?(hl*12).toFixed(1)+" months":hl.toFixed(1)+" years");
    $("sMean").textContent=mm.m.toFixed(2)+"%";
    $("sRange").textContent=(mm.m-1.96*mm.sd).toFixed(1)+"% to "+(mm.m+1.96*mm.sd).toFixed(1)+"%";
    var pn=cdf((0-mm.m)/mm.sd)*100;
    $("sNeg").textContent=pn<0.1?"<0.1%":pn.toFixed(1)+"%";
  }
  function labels(){ids.forEach(function(id){$("o"+id).textContent=fmt[id](val(id))})}
  function update(resim){labels();if(resim)simulate();stats();draw()}
  ids.forEach(function(id){$(id).addEventListener("input",function(){update(true)})});
  $("new").addEventListener("click",function(){seed=(seed*1664525+1013904223)>>>0;update(true)});
  Array.prototype.forEach.call(document.querySelectorAll("button[data-p]"),function(btn){
    btn.addEventListener("click",function(){var v=btn.getAttribute("data-p").split(",");["a","b","s","r0"].forEach(function(id,i){$(id).value=v[i]});update(true)})});
  if(window.ResizeObserver)new ResizeObserver(function(){draw()}).observe(cv);else window.addEventListener("resize",draw);
  if(window.matchMedia)window.matchMedia("(prefers-color-scheme: dark)").addEventListener("change",draw);
  update(true);
})();
</script>
</body>
</html>

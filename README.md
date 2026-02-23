<!DOCTYPE html>
<html lang="fa">
<head>
<meta charset="UTF-8">
<title>📊 Voyager – BER vs SNR</title>
<style>
html,body{margin:0;overflow:hidden;background:#000015;font-family:system-ui}
canvas{display:block}
.hud{
 position:absolute;left:16px;top:16px;
 padding:14px 18px;border-radius:14px;
 background:rgba(255,255,255,0.06);
 backdrop-filter:blur(10px);
 color:#d9f3ff;font-size:13px;min-width:460px;
}
.good{color:#9ff0ff}
.mid{color:#ffd29f}
.bad{color:#ff9f9f}
.warn{color:#ffb366}
</style>
</head>
<body>

<canvas id="c"></canvas>
<div class="hud" id="hud"></div>

<script>
const c=document.getElementById("c");
const ctx=c.getContext("2d");
let w,h;
function resize(){w=c.width=innerWidth;h=c.height=innerHeight}
resize();addEventListener("resize",resize);

const AU=65;
const sun={x:w/2,y:h/2};
const earth={a:AU*1,angle:0,r:6};
const heliopause=120;

const ship={
 distance:AU*1,
 angle:0.4,
 velocity:0.22
};

const txPower=220;

// Error function approximation
function erfc(x){
 // Numerical approximation
 const z=Math.abs(x);
 const t=1/(1+0.5*z);
 const r=t*Math.exp(-z*z-1.26551223+
   t*(1.00002368+
   t*(0.37409196+
   t*(0.09678418+
   t*(-0.18628806+
   t*(0.27886807+
   t*(-1.13520398+
   t*(1.48851587+
   t*(-0.82215223+
   t*0.17087277)))))))));
 return x>=0 ? r : 2-r;
}

// BPSK BER
function computeBER(snr){
 return 0.5*erfc(Math.sqrt(snr));
}

function pos(a,ang){
 return {x:sun.x+Math.cos(ang)*a,y:sun.y+Math.sin(ang)*a};
}

function link(distAU){
 let signal=txPower/(distAU*distAU*160);
 let noise=0.12+Math.random()*0.05;
 if(distAU>heliopause) noise+=0.3;
 return signal/noise;
}

function update(){
 earth.angle+=0.0012;
 ship.distance+=ship.velocity;
 ship.angle+=0.002;
}

function draw(){
 ctx.fillStyle="rgba(0,0,20,0.4)";
 ctx.fillRect(0,0,w,h);

 update();

 const pe=pos(earth.a,earth.angle);
 const pship={
  x:sun.x+Math.cos(ship.angle)*ship.distance,
  y:sun.y+Math.sin(ship.angle)*ship.distance
 };

 // Sun
 ctx.fillStyle="#ffcc66";
 ctx.beginPath();ctx.arc(sun.x,sun.y,12,0,Math.PI*2);ctx.fill();

 // Earth
 ctx.fillStyle="#2a7fff";
 ctx.beginPath();ctx.arc(pe.x,pe.y,6,0,Math.PI*2);ctx.fill();

 // Ship
 ctx.fillStyle="#fff";
 ctx.beginPath();ctx.arc(pship.x,pship.y,4,0,Math.PI*2);ctx.fill();

 const distAU=Math.hypot(pship.x-pe.x,pship.y-pe.y)/AU;
 const snr=link(distAU);
 const ber=computeBER(snr);

 // Packet loss estimate
 const packetLoss=Math.min(1, ber*1000);

 // Effective data rate (kbps)
 const baseRate=160; 
 const effectiveRate=baseRate*(1-packetLoss);

 let cls=ber<1e-5?"good":ber<1e-3?"mid":"bad";

 ctx.strokeStyle=cls==="good"?"rgba(120,220,255,0.5)":
                 cls==="mid" ?"rgba(255,200,120,0.5)":
                              "rgba(255,120,120,0.5)";
 ctx.beginPath();
 ctx.moveTo(pe.x,pe.y);
 ctx.lineTo(pship.x,pship.y);
 ctx.stroke();

 document.getElementById("hud").innerHTML=`
📊 BER Simulation (BPSK)<br>
📏 Distance: ${distAU.toFixed(1)} AU<br>
📶 SNR: ${snr.toFixed(2)}<br>
📉 BER: <span class="${cls}">${ber.toExponential(2)}</span><br>
📦 Packet Loss: ${(packetLoss*100).toFixed(2)} %<br>
🚀 Effective Data Rate: ${effectiveRate.toFixed(1)} kbps<br>
🕒 Latency: ${(distAU*8.3/60).toFixed(2)} h
`;

 requestAnimationFrame(draw);
}

draw();
</script>
</body>
</html>

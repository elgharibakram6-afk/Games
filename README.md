<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>Akra Games 500+</title>
<meta name="viewport" content="width=device-width, initial-scale=1">
<style>
body{margin:0;font-family:Arial;background:#0b0f19;color:#fff}
#search{margin:15px;padding:8px;width:calc(100% - 40px);border-radius:6px;border:none}
#games{display:grid;grid-template-columns:repeat(auto-fill,minmax(140px,1fr));gap:15px;padding:20px}
.game{background:#11162a;border-radius:10px;padding:8px;text-align:center;position:relative;height:220px;transition:.3s}
.game:hover{transform:scale(1.05)}
.game img{width:120px;height:80px;border-radius:6px}
.locked::after{content:"🔒";position:absolute;top:5px;right:5px}
.jugado::after{content:"✔";position:absolute;top:5px;left:5px;color:#2ecc71}
.premium{border:2px solid gold}
.premium::before{content:"🔥";position:absolute;top:-8px;left:-8px}
button{margin-top:4px;padding:4px 8px;border:0;border-radius:4px;background:#3498db;color:#fff;cursor:pointer;font-size:12px}
#adminBtn{position:fixed;top:10px;left:10px;background:crimson;padding:6px 10px;border-radius:6px}
#adminPanel{display:none;position:fixed;top:50px;right:10px;background:#1b1f3b;padding:15px;border-radius:10px;width:260px}
.sub{font-size:11px;color:#f1c40f}
</style>
</head>
<body>

<button id="adminBtn" onclick="loginAdmin()">ADMIN</button>
<input id="search" placeholder="Buscar juego..." oninput="filtrar()">
<div id="games"></div>

<div id="adminPanel">
<h3>ADMIN</h3>
<button onclick="adminAll()">Desbloquear TODO</button>
<button onclick="darMedium()">Medium permanente</button>
<button onclick="darPremium()">Premium permanente</button>
<button onclick="expulsar()">Expulsar usuarios</button>
</div>

<script>
let admin=false;
let filtro="";
let progreso=JSON.parse(localStorage.getItem("progreso"))||{};
let subs=JSON.parse(localStorage.getItem("subs"))||{};
let permanente=JSON.parse(localStorage.getItem("perm"))||{};

let juegos=[];
for(let i=1;i<=100;i++) juegos.push({n:`Basic ${i}`,t:"basic"});
for(let i=1;i<=150;i++) juegos.push({n:`Medium ${i}`,t:"medium"});
for(let i=1;i<=250;i++) juegos.push({n:`Premium ${i}`,t:"premium"});

function tiempo(ms){
 if(ms<=0) return "Caducado";
 let d=Math.floor(ms/86400000);
 return d+" días";
}

function mostrar(){
 let c=document.getElementById("games");
 c.innerHTML="";
 let now=Date.now();
 juegos.filter(j=>j.n.toLowerCase().includes(filtro)).forEach(j=>{
  let activo = permanente[j.t] || (subs[j.t] && subs[j.t]>now);
  let locked = (j.t!="basic" && !admin && !activo);
  c.innerHTML+=`
  <div class="game ${locked?"locked":""} ${j.t=="premium"?"premium":""} ${progreso[j.n]?"jugado":""}">
   <img src="https://via.placeholder.com/120x80">
   <h4>${j.n}</h4>
   <button onclick="${locked?"alert('Bloqueado')":"jugar('"+j.n+"')"}">
    ${locked?"Bloqueado":"Jugar"}
   </button>
   ${!admin && j.t!="basic"?`<button onclick="sub('${j.t}')">Comprar 2 meses</button>`:""}
   ${subs[j.t]&&!permanente[j.t]?`<div class="sub">⏳ ${tiempo(subs[j.t]-now)}</div>`:""}
  </div>`;
 });
}

function jugar(n){
 progreso[n]=true;
 localStorage.setItem("progreso",JSON.stringify(progreso));
 mostrar();
}

function sub(t){
 let add=60*24*60*60*1000;
 subs[t]=(subs[t]&&subs[t]>Date.now()?subs[t]:Date.now())+add;
 localStorage.setItem("subs",JSON.stringify(subs));
 mostrar();
}

function filtrar(){
 filtro=document.getElementById("search").value.toLowerCase();
 mostrar();
}

function loginAdmin(){
 let p=prompt("Contraseña admin");
 if(p==="akraM2011"){
  admin=true;
  document.getElementById("adminPanel").style.display="block";
  mostrar();
 }else alert("Incorrecta");
}

function adminAll(){
 permanente.medium=true;
 permanente.premium=true;
 localStorage.setItem("perm",JSON.stringify(permanente));
 mostrar();
}

function darMedium(){
 permanente.medium=true;
 localStorage.setItem("perm",JSON.stringify(permanente));
 mostrar();
}

function darPremium(){
 permanente.premium=true;
 localStorage.setItem("perm",JSON.stringify(permanente));
 mostrar();
}

function expulsar(){
 localStorage.clear();
 location.reload();
}

mostrar();
</script>
</body>
</html>

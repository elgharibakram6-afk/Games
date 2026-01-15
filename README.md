# Games
const fs = require('fs'), archiver = require('archiver'), path = require('path');
const carpeta = path.join(__dirname,'akra-games');
if(!fs.existsSync(carpeta)) fs.mkdirSync(carpeta);

const archivos = {
"index.html":`<!DOCTYPE html><html lang="es"><head><meta charset="UTF-8"><title>Akra Games</title><meta name="viewport" content="width=device-width,initial-scale=1"><link rel="manifest" href="manifest.json"><style>body{margin:0;background:#0b0f19;color:#fff;font-family:Arial}#loginScreen{padding:50px;text-align:center}button{padding:6px 10px;border:0;border-radius:6px;cursor:pointer;margin:3px}#adminPanel{display:none;position:fixed;top:70px;right:15px;width:280px;background:#1b1f3b;padding:10px;border-radius:8px;overflow:auto;max-height:90vh}</style></head><body>
<div id="loginScreen"><h2>🎮 Bienvenido a Akra Games</h2><input id="usernameInput" placeholder="Nombre"/><button onclick="entrar()">Entrar</button></div>
<button id="adminBtn" style="display:none">Admin</button>
<div id="games"></div>
<div id="adminPanel">
<h3>Admin</h3>
<button onclick="verRankingGlobal()">Ranking</button>
<button onclick="verUsuariosLogros()">Logros</button>
<button onclick="expulsarJugador()">Expulsar</button>
<button onclick="desbloquearTodos()">Desbloquear</button>
<button onclick="activarPremium()">Premium ON</button>
<button onclick="desactivarPremium()">Premium OFF</button>
<button onclick="resetear()">Resetear</button>
<input id="buscarJuego" placeholder="Buscar..." onkeyup="filtrarJuegos()"/>
<div id="chatAdmin"></div><input id="chatAdminInput" placeholder="Mensaje" onkeydown="if(event.key==='Enter')enviarChatAdmin()">
</div>
<script src="https://cdn.socket.io/4.7.2/socket.io.min.js"></script>
<script src="main.js"></script>
</body></html>`,

"main.js":`// main.js simplificado (elimina referencias a monedas y niveles)`,

"server.js":`const express=require("express"),app=express(),http=require("http").createServer(app),io=require("socket.io")(http),cors=require("cors");
app.use(cors());app.use(express.json());app.use(express.static("public"));
const ADMIN_PASS="akraM2011";
let usuariosPremium={},globalRanking={},expulsados={},chatMessages=[];
app.post("/admin/login",(r,s)=>{r.body.pass===ADMIN_PASS?s.json({ok:true}):s.status(401).json({ok:false})});
app.post("/admin/desbloquear",(r,s)=>{const u=r.body.user;if(!u)return s.status(400).json({ok:false});usuariosPremium[u]=Date.now()+60*24*60*60*1000;s.json({ok:true})});
app.post("/admin/expulsar",(r,s)=>{const u=r.body.user;expulsados[u]=true;for(let [id,sk] of io.sockets.sockets)if(sk.userId===u){sk.emit("expulsado","Expulsado");sk.disconnect()}s.json({ok:true})});
app.post("/ranking/update",(r,s)=>{const {user,juegos,horas}=r.body;globalRanking[user]={juegos,horas};s.json({ok:true})});
app.get("/ranking",(r,s)=>{const ranking=Object.entries(globalRanking).sort((a,b)=>b[1].juegos-a[1].juegos||b[1].horas-a[1].horas).map(([user,d])=>({user,...d}));s.json(ranking)});
io.on("connection",s=>{s.on("setUser",u=>{s.userId=u;if(expulsados[u]){s.emit("expulsado","Expulsado");s.disconnect()}});s.on("chat",m=>{chatMessages.push(m);io.emit("chat",m)});s.on("playerMove",d=>{s.broadcast.emit("playerMove",d)})});
http.listen(process.env.PORT||3000,()=>console.log("Servidor activo"));`,

"package.json":`{"name":"akra-games","version":"1.0.0","main":"server.js","scripts":{"start":"node server.js"},"dependencies":{"express":"^4.18.2","socket.io":"^4.7.2","cors":"^2.8.5"}}`,

"manifest.json":`{"name":"Akra Games","short_name":"AkraGames","start_url":"./index.html","display":"standalone","background_color":"#0b0f19","theme_color":"#3498db","icons":[{"src":"icon-192.png","sizes":"192x192","type":"image/png"},{"src":"icon-512.png","sizes":"512x512","type":"image/png"}]}`,

"sw.js":`self.addEventListener('install',e=>{console.log('Service Worker instalado')});self.addEventListener('fetch',e=>{e.respondWith(fetch(e.request))});`
};

for(let n in archivos)fs.writeFileSync(path.join(carpeta,n),archivos[n]);

const output=fs.createWriteStream(path.join(__dirname,'akra-games.zip')),archive=archiver('zip',{zlib:{level:9}});
archive.pipe(output);archive.directory(carpeta+'/',false);archive.finalize();

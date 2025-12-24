<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Orkut Clássico Definitivo</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{margin:0;font-family:Arial;background:#d0e6ff}
header{background:#3b5998;color:#fff;padding:15px;text-align:center;font-size:22px}
#login,#app{display:none;padding:15px}
.card{background:#fff;border-radius:8px;padding:10px;margin-bottom:10px}
#layout{display:flex;gap:10px}
#users{width:30%;max-height:80vh;overflow:auto}
#chatArea{width:70%}
.msg{background:#f1f1f1;margin:5px 0;padding:5px;border-radius:5px}
.me{background:#d1e7ff}
button{background:#3b5998;color:#fff;border:none;padding:8px;border-radius:5px;width:100%}
input,textarea{width:100%;padding:8px;margin:5px 0}
.user{padding:5px;border-bottom:1px solid #ccc;cursor:pointer}
.user:hover{background:#eee}
img,video{max-width:200px;border-radius:5px;margin-top:5px}
</style>
</head>

<body>

<header>💙 Orkut Clássico Definitivo</header>

<div id="login" class="card">
  <h3>Digite seu nome</h3>
  <input id="nome">
  <button onclick="entrar()">Entrar</button>
</div>

<div id="app">
  <div id="layout">
    <div id="users" class="card">
      <h3>Online</h3>
      <div id="listaUsers"></div>
    </div>

    <div id="chatArea" class="card">
      <h3 id="chatCom">Selecione alguém</h3>
      <div id="msgs"></div>
      <textarea id="texto" placeholder="Mensagem"></textarea>
      <input type="file" id="file">
      <button onclick="enviar()">Enviar</button>
    </div>
  </div>
</div>

<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-storage-compat.js"></script>

<script>
firebase.initializeApp({
 apiKey: "AIzaSyCUb2VNq9AETlzhE4w0eSwRPA4yBzrVC68",
 authDomain: "meu-50495.firebaseapp.com",
 projectId: "meu-50495",
 storageBucket: "meu-50495.firebasestorage.app",
 messagingSenderId: "941036013080",
 appId: "1:941036013080:web:61281776f379d87867a56d"
});

const db=firebase.firestore();
const storage=firebase.storage();

let me=null;
let chatUser=null;

login.style.display="block";

function entrar(){
  if(!nome.value)return;
  me=Date.now().toString();
  db.collection("online").doc(me).set({nome:nome.value});
  login.style.display="none";
  app.style.display="block";
  listar();
}

window.onbeforeunload=()=>db.collection("online").doc(me).delete();

function listar(){
  db.collection("online").onSnapshot(s=>{
    listaUsers.innerHTML="";
    s.forEach(d=>{
      if(d.id!==me){
        let div=document.createElement("div");
        div.className="user";
        div.innerText=d.data().nome;
        div.onclick=()=>abrirChat(d.id,d.data().nome);
        listaUsers.appendChild(div);
      }
    });
  });
}

function abrirChat(id,nome){
  chatUser=id;
  chatCom.innerText="Chat com "+nome;
  carregar();
}

function carregar(){
  db.collection("msgs").orderBy("t").onSnapshot(s=>{
    msgs.innerHTML="";
    s.forEach(d=>{
      let m=d.data();
      if((m.from===me && m.to===chatUser)||(m.from===chatUser && m.to===me)){
        if(!m.del?.includes(me)){
          let div=document.createElement("div");
          div.className="msg "+(m.from===me?"me":"");
          if(m.text)div.innerHTML+=m.text;
          if(m.media){
            if(m.type.startsWith("video")){
              div.innerHTML+=`<video controls src="${m.media}"></video>`;
            }else{
              div.innerHTML+=`<img src="${m.media}">`;
            }
          }
          div.onclick=()=>db.collection("msgs").doc(d.id).update({
            del:firebase.firestore.FieldValue.arrayUnion(me)
          });
          msgs.appendChild(div);
        }
      }
    });
  });
}

function enviar(){
  if(!chatUser)return;
  let f=file.files[0];
  if(f){
    let ref=storage.ref("midia/"+Date.now());
    ref.put(f).then(r=>r.ref.getDownloadURL()).then(url=>{
      salvar({media:url,type:f.type});
    });
  }else if(texto.value){
    salvar({text:texto.value});
  }
  texto.value=""; file.value="";
}

function salvar(obj){
  db.collection("msgs").add({
    from:me,
    to:chatUser,
    ...obj,
    t:Date.now(),
    del:[]
  });
}
</script>

</body>
</html>
 # Orkut.cr

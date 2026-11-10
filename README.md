
<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>NextGen Social</title>

<style>
*{margin:0;padding:0;box-sizing:border-box}
body{background:#101117;color:#fff;font-family:Arial,sans-serif}
header{height:70px;background:#191b25;display:flex;align-items:center;justify-content:space-between;padding:0 5%;border-bottom:1px solid #303342;position:sticky;top:0;z-index:5}
.logo{font-size:23px;font-weight:bold}.logo span{color:#956cff}
input,textarea{background:#292c3a;color:white;border:1px solid #3b3e50;border-radius:10px;padding:12px}
button{cursor:pointer;border:0;border-radius:9px;padding:11px 16px;background:#8758ed;color:white;font-weight:bold}
button:hover{filter:brightness(1.2)}
.layout{max-width:1250px;margin:30px auto;padding:0 15px;display:grid;grid-template-columns:220px 1fr 250px;gap:25px}
.sidebar,.right,.main{background:#191b25;border-radius:15px;padding:20px;height:max-content}
.sidebar button{display:block;width:100%;text-align:left;margin-bottom:12px;background:#252735}
h1{font-size:25px;margin-bottom:8px}
.muted{color:#999;font-size:13px}
.create{display:flex;gap:10px;margin:20px 0}
.create input{flex:1;min-width:0}
.post{background:#252735;padding:18px;border-radius:13px;margin:15px 0}
.post-head{display:flex;align-items:center;justify-content:space-between;margin-bottom:15px}
.post p{white-space:pre-wrap;overflow-wrap:anywhere;line-height:1.6}
.post-actions{display:flex;gap:10px;margin-top:15px}
.like{background:#39314f}
.avatar{width:38px;height:38px;border-radius:50%;background:#8758ed;display:inline-grid;place-items:center;font-weight:bold}
.modal{display:none;position:fixed;inset:0;background:#000c;z-index:10;align-items:center;justify-content:center;padding:15px}
.modal-content{background:#191b25;padding:25px;border-radius:15px;width:100%;max-width:450px}
.modal-content input,.modal-content textarea{width:100%;margin:12px 0}
.modal-content textarea{height:120px}
.close{background:#444;margin-left:5px}
.trend{padding:12px 0;border-bottom:1px solid #303342;color:#b49aff}
.profile{text-align:center;padding:15px}
.profile .avatar{width:65px;height:65px;font-size:24px;margin-bottom:10px}
.mobile{display:none}
@media(max-width:900px){.layout{grid-template-columns:180px 1fr}.right{display:none}}
@media(max-width:650px){
header{padding:0 12px}.logo{font-size:17px}
.layout{display:block;margin:15px auto;padding:0 10px}
.sidebar{display:flex;overflow:auto;gap:5px;margin-bottom:15px;padding:10px}
.sidebar button{white-space:nowrap;width:auto;margin:0;font-size:12px}
.main{padding:14px}.create input{width:50%}
.mobile{display:flex;position:fixed;bottom:0;left:0;right:0;background:#191b25;padding:10px;justify-content:space-around;z-index:5;border-top:1px solid #303342}
.mobile button{font-size:12px;padding:10px}
body{padding-bottom:65px}
}
</style>
</head>

<body>

<header>
<div class="logo">NextGen <span>Social</span></div>
<input id="search" placeholder="Ara..." oninput="render()">
<button onclick="openProfile()">Profil</button>
</header>

<div class="layout">

<aside class="sidebar">
<button onclick="setPage('home')">⌂ Ana Sayfa</button>
<button onclick="setPage('explore')">⌕ Keşfet</button>
<button onclick="setPage('following')">♡ Takip Ettiklerim</button>
<button onclick="setPage('profile')">◉ Profilim</button>
<button onclick="openPost()">＋ Gönderi Oluştur</button>
</aside>

<main class="main">

<h1 id="pageTitle">NextGen Social</h1>
<p class="muted">Kendi sosyal ağın, kendi topluluğun.</p>

<div class="create">
<input id="quickPost" placeholder="Ne paylaşmak istersin?">
<button onclick="quickPublish()">Paylaş</button>
</div>

<div id="feed"></div>

</main>

<aside class="right">
<div class="profile">
<div class="avatar" id="profileAvatar">N</div>
<h3 id="profileName">NextGen Üyesi</h3>
<p class="muted" id="profileBio">NextGen Social'a hoş geldin!</p>
<br>
<button onclick="openProfile()">Profili Düzenle</button>
</div>
<hr style="border-color:#303342;margin:15px 0">
<h3>Keşfet</h3>
<div class="trend">#NextGen</div>
<div class="trend">#YeniBaşlangıç</div>
<div class="trend">#SosyalMedya</div>
<div class="trend">#GünlükPaylaşım</div>
</aside>

</div>

<div class="mobile">
<button onclick="setPage('home')">Ana Sayfa</button>
<button onclick="setPage('explore')">Keşfet</button>
<button onclick="openPost()">＋ Paylaş</button>
<button onclick="setPage('profile')">Profil</button>
</div>

<div class="modal" id="postModal">
<div class="modal-content">
<h2>Yeni Gönderi</h2>
<textarea id="postText" placeholder="Düşüncelerini paylaş..."></textarea>
<button onclick="publish()">Paylaş</button>
<button class="close" onclick="closePost()">Kapat</button>
</div>
</div>

<div class="modal" id="profileModal">
<div class="modal-content">
<h2>Profilini Düzenle</h2>
<input id="nameInput" placeholder="Adın">
<textarea id="bioInput" placeholder="Kendinden bahset"></textarea>
<button onclick="saveProfile()">Kaydet</button>
<button class="close" onclick="closeProfile()">Kapat</button>
</div>
</div>

<script>
let posts=JSON.parse(localStorage.getItem("ng_posts")||"[]");
let user=JSON.parse(localStorage.getItem("ng_user")||'{"name":"NextGen Üyesi","bio":"NextGen Social kullanıcısı"}');
let page="home";

function save(){
localStorage.setItem("ng_posts",JSON.stringify(posts));
localStorage.setItem("ng_user",JSON.stringify(user));
}

function escapeHTML(text){
let div=document.createElement("div");
div.textContent=text;
return div.innerHTML;
}

function openPost(){
document.getElementById("postModal").style.display="flex";
}

function closePost(){
document.getElementById("postModal").style.display="none";
}

function publish(){
let text=document.getElementById("postText").value.trim();
if(!text)return alert("Gönderi yazmalısın!");

posts.unshift({
id:Date.now(),
name:user.name,
text:text,
likes:0,
comments:[]
});

save();
document.getElementById("postText").value="";
closePost();
render();
}

function quickPublish(){
document.getElementById("postText").value=document.getElementById("quickPost").value;
document.getElementById("quickPost").value="";
publish();
}

function likePost(id){
let post=posts.find(p=>p.id===id);
if(post){
post.likes++;
save();
render();
}
}

function commentPost(id,input){
let post=posts.find(p=>p.id===id);
let text=input.value.trim();
if(!text)return;
post.comments.push({name:user.name,text:text});
save();
render();
}

function render(){
let feed=document.getElementById("feed");
let search=document.getElementById("search").value.toLocaleLowerCase("tr");
let list=[...posts];

if(page==="profile")list=list.filter(p=>p.name===user.name);
if(search)list=list.filter(p=>(p.text+p.name).toLocaleLowerCase("tr").includes(search));
if(page==="popular")list.sort((a,b)=>b.likes-a.likes);

feed.innerHTML="";

if(!list.length){
feed.innerHTML='<div class="post">Henüz gönderi yok. İlk paylaşımı sen yap!</div>';
}

list.forEach(p=>{
let div=document.createElement("div");
div.className="post";

div.innerHTML=`
<div class="post-head">
<strong>${escapeHTML(p.name)}</strong>
<span class="muted">NextGen üyesi</span>
</div>
<p>${escapeHTML(p.text)}</p>
<div class="post-actions">
<button class="like" onclick="likePost(${p.id})">♥ ${p.likes} Beğeni</button>
<button onclick="this.parentElement.nextElementSibling.hidden=false">Yorumlar (${p.comments.length})</button>
</div>
<div class="comments" hidden>
${p.comments.map(c=>`<p style="margin:10px 0"><b>${escapeHTML(c.name)}:</b> ${escapeHTML(c.text)}</p>`).join("")}
<div style="display:flex;gap:5px;margin-top:10px">
<input placeholder="Yorum yaz..." style="width:75%">
<button class="commentButton">Gönder</button>
</div>
</div>`;

div.querySelector(".commentButton").onclick=function(){
commentPost(p.id,div.querySelector(".comments input"));
};
feed.appendChild(div);
});
}

function setPage(p){
page=p;
document.getElementById("pageTitle").textContent=
p==="home"?"Ana Akış":
p==="explore"?"Keşfet":
p==="following"?"Takip Ettiklerim":"Profilim";

if(p==="following")alert("Takip sistemi için gerçek kullanıcı veritabanı bağlantısı gereklidir.");
render();
}

function openProfile(){
document.getElementById("nameInput").value=user.name;
document.getElementById("bioInput").value=user.bio;
document.getElementById("profileModal").style.display="flex";
}

function closeProfile(){
document.getElementById("profileModal").style.display="none";
}

function saveProfile(){
user.name=document.getElementById("nameInput").value.trim()||"NextGen Üyesi";
user.bio=document.getElementById("bioInput").value.trim();
document.getElementById("profileName").textContent=user.name;
document.getElementById("profileBio").textContent=user.bio;
document.getElementById("profileAvatar").textContent=user.name[0].toUpperCase();
save();
closeProfile();
render();
}

render();
document.getElementById("profileName").textContent=user.name;
document.getElementById("profileBio").textContent=user.bio;
document.getElementById("profileAvatar").textContent=user.name[0].toUpperCase();
</script>

</body>
</html>

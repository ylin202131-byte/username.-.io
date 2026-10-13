<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>幼师打卡话术助手</title>
<style>
*{margin:0;padding:0;box-sizing:border-box;font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,sans-serif;}
body{background:#f6f7f9;color:#333;padding-bottom:80px;}
.tab-bar{position:fixed;bottom:0;left:0;right:0;background:#fff;display:flex;border-top:1px solid #eee;z-index:99;}
.tab-item{flex:1;text-align:center;padding:12px 0;font-size:14px;color:#666;}
.tab-item.active{color:#36b37e;}
.page{display:none;padding:16px;}
.page.active{display:block;}
input{width:100%;padding:12px 14px;border-radius:12px;border:1px solid #eee;background:#fff;font-size:16px;margin-bottom:12px;}
.scroll-tab{display:flex;overflow-x:auto;gap:8px;margin:12px 0;padding-bottom:4px;}
.scroll-tab>span{white-space:nowrap;padding:8px 14px;background:#fff;border-radius:8px;font-size:14px;color:#666;cursor:pointer;}
.scroll-tab>span.active{background:#36b37e;color:#fff;}
.card{background:#fff;border-radius:16px;padding:18px;margin-bottom:14px;}
.card-tip{font-size:12px;color:#888;margin-bottom:8px;}
.card-text{font-size:16px;line-height:1.6;margin-bottom:12px;}
.btn-group{display:flex;gap:8px;flex-wrap:wrap;}
button{padding:8px 12px;border-radius:8px;border:none;font-size:14px;cursor:pointer;}
.btn-primary{background:#36b37e;color:#fff;}
.btn-default{background:#eee;color:#333;}
.empty{text-align:center;padding:40px;color:#999;}
.title{font-size:18px;font-weight:bold;margin:14px 0;}
.stat-row{display:flex;gap:10px;margin:10px 0;}
.check-item{display:flex;align-items:center;gap:8px;padding:10px;background:#fff;border-radius:10px;margin-bottom:8px;}
.toast{position:fixed;top:40px;left:50%;transform:translateX(-50%);background:rgba(0,0,0,0.7);color:#fff;padding:10px 16px;border-radius:8px;display:none;z-index:1000;}
</style>
</head>
<body>
<div id="toast" class="toast"></div>

<!-- 首页 -->
<div class="page active" id="page-index">
  <div class="title">📅 今日专属表扬</div>
  <input id="global-name" placeholder="输入幼儿名字，自动替换XX">
  <div style="margin:10px 0;">
    <button class="btn-primary" onclick="refreshDaily()">🔄 换一换</button>
  </div>
  <div id="daily-container"></div>
  <div class="title">📜历史每日话术</div>
  <div id="history-daily"></div>
</div>

<!-- 话术库 -->
<div class="page" id="page-words">
  <div class="title">📚话术库</div>
  <input id="search-word" placeholder="搜索话术关键词" oninput="filterWords()">
  <div class="scroll-tab" id="words-tab"></div>
  <div id="words-container"></div>
</div>

<!-- AI生成 -->
<div class="page" id="page-ai">
  <div class="title">🤖AI文案生成</div>
  <input id="ai-scene" placeholder="输入场景，例：小班阅读打卡 / 家长担心孩子午睡">
  <div style="margin:10px 0;">
    <button class="btn-primary" onclick="genAi()">生成文案</button>
    <button class="btn-default" onclick="genWholeSet()">✨一键生成整套打卡话术</button>
  </div>
  <div id="ai-result"></div>
</div>

<!-- 家长对话 -->
<div class="page" id="page-parent">
  <div class="title">👨‍👩‍👧‍👦家长对话回复库</div>
  <input id="search-parent" placeholder="搜索：请假、吃饭、投诉、打卡" oninput="filterParent()">
  <div class="scroll-tab" id="parent-tab"></div>
  <div class="scroll-tab" id="style-tab">
    <span class="active" data-tag="全部">全部</span>
    <span data-tag="简短一句话">简短一句话</span>
    <span data-tag="温柔共情版">温柔共情版</span>
    <span data-tag="正式稳重版">正式稳重版</span>
  </div>
  <div id="parent-container"></div>
</div>

<!-- 我的收藏 -->
<div class="page" id="page-collect">
  <div class="title">❤️我的收藏</div>
  <div id="collect-container"></div>
  <div style="margin-top:20px;">
    <div class="title">✏️新增自定义话术</div>
    <input id="custom-text" placeholder="输入自定义话术">
    <button class="btn-primary" onclick="addCustom()">保存</button>
  </div>
</div>

<!-- 打卡台账 -->
<div class="page" id="page-record">
  <div class="title">📋打卡台账</div>
  <input id="student-name" placeholder="添加幼儿名字，回车新增">
  <div id="student-list"></div>
  <div id="record-stat"></div>
  <button class="btn-default" onclick="clearToday()">清空今日记录</button>
</div>

<!--底部Tab-->
<div class="tab-bar">
  <div class="tab-item active" data-page="page-index">首页</div>
  <div class="tab-item" data-page="page-words">话术库</div>
  <div class="tab-item" data-page="page-ai">AI生成</div>
  <div class="tab-item" data-page="page-parent">家长对话</div>
  <div class="tab-item" data-page="page-collect">我的收藏</div>
  <div class="tab-item" data-page="page-record">打卡台账</div>
</div>

<script>
// ========== 全局存储 ==========
const STORAGE_KEYS = {collect:"talkCollect",students:"studentList",record:"recordData"};
// 【打卡表扬话术池】
const praisePool = {
  whole:[
    "✨今日打卡已收到！为坚持打卡的宝贝点赞，每一次小小的分享，都是成长的印记，感谢家长的耐心陪伴，我们继续加油💐",
    "👏为今天积极完成小任务的小朋友喝彩！好习惯贵在日复一日的坚持，愿宝贝们在打卡中收获快乐与进步🌟",
    "💫感谢各位大朋友的配合，看到宝贝认真投入的模样，老师内心满是欢喜。愿我们继续携手，陪伴孩子慢慢成长！",
    "🎉又一批精彩打卡收到！不追求完美，贵在愿意参与，敢于尝试就是最大的进步，宝贝们继续努力👍",
    "🌞打卡不是任务，是见证成长的小窗口。感谢宝贝的努力，感谢家长的陪伴，期待明天更多精彩表现。",
    "💖每一份打卡，都是宝贝努力的证明。为敢于展示自己的小朋友鼓掌，坚持下去，会遇见更棒的自己。",
    "📝今日打卡圆满收官！有的宝贝全程专注投入，有的宝贝慢慢进步，每一份努力都值得被看见，继续保持！",
    "🌟小小打卡，大大成长。表扬积极参与的家庭，愿宝贝把坚持变成习惯，慢慢积攒属于自己的小光芒。",
    "💪不怕做得慢，就怕不坚持。看到宝贝们一天天的变化，老师由衷开心，让我们继续并肩前行。",
    "🌈感谢家长用心记录宝贝的瞬间，每一次打卡都是珍贵回忆。愿宝贝保持这份热情，大胆展现自我。",
    "✨参与即优秀！为今天愿意完成打卡任务的小朋友点赞，点滴积累，终会闪闪发光。",
    "🎈好习惯从一次次打卡开始，感谢大朋友的陪伴引导，宝贝们的进步看得见，继续加油向前冲！",
    "💐很高兴看见宝贝们的打卡分享，认真的模样格外可爱。愿大家保持这份热爱，收获满满的成长。",
    "👍成长没有捷径，贵在日复一日的坚持。表扬今天打卡的宝贝，期待明天更多小朋友加入进来。",
    "☀️每一次打卡，都是一次自我突破。为努力的小朋友点赞，感谢家长一路陪伴，我们明天再见！"
  ],
  single:[
    "🌟XX宝贝太棒啦！认真完成今日打卡，态度特别端正，看得见你的进步，继续闪闪发光！",
    "💯为XX小朋友点赞！愿意认真完成小任务，敢于展示自己，这份勇气值得大大的表扬👏",
    "✨XX宝贝打卡完成！看得出来你很用心，一点一滴都在进步，期待你更多精彩表现。",
    "💖XX宝贝表现超亮眼！认真对待每一次小任务，继续保持，你会越来越优秀。",
    "🎉给XX小朋友点个赞！积极参与打卡，努力的样子特别可爱，继续加油！",
    "👍XX宝贝，你的打卡老师收到啦！愿意坚持就是最大的胜利，继续努力，未来可期。",
    "🌞XX宝贝，从打卡中能看到你的用心，每一次分享都有新收获，为你喝彩。",
    "💫XX小朋友太棒咯！认真投入不敷衍，坚持下去，会收获意想不到的成长。",
    "📖XX宝贝打卡完成！专注的模样真迷人，继续保持这份认真，越来越出色。",
    "💐表扬XX宝贝！积极参与，乐于展示，愿你一直保有这份热情，快乐成长。",
    "🎈XX小朋友，你的努力老师看在眼里，每一次打卡都是一次进步，继续加油！",
    "✨XX宝贝，很棒的打卡！敢于大胆展示自己，这份自信特别珍贵，继续保持。",
    "💪XX宝贝完成今日打卡！不怕挑战，认真对待，为努力的你点赞👍",
    "🌈XX小朋友，打卡完成啦！能感受到你的用心，日积月累，你会越来越厉害。",
    "💖XX宝贝，感谢你的认真分享，每一次坚持，都在悄悄变优秀！"
  ],
  short:[
    "👍XX宝贝真棒，继续坚持！","✨打卡完成，为你点赞！","💯表现优秀，再接再厉！","👏很棒哦，继续加油！",
    "💫为努力的你点赞！","🌟做得很好，保持住！","🎉打卡成功，你超棒！","💖继续保持这份认真！",
    "☀️宝贝表现很不错！","📝点赞你的认真！","💪继续努力，越来越棒！","🎈太棒啦，给你鼓掌👏",
    "✨看得见你的进步！","💐值得大大的表扬！","👍继续加油，期待明天！"
  ]
};
//家长对话话术库
const parentTalkRaw = [
  {cat:"feedback",catName:"日常反馈",style:"温柔共情版",content:"XX妈妈您好～XX今天在园整体状态很不错，和小伙伴互动积极，上课愿意举手表达。如果在家可以多鼓励宝贝大胆说出想法哦😊",tip:"适合私信"},
  {cat:"feedback",catName:"日常反馈",style:"温柔共情版",content:"XX爸爸，跟您简单反馈：今天XX午睡入睡稍慢，但是能安静躺好不吵闹，进步很大，我们会持续留意。",tip:"适合私信"},
  {cat:"feedback",catName:"日常反馈",style:"简短一句话",content:"XX今天主动帮小伙伴整理玩具，责任心特别强，值得表扬👍",tip:"适合私信"},
  {cat:"feedback",catName:"日常反馈",style:"简短一句话",content:"XX今天上课很积极，愿意大胆举手表达自己想法，表达能力进步明显。",tip:"适合私信"},
  {cat:"question",catName:"家长提问",style:"正式稳重版",content:"收到您的问题😊 今天课堂主要练习XX内容，宝贝在课堂上可以跟上节奏，回家简单巩固就好，不用额外增加压力。",tip:"私信/群内"},
  {cat:"question",catName:"家长提问",style:"温柔共情版",content:"您好，孩子之间玩耍偶尔推搡是探索社交的过程，我们现场已经引导，教宝贝用语言表达需求，后续持续关注。",tip:"适合私信"},
  {cat:"leave",catName:"请假缺勤",style:"简短一句话",content:"收到啦，好好在家休息，多喝温水，祝宝贝早日康复，幼儿园这边功课我们之后帮宝贝简单补上🌷",tip:"私信"},
  {cat:"leave",catName:"请假缺勤",style:"温柔共情版",content:"收到请假，宝贝好好休息，康复之后再来幼儿园，打卡任务不用补。",tip:"私信"},
  {cat:"checkin",catName:"打卡沟通",style:"温柔共情版",content:"XX妈妈没关系，今天没时间打卡不用有压力，有空的时候再补也可以，坚持最重要，不强求每天必须完成✅",tip:"私信"},
  {cat:"checkin",catName:"打卡沟通",style:"简短一句话",content:"打卡视频不用很长，1分钟以内记录孩子状态就足够，不用特意准备道具。",tip:"群内回复"},
  {cat:"calm",catName:"安抚情绪",style:"温柔共情版",content:"特别理解您的担心，刚入园/现阶段宝贝需要一点适应时间，我们会多陪伴照顾，有情况及时和您同步。",tip:"适合私信"},
  {cat:"calm",catName:"安抚情绪",style:"温柔共情版",content:"我理解您希望孩子表现优秀的心情，但每个孩子节奏不同，我们多鼓励，少比较。",tip:"适合私信"},
  {cat:"thank",catName:"感谢邀约",style:"正式稳重版",content:"非常感谢XX家长一直以来的配合，家园同心，孩子才能更好地成长❤️",tip:"群内"},
  {cat:"thank",catName:"感谢邀约",style:"简短一句话",content:"感谢您积极参与本次亲子活动，有您的陪伴，宝贝收获满满🥰",tip:"群内"},
  {cat:"complaint",catName:"矛盾投诉",style:"正式稳重版",content:"收到您反馈的这件事，非常理解您的心情，我会立刻核实情况，稍后和您详细沟通。",tip:"【重要私信】"},
  {cat:"complaint",catName:"矛盾投诉",style:"正式稳重版",content:"这件事情我们已经调查清楚，已经对小朋友进行引导教育，后续在活动中我们加强看护，请您放心。",tip:"【重要私信】"}
];
//话术库分类
const wordsLib = [
  {name:"全班统一打卡总结",list:praisePool.whole},
  {name:"单个幼儿点对点表扬",list:praisePool.single},
  {name:"简短刷屏回复",list:praisePool.short}
];
//全局变量
let appData = {collect:[],students:[],record:{},dailySeed:Date.now(),parentCurrentCat:"feedback",parentStyleTag:"全部",parentSearch:"",wordSearch:""};
//本地存储读写
function loadStorage(){
  const c = localStorage.getItem(STORAGE_KEYS.collect);
  const s = localStorage.getItem(STORAGE_KEYS.students);
  const r = localStorage.getItem(STORAGE_KEYS.record);
  if(c) appData.collect = JSON.parse(c);
  if(s) appData.students = JSON.parse(s);
  if(r) appData.record = JSON.parse(r);
}
function saveStorage(){
  localStorage.setItem(STORAGE_KEYS.collect,JSON.stringify(appData.collect));
  localStorage.setItem(STORAGE_KEYS.students,JSON.stringify(appData.students));
  localStorage.setItem(STORAGE_KEYS.record,JSON.stringify(appData.record));
}
//Toast提示
function showToast(msg){
  const t = document.getElementById("toast");
  t.innerText = msg;t.style.display="block";
  setTimeout(()=>t.style.display="none",1800);
}
//替换XX
function replaceXX(str){
  const name = document.getElementById("global-name").value.trim();
  return str.replace(/XX/g,name||"XX");
}
//复制文本
async function copyText(txt){
  const final = replaceXX(txt);
  await navigator.clipboard.writeText(final);
  showToast("✅已复制");
}
//收藏
function toggleCollect(item){
  const idx = appData.collect.findIndex(x=>x.content === item.content);
  if(idx>=0) appData.collect.splice(idx,1);
  else appData.collect.push(item);
  saveStorage();renderCollect();showToast("❤️收藏更新");
}
//页面切换
document.querySelectorAll(".tab-item").forEach(tab=>{
  tab.onclick = ()=>{
    document.querySelectorAll(".tab-item").forEach(i=>i.classList.remove("active"));
    tab.classList.add("active");
    const pid = tab.dataset.page;
    document.querySelectorAll(".page").forEach(p=>p.classList.remove("active"));
    document.getElementById(pid).classList.add("active");
  }
})
//首页 每日话术
function getDailyPraise(){
  const seed = new Date().toDateString();
  const h = simpleHash(seed);
  const wIdx = h % praisePool.whole.length;
  const sIdx = h % praisePool.single.length;
  const shIdx = h % praisePool.short.length;
  return {
    whole:praisePool.whole[wIdx],
    single:praisePool.single[sIdx],
    short:praisePool.short[shIdx]
  }
}
function simpleHash(s){let h=0;for(let i=0;i<s.length;i++)h=((h<<5)-h)+s.charCodeAt(i)|0;return Math.abs(h);}
function refreshDaily(){renderIndex();showToast("🔄已换新一套话术")}
function renderIndex(){
  const daily = getDailyPraise();
  const html = `
    <div class="card">
      <div class="card-tip">📢全班总评</div>
      <div class="card-text">${daily.whole}</div>
      <div class="btn-group">
        <button class="btn-primary" onclick="copyText(\`${daily.whole}\`)">一键复制</button>
        <button class="btn-default" onclick="toggleCollect({content:\`${daily.whole}\`})">❤️收藏</button>
      </div>
    </div>
    <div class="card">
      <div class="card-tip">👶单人表扬</div>
      <div class="card-text">${daily.single}</div>
      <div class="btn-group">
        <button class="btn-primary" onclick="copyText(\`${daily.single}\`)">一键复制</button>
        <button class="btn-default" onclick="toggleCollect({content:\`${daily.single}\`})">❤️收藏</button>
      </div>
    </div>
    <div class="card">
      <div class="card-tip">⚡简短回复</div>
      <div class="card-text">${daily.short}</div>
      <div class="btn-group">
        <button class="btn-primary" onclick="copyText(\`${daily.short}\`)">一键复制</button>
        <button class="btn-default" onclick="toggleCollect({content:\`${daily.short}\`})">❤️收藏</button>
      </div>
    </div>
  `;
  document.getElementById("daily-container").innerHTML = html;
}
//话术库渲染
let activeWordCat = 0;
function renderWordsTab(){
  const wrap = document.getElementById("words-tab");
  wrap.innerHTML = wordsLib.map((item,i)=>`<span class="${i===activeWordCat?'active':''}" onclick="switchWordCat(${i})">${item.name}</span>`).join("");
}
function switchWordCat(i){activeWordCat = i;filterWords();}
function filterWords(){
  const kw = document.getElementById("search-word").value.toLowerCase();
  const list = wordsLib[activeWordCat].list.filter(txt=>txt.toLowerCase().includes(kw));
  let html = "";
  list.forEach(txt=>{
    html += `
      <div class="card">
        <div class="card-text">${txt}</div>
        <div class="btn-group">
          <button class="btn-primary" onclick="copyText(\`${txt}\`)">一键复制</button>
          <button class="btn-default" onclick="toggleCollect({content:\`${txt}\`})">❤️收藏</button>
        </div>
      </div>`
  })
  if(!list.length) html = "<div class='empty'>暂无匹配话术</div>";
  document.getElementById("words-container").innerHTML = html;
}
//家长对话页面
function renderParentTab(){
  const cats = [...new Set(parentTalkRaw.map(x=>x.catName))];
  const wrap = document.getElementById("parent-tab");
  wrap.innerHTML = cats.map((c,i)=>`<span class="${(appData.parentCurrentCat===c)?'active':''}" onclick="switchParentCat('${c}')">${c}</span>`).join("");
}
function switchParentCat(c){appData.parentCurrentCat = c;filterParent();}
document.querySelectorAll("#style-tab span").forEach(sp=>{
  sp.onclick = ()=>{
    document.querySelectorAll("#style-tab span").forEach(s=>s.classList.remove("active"));
    sp.classList.add("active");
    appData.parentStyleTag = sp.dataset.tag;filterParent();
  }
})
function filterParent(){
  const kw = document.getElementById("search-parent").value.toLowerCase();
  let arr = parentTalkRaw.filter(x=>x.catName === appData.parentCurrentCat);
  if(appData.parentStyleTag !== "全部") arr = arr.filter(x=>x.style === appData.parentStyleTag);
  if(kw) arr = arr.filter(x=>x.content.toLowerCase().includes(kw) || x.tip.toLowerCase().includes(kw));
  let html = "";
  arr.forEach(item=>{
    html += `
      <div class="card">
        <div class="card-tip">${item.tip}｜${item.style}</div>
        <div class="card-text">${item.content}</div>
        <div class="btn-group">
          <button class="btn-primary" onclick="copyText(\`${item.content}\`)">一键复制</button>
          <button class="btn-default" onclick="toggleCollect({content:\`${item.content}\`})">❤️收藏</button>
        </div>
      </div>`
  })
  if(!arr.length) html = "<div class='empty'>暂无匹配话术</div>";
  document.getElementById("parent-container").innerHTML = html;
}
//收藏页面
function renderCollect(){
  let html = "";
  appData.collect.forEach((item,idx)=>{
    html += `
    <div class="card">
      <div class="card-text">${item.content}</div>
      <div class="btn-group">
        <button class="btn-primary" onclick="copyText(\`${item.content}\`)">复制</button>
        <button class="btn-default" onclick="delCollect(${idx})">删除</button>
      </div>
    </div>`
  })
  if(!appData.collect.length) html = "<div class='empty'>暂无收藏话术</div>";
  document.getElementById("collect-container").innerHTML = html;
}
function delCollect(idx){
  appData.collect.splice(idx,1);saveStorage();renderCollect();showToast("已删除");
}
function addCustom(){
  const txt = document.getElementById("custom-text").value.trim();
  if(!txt) return showToast("请输入话术");
  appData.collect.push({content:txt});
  saveStorage();renderCollect();document.getElementById("custom-text").value="";showToast("保存成功");
}
//AI生成
function genAi(){
  const scene = document.getElementById("ai-scene").value.trim();
  if(!scene) return showToast("请输入场景");
  let res = `【AI生成文案】场景：${scene}\n1. 老师您好，XX今天表现很不错，继续加油\n2. XX宝贝很用心，点滴都在进步`;
  document.getElementById("ai-result").innerHTML = `<div class="card"><div class="card-text">${res}</div><button class="btn-primary" onclick="copyText(\`${res}\`)">复制全部</button></div>`;
}
function genWholeSet(){
  const w = praisePool.whole[simpleHash(Date.now())%praisePool.whole.length];
  const s = praisePool.single[simpleHash(Date.now())%praisePool.single.length];
  const sh = praisePool.short[simpleHash(Date.now())%praisePool.short.length];
  const all = `【全班总评】\n${w}\n\n【单人表扬】\n${s}\n\n【简短回复】\n${sh}`;
  document.getElementById("ai-result").innerHTML = `<div class="card"><div class="card-text">${all}</div><button class="btn-primary" onclick="copyText(\`${all}\`)">复制整套</button></div>`;
}
//打卡台账
function renderRecord(){
  const listWrap = document.getElementById("student-list");
  const statWrap = document.getElementById("record-stat");
  const today = new Date().toDateString();
  if(!appData.record[today]) appData.record[today] = [];
  let html = "";
  appData.students.forEach((name,i)=>{
    const checked = appData.record[today].includes(name);
    html += `<div class="check-item">
      <input type="checkbox" ${checked?"checked":""} onchange="toggleStudentRecord('${name}')">
      <span>${name}</span>
      <button class="btn-default" onclick="delStudent(${i})">删除</button>
    </div>`
  })
  listWrap.innerHTML = html;
  const total = appData.students.length;
  const done = appData.record[today].length;
  statWrap.innerHTML = `<div class="title">今日打卡统计：${done}/${total}</div>`;
}
document.getElementById("student-name").onkeydown = function(e){
  if(e.key==="Enter"){
    const name = e.target.value.trim();
    if(!name) return;
    appData.students.push(name);
    saveStorage();renderRecord();e.target.value="";
  }
}
function toggleStudentRecord(name){
  const today = new Date().toDateString();
  if(!appData.record[today]) appData.record[today]=[];
  const idx = appData.record[today].indexOf(name);
  if(idx>=0) appData.record[today].splice(idx,1);
  else appData.record[today].push(name);
  saveStorage();renderRecord();
}
function delStudent(idx){appData.students.splice(idx,1);saveStorage();renderRecord();}
function clearToday(){
  const today = new Date().toDateString();
  delete appData.record[today];
  saveStorage();renderRecord();showToast("已清空今日打卡");
}
//初始化
window.onload = function(){
  loadStorage();
  renderIndex();
  renderWordsTab();filterWords();
  renderParentTab();filterParent();
  renderCollect();
  renderRecord();
}
</script>
</body>
</html>

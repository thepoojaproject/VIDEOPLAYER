<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Video Player</title>
<style>
  :root{
    --bg:#0f1724; --card:#0b1220; --accent:#7c5cff; --muted:#9aa4b2; --glass: rgba(255,255,255,0.03);
    --maxw:1100px;
  }
  *{box-sizing:border-box}
  body{
    margin:0; font-family:Inter,system-ui,Segoe UI,Roboto,"Helvetica Neue",Arial; background:
    linear-gradient(180deg,#071028 0%, #071720 60%); color:#e6eef6; min-height:100vh;
    display:flex; align-items:center; justify-content:center; padding:24px;
  }

  .app{
    width:100%; max-width:var(--maxw); background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
    border-radius:12px; box-shadow: 0 6px 30px rgba(2,6,23,0.7); overflow:hidden;
    display:grid; grid-template-columns: 1fr 320px; gap:18px;
  }

  @media (max-width:920px){ .app{ grid-template-columns: 1fr; } .sidebar{ order:2 } }

  .player-wrap{ padding:18px; display:flex; flex-direction:column; gap:12px; }
  .video-card{ background:var(--glass); border-radius:8px; padding:10px; display:flex; flex-direction:column; gap:12px; }
  .video-container{ position:relative; background:#000; border-radius:6px; overflow:hidden; display:flex; align-items:center; justify-content:center; height:56vh; min-height:260px; }
  video{ width:100%; height:100%; object-fit:contain; background:#000; display:block; }

  .controls{ display:flex; gap:8px; align-items:center; flex-wrap:wrap; }
  .btn{ background:transparent; border:1px solid rgba(255,255,255,0.06); padding:8px 12px; border-radius:8px; color:var(--muted); cursor:pointer; font-weight:600; }
  .btn.primary{ background:linear-gradient(90deg,var(--accent), #5b45ff); color:white; border: none; box-shadow: 0 6px 18px rgba(124,92,255,0.14); }
  .range{ -webkit-appearance:none; width:220px; height:6px; background:rgba(255,255,255,0.06); border-radius:6px; outline:none; }
  .progress{ width:100%; height:8px; background:rgba(255,255,255,0.04); border-radius:6px; cursor:pointer; }
  .time{ color:var(--muted); font-size:13px; min-width:80px; text-align:right; }

  .meta{ display:flex; justify-content:space-between; align-items:center; gap:12px; }
  .title{ font-weight:700; font-size:16px; }
  .subtitle{ color:var(--muted); font-size:13px; margin-top:4px; }

  .sidebar{ padding:18px; border-left:1px solid rgba(255,255,255,0.02); display:flex; flex-direction:column; gap:12px; min-width:260px; }
  .card{ background:rgba(255,255,255,0.02); padding:12px; border-radius:10px; }
  .file-input{ display:flex; gap:8px; align-items:center; flex-wrap:wrap; }
  input[type="file"]{ display:none; }
  .dropzone{ border:1px dashed rgba(255,255,255,0.04); padding:12px; border-radius:8px; text-align:center; color:var(--muted); cursor:pointer; }

  .playlist{ display:flex; flex-direction:column; gap:6px; max-height:40vh; overflow:auto; padding-right:6px; }
  .pl-item{ display:flex; justify-content:space-between; gap:8px; padding:8px; border-radius:8px; align-items:center; cursor:pointer; }
  .pl-item:hover{ background:rgba(255,255,255,0.01); }
  .pl-active{ background:linear-gradient(90deg,var(--accent), #5b45ff); color:white; }
  .small{ font-size:13px; color:var(--muted) }

  .flex{ display:flex; gap:8px; align-items:center; }
  .grow{ flex:1; }
  .muted{ color:var(--muted) }
  .input{ background:transparent; border:1px solid rgba(255,255,255,0.04); padding:8px; border-radius:8px; color:inherit; width:100%; }

  footer.app-footer{ grid-column:1/-1; text-align:center; padding:8px 12px; color:var(--muted); font-size:13px; }

  .icon{ width:18px; height:18px; display:inline-block; margin-right:6px; vertical-align:-2px; }
  .danger{ color:#ff6b6b }
</style>
</head>
<body>
<div class="app" id="app">
  <div class="player-wrap">
    <div class="video-card">
      <div class="meta">
        <div>
          <div class="title" id="title">No video loaded</div>
          <div class="subtitle" id="subtitle">Drag & drop a file, open a URL or add to playlist</div>
        </div>
        <div class="flex">
          <button class="btn" id="btn-pip" title="Picture in Picture">PIP</button>
          <button class="btn" id="btn-full" title="Fullscreen">⤢</button>
          <button class="btn" id="btn-download" title="Download current link">Download</button>
        </div>
      </div>

      <div class="video-container" id="videoContainer">
        <video id="video" controls crossorigin playsinline></video>
      </div>

      <div style="display:flex; gap:12px; align-items:center;">
        <div style="flex:1;">
          <div class="progress" id="progressBar"></div>
        </div>
        <div class="time" id="time">00:00 / 00:00</div>
      </div>

      <div class="controls">
        <div class="flex">
          <button class="btn primary" id="playPause">Play</button>
          <button class="btn" id="stop">Stop</button>
          <button class="btn" id="prev">Prev</button>
          <button class="btn" id="next">Next</button>
        </div>

        <div class="flex">
          <label class="small muted">Vol</label>
          <input id="volume" class="range" type="range" min="0" max="1" step="0.01" />
          <button class="btn" id="mute">Mute</button>
        </div>

        <div class="flex">
          <label class="small muted">Speed</label>
          <select id="speed" class="input" style="width:88px">
            <option>0.25</option><option>0.5</option><option>0.75</option>
            <option selected>1</option><option>1.25</option><option>1.5</option><option>2</option>
          </select>
          <label class="small muted">Loop <input type="checkbox" id="loop"></label>
          <button class="btn" id="shuffleBtn">Shuffle</button>
        </div>

        <div class="flex grow" style="justify-content:flex-end">
          <label class="small muted">Subtitles</label>
          <input type="file" id="subFile" accept=".vtt" class="btn" />
          <button class="btn" id="removeSub">Remove</button>
        </div>
      </div>
    </div>

    <div class="card" style="display:flex; gap:8px; flex-wrap:wrap; align-items:center;">
      <div style="flex:1">
        <input id="urlInput" class="input" placeholder="Paste video URL (http(s)...) and press Add" />
      </div>
      <button class="btn primary" id="addUrlBtn">Add URL</button>

      <label class="file-input">
        <label for="filePicker" class="dropzone" id="dropzone">Click or Drag & Drop files here</label>
        <input id="filePicker" type="file" accept="video/*" multiple />
      </label>
    </div>
  </div>

  <div class="sidebar">
    <div class="card">
      <div style="display:flex; justify-content:space-between; align-items:center;">
        <div style="font-weight:700">Playlist</div>
        <div class="small muted">Items: <span id="plCount">0</span></div>
      </div>
      <div class="playlist" id="playlist"></div>

      <div style="display:flex; gap:8px; margin-top:10px;">
        <button class="btn" id="clearPl">Clear</button>
        <button class="btn" id="savePl">Export JSON</button>
        <button class="btn" id="loadPl">Import JSON</button>
        <input type="file" id="plImport" accept="application/json" style="display:none" />
      </div>
    </div>

    <div class="card">
      <div style="font-weight:700">Shortcuts</div>
      <div class="small muted" style="margin-top:8px; line-height:1.6">
        Space: Play/Pause<br>
        ← / → : Seek -/+ 5s<br>
        ↑ / ↓ : Volume + / -<br>
        F : Fullscreen<br>
        P : Picture-in-Picture<br>
        L : Toggle loop<br>
        S : Stop<br>
        O : Open file dialog
      </div>
    </div>

    <div class="card" style="display:flex; gap:8px; align-items:center; justify-content:space-between;">
      <div>
        <div style="font-weight:700">About</div>
        <div class="small muted">Simple, fast web video player — save as a file and open.</div>
      </div>
      <div class="small muted">v1.0</div>
    </div>
  </div>

  <footer class="app-footer">Made with ❤️ —  By Armeen</footer>
</div>

<!-- Full, ready-to-paste script (includes YouTube IFrame API support) -->
<script>
/* Video Player with YouTube iframe support */

const video = document.getElementById('video');
const playPause = document.getElementById('playPause');
const stopBtn = document.getElementById('stop');
const prevBtn = document.getElementById('prev');
const nextBtn = document.getElementById('next');
const volume = document.getElementById('volume');
const mute = document.getElementById('mute');
const speed = document.getElementById('speed');
const loopBox = document.getElementById('loop');
const progressBar = document.getElementById('progressBar');
const titleEl = document.getElementById('title');
const subtitleEl = document.getElementById('subtitle');
const dropzone = document.getElementById('dropzone');
const filePicker = document.getElementById('filePicker');
const addUrlBtn = document.getElementById('addUrlBtn');
const urlInput = document.getElementById('urlInput');
const playlistEl = document.getElementById('playlist');
const plCount = document.getElementById('plCount');
const clearPl = document.getElementById('clearPl');
const savePl = document.getElementById('savePl');
const loadPl = document.getElementById('loadPl');
const plImport = document.getElementById('plImport');
const btnFull = document.getElementById('btn-full');
const btnPip = document.getElementById('btn-pip');
const btnDownload = document.getElementById('btn-download');
const subFile = document.getElementById('subFile');
const removeSub = document.getElementById('removeSub');
const videoContainer = document.getElementById('videoContainer');
const timeEl = document.getElementById('time');

let playlist = []; // {title, src, type, origin}
let currentIndex = -1;
let isShuffle = false;

// YouTube player objects
let ytPlayer = null;
let currentIsYouTube = false;
let ytPlayerReady = false;

/* Load YouTube IFrame API (async) */
(function loadYouTubeAPI(){
  const tag = document.createElement('script');
  tag.src = "https://www.youtube.com/iframe_api";
  document.head.appendChild(tag);
})();

/* global function called by YouTube API when ready */
function onYouTubeIframeAPIReady(){
  // required global for API; we create players on demand
}

/* helpers */
const fmt = s => {
  if (!isFinite(s)) return '00:00';
  const h = Math.floor(s/3600), m = Math.floor((s%3600)/60), sec = Math.floor(s%60);
  if (h>0) return `${String(h).padStart(2,'0')}:${String(m).padStart(2,'0')}:${String(sec).padStart(2,'0')}`;
  return `${String(m).padStart(2,'0')}:${String(sec).padStart(2,'0')}`;
};

function isYouTubeUrl(u){
  try {
    const url = new URL(u);
    return /(^|\.)youtube\.com$/.test(url.hostname) || /^youtu\.be$/.test(url.hostname) || url.hostname.endsWith('youtube-nocookie.com');
  } catch(e){ return false; }
}

function extractYouTubeID(u){
  try {
    const url = new URL(u);
    if (url.hostname === 'youtu.be') return url.pathname.slice(1);
    if (url.searchParams.get('v')) return url.searchParams.get('v');
    const parts = url.pathname.split('/');
    return parts.pop() || parts.pop();
  } catch(e){
    return null;
  }
}

/* Playlist rendering */
function renderPlaylist(){
  playlistEl.innerHTML = '';
  playlist.forEach((it, idx) => {
    const div = document.createElement('div');
    div.className = 'pl-item' + (idx===currentIndex ? ' pl-active' : '');
    div.innerHTML = `<div style="overflow:hidden">
        <div style="font-weight:600">${it.title||('Video '+(idx+1))}</div>
        <div class="small muted">${it.origin || it.type || ''}</div>
      </div>
      <div style="display:flex; gap:6px; align-items:center">
        <button class="btn" data-idx="${idx}" title="Play">▶</button>
        <button class="btn" data-remove="${idx}" title="Remove">✖</button>
      </div>`;
    playlistEl.appendChild(div);
  });
  plCount.textContent = playlist.length;
}

/* Switch source (handles both direct video and YouTube) */
function setSourceByIndex(i, autoplay=true){
  if (i<0 || i>=playlist.length) return;
  currentIndex = i;
  const it = playlist[i];

  // cleanup previous YouTube player (if any) when switching to non-YT
  if (ytPlayer && !isYouTubeUrl(it.src)) {
    try { ytPlayer.destroy(); } catch(e){}
    ytPlayer = null;
    ytPlayerReady = false;
  }

  if (isYouTubeUrl(it.src)){
    // load YouTube iframe
    currentIsYouTube = true;
    video.style.display = 'none';
    try { video.pause(); } catch(e){}
    video.removeAttribute('src'); video.load();
    loadYouTube(it.src, it.title);
    titleEl.textContent = it.title || 'YouTube Video';
    subtitleEl.textContent = it.src;
  } else {
    // normal direct video
    currentIsYouTube = false;
    removeYouTubeIframe();
    video.style.display = 'block';
    video.src = it.src;
    video.dataset.origin = it.origin || 'url';
    titleEl.textContent = it.title || 'Video';
    subtitleEl.textContent = it.src;
    if (autoplay) video.play().catch(()=>{});
  }
  renderPlaylist();
  updateUIForSource();
}

/* create YouTube iframe player */
function loadYouTube(url, title){
  const vid = extractYouTubeID(url);
  if (!vid) { alert('Unable to parse YouTube ID.'); return; }

  let existing = document.getElementById('ytFrameWrapper');
  if (!existing){
    const wrapper = document.createElement('div');
    wrapper.id = 'ytFrameWrapper';
    wrapper.style.width = '100%';
    wrapper.style.height = '100%';
    wrapper.style.display = 'block';
    wrapper.style.position = 'relative';
    wrapper.style.background = '#000';
    videoContainer.appendChild(wrapper);
  } else {
    existing.innerHTML = '';
    existing.style.display = 'block';
  }

  if (ytPlayer){
    try { ytPlayer.destroy(); } catch(e){}
    ytPlayer = null;
    ytPlayerReady = false;
  }

  const frame = document.createElement('div');
  frame.id = 'ytplayer_' + Date.now();
  document.getElementById('ytFrameWrapper').appendChild(frame);

  function createPlayer(){
    try {
      ytPlayer = new YT.Player(frame.id, {
        height: '100%',
        width: '100%',
        videoId: vid,
        playerVars: {
          controls: 1,
          modestbranding: 1,
          rel: 0,
          enablejsapi: 1,
          origin: location.origin
        },
        events: {
          'onReady': (e) => { ytPlayerReady = true; },
          'onStateChange': onYouTubeStateChange,
          'onError': (ev) => { console.error('YT error', ev); alert('YouTube playback error'); }
        }
      });
    } catch(err){
      setTimeout(createPlayer, 300);
    }
  }
  createPlayer();
}

/* remove iframe if exists */
function removeYouTubeIframe(){
  const wrapper = document.getElementById('ytFrameWrapper');
  if (wrapper){
    try {
      if (ytPlayer) ytPlayer.destroy();
    } catch(e){}
    wrapper.remove();
    ytPlayer = null;
    ytPlayerReady = false;
  }
}

/* YouTube state changes -> update play/pause label */
function onYouTubeStateChange(e){
  if (e.data === YT.PlayerState.PLAYING) playPause.textContent = 'Pause';
  else if (e.data === YT.PlayerState.PAUSED || e.data === YT.PlayerState.ENDED) playPause.textContent = 'Play';
  if (e.data === YT.PlayerState.ENDED) {
    setTimeout(()=> {
      if (isShuffle) setSourceByIndex(Math.floor(Math.random()*playlist.length));
      else {
        if (currentIndex < playlist.length - 1) setSourceByIndex(currentIndex + 1);
      }
    }, 300);
  }
}

/* UI enabling/disabling depending on source type */
function updateUIForSource(){
  if (currentIsYouTube){
    btnPip.disabled = true; btnPip.classList.add('muted');
    subFile.disabled = true; removeSub.disabled = true;
    btnDownload.disabled = false;
  } else {
    btnPip.disabled = false; btnPip.classList.remove('muted');
    subFile.disabled = false; removeSub.disabled = false;
    btnDownload.disabled = false;
  }
}

/* Player controls wiring (works for both youtube and <video>) */
playPause.addEventListener('click', ()=> {
  if (currentIsYouTube){
    if (!ytPlayerReady) return;
    const st = ytPlayer.getPlayerState();
    if (st === YT.PlayerState.PLAYING) ytPlayer.pauseVideo();
    else ytPlayer.playVideo();
  } else {
    if (video.paused) video.play(); else video.pause();
  }
});

stopBtn.addEventListener('click', ()=>{
  if (currentIsYouTube && ytPlayerReady){
    ytPlayer.stopVideo();
  } else {
    video.pause(); video.currentTime = 0;
  }
});

prevBtn.addEventListener('click', ()=> {
  if (playlist.length===0) return;
  if (isShuffle) {
    setSourceByIndex(Math.floor(Math.random()*playlist.length));
  } else {
    setSourceByIndex((currentIndex<=0) ? playlist.length-1 : currentIndex-1);
  }
});
nextBtn.addEventListener('click', ()=> {
  if (playlist.length===0) return;
  if (isShuffle) {
    setSourceByIndex(Math.floor(Math.random()*playlist.length));
  } else {
    setSourceByIndex((currentIndex>=playlist.length-1) ? 0 : currentIndex+1);
  }
});

/* Video element events */
video.addEventListener('play', ()=> { if (!currentIsYouTube) playPause.textContent='Pause'; });
video.addEventListener('pause', ()=> { if (!currentIsYouTube) playPause.textContent='Play'; });
video.addEventListener('timeupdate', ()=> {
  if (currentIsYouTube) return;
  const pct = (video.currentTime / (video.duration || 1)) * 100;
  progressBar.style.background = `linear-gradient(90deg, var(--accent) ${pct}%, rgba(255,255,255,0.02) ${pct}%)`;
  timeEl.textContent = `${fmt(video.currentTime)} / ${fmt(video.duration)}`;
});

/* progress click (works only for <video>) */
progressBar.addEventListener('click', (e)=>{
  if (currentIsYouTube) return;
  const rect = progressBar.getBoundingClientRect();
  const x = e.clientX - rect.left;
  const pct = x / rect.width;
  if (video.duration) video.currentTime = pct * video.duration;
});

/* volume */
volume.value = 1;
volume.addEventListener('input', ()=> {
  const v = Number(volume.value);
  if (currentIsYouTube && ytPlayerReady) {
    try { ytPlayer.setVolume(Math.round(v*100)); }
    catch(e){ }
  } else {
    video.volume = v;
  }
  if ((currentIsYouTube && ytPlayerReady && ytPlayer.isMuted && ytPlayer.isMuted()) || video.muted) mute.textContent='Unmute';
  else mute.textContent='Mute';
});

mute.addEventListener('click', ()=> {
  if (currentIsYouTube && ytPlayerReady){
    try {
      if (ytPlayer.isMuted && ytPlayer.isMuted()) { ytPlayer.unMute(); mute.textContent='Mute'; }
      else { ytPlayer.mute(); mute.textContent='Unmute'; }
    } catch(e){}
  } else {
    video.muted = !video.muted;
    mute.textContent = video.muted ? 'Unmute' : 'Mute';
  }
});

/* speed & loop */
speed.addEventListener('change', ()=> {
  const val = Number(speed.value);
  if (currentIsYouTube && ytPlayerReady){
    try { ytPlayer.setPlaybackRate(val); } catch(e){}
  } else {
    video.playbackRate = val;
  }
});
loopBox.addEventListener('change', ()=> {
  if (!currentIsYouTube) video.loop = loopBox.checked;
});

/* shuffle */
document.getElementById('shuffleBtn').addEventListener('click', (e)=>{
  isShuffle = !isShuffle;
  e.target.classList.toggle('primary', isShuffle);
});

/* keyboard shortcuts */
window.addEventListener('keydown', (e)=>{
  if (e.target.tagName === 'INPUT' || e.target.tagName === 'SELECT' || e.target.isContentEditable) return;
  switch(e.key.toLowerCase()){
    case ' ': e.preventDefault(); if (currentIsYouTube){ if (ytPlayerReady){ const st = ytPlayer.getPlayerState(); if (st===YT.PlayerState.PLAYING) ytPlayer.pauseVideo(); else ytPlayer.playVideo(); } } else { if (video.paused) video.play(); else video.pause(); } break;
    case 'arrowleft': if (!currentIsYouTube && video.currentTime) video.currentTime = Math.max(0, video.currentTime - 5); break;
    case 'arrowright': if (!currentIsYouTube && video.currentTime) video.currentTime = Math.min(video.duration || 0, video.currentTime + 5); break;
    case 'arrowup': volume.value = Math.min(1, Number(volume.value) + 0.1); volume.dispatchEvent(new Event('input')); break;
    case 'arrowdown': volume.value = Math.max(0, Number(volume.value) - 0.1); volume.dispatchEvent(new Event('input')); break;
    case 'f': toggleFullscreen(); break;
    case 'p': togglePIP(); break;
    case 'l': loopBox.checked = !loopBox.checked; if (!currentIsYouTube) video.loop = loopBox.checked; break;
    case 's': if (currentIsYouTube && ytPlayerReady) ytPlayer.stopVideo(); else { video.pause(); video.currentTime = 0; } break;
    case 'o': filePicker.click(); break;
  }
});

/* Fullscreen */
function toggleFullscreen(){
  const el = document.getElementById('videoContainer');
  if (!document.fullscreenElement) el.requestFullscreen().catch(()=>{});
  else document.exitFullscreen().catch(()=>{});
}
btnFull.addEventListener('click', toggleFullscreen);

/* Picture-in-Picture */
async function togglePIP(){
  if (currentIsYouTube){ alert('Picture-in-Picture is not available for embedded YouTube iframe in this player.'); return; }
  try {
    if (document.pictureInPictureElement) await document.exitPictureInPicture();
    else await video.requestPictureInPicture();
  } catch(e){}
}
btnPip.addEventListener('click', togglePIP);

/* Download */
btnDownload.addEventListener('click', ()=>{
  const src = currentIsYouTube ? (playlist[currentIndex] && playlist[currentIndex].src) : (video.currentSrc || '');
  if (!src) return alert('No source to download.');
  if (isYouTubeUrl(src)) {
    window.open(src, '_blank');
    return;
  }
  const a = document.createElement('a');
  a.href = src;
  a.download = (playlist[currentIndex] && playlist[currentIndex].title) ? playlist[currentIndex].title : 'video';
  document.body.appendChild(a);
  a.click();
  a.remove();
});

/* subtitle handling */
subFile.addEventListener('change', (e)=>{
  if (currentIsYouTube){ alert('Subtitles (.vtt) cannot be added to YouTube iframe here. YouTube may already have captions.'); subFile.value=''; return; }
  const f = e.target.files && e.target.files[0];
  if (!f) return;
  const url = URL.createObjectURL(f);
  const existing = video.querySelectorAll('track'); existing.forEach(t=>t.remove());
  const currentTrack = document.createElement('track');
  currentTrack.kind='subtitles'; currentTrack.label=f.name; currentTrack.src=url; currentTrack.default=true;
  video.appendChild(currentTrack);
  subFile.value='';
});
removeSub.addEventListener('click', ()=> {
  if (currentIsYouTube){ alert('No subtitle to remove for YouTube iframe here.'); return; }
  const existing = video.querySelectorAll('track'); existing.forEach(t=>{
    if (t.src && t.src.startsWith('blob:')) URL.revokeObjectURL(t.src);
    t.remove();
  });
});

/* Playlist functions */
filePicker.addEventListener('change', ev=>{
  const files = Array.from(ev.target.files || []);
  addFilesToPlaylist(files);
  filePicker.value = '';
});

function addFilesToPlaylist(files){
  files.forEach(f=>{
    const url = URL.createObjectURL(f);
    playlist.push({ title: f.name, src: url, type: f.type, origin: 'file', _fileRef: f });
  });
  if (currentIndex===-1 && playlist.length>0) setSourceByIndex(0, false);
  renderPlaylist();
}

/* drag & drop */
dropzone.addEventListener('dragover', e=>{ e.preventDefault(); dropzone.style.borderColor='rgba(255,255,255,0.2)'; });
dropzone.addEventListener('dragleave', e=>{ dropzone.style.borderColor='rgba(255,255,255,0.04)'; });
dropzone.addEventListener('drop', e=>{
  e.preventDefault(); dropzone.style.borderColor='rgba(255,255,255,0.04)';
  const items = Array.from(e.dataTransfer.files || []);
  if (items.length) addFilesToPlaylist(items);
});
dropzone.addEventListener('click', ()=> filePicker.click());

/* Add URL (detect youtube) */
addUrlBtn.addEventListener('click', ()=>{
  const url = urlInput.value.trim();
  if (!url) return alert('Paste a video URL first.');
  try { new URL(url); } catch(e){ return alert('Invalid URL'); }
  const name = url.split('/').pop().split('?')[0] || url;
  const origin = isYouTubeUrl(url) ? 'youtube' : 'url';
  playlist.push({ title: name, src: url, type: 'video', origin });
  if (currentIndex===-1) setSourceByIndex(0,false);
  renderPlaylist();
  urlInput.value='';
});

/* Playlist click handlers */
playlistEl.addEventListener('click', (e)=>{
  const idx = e.target.getAttribute('data-idx');
  const rem = e.target.getAttribute('data-remove');
  if (idx !== null) {
    setSourceByIndex(Number(idx));
  } else if (rem !== null) {
    const i = Number(rem);
    if (playlist[i] && playlist[i].origin==='file' && playlist[i].src.startsWith('blob:')) {
      URL.revokeObjectURL(playlist[i].src);
    }
    playlist.splice(i,1);
    if (i === currentIndex) {
      if (playlist.length===0) {
        currentIndex = -1;
        video.removeAttribute('src'); video.load(); titleEl.textContent='No video loaded'; subtitleEl.textContent='Drag & drop a file, open a URL or add to playlist';
        removeYouTubeIframe();
      } else {
        setSourceByIndex(Math.min(i, playlist.length-1));
      }
    } else if (i < currentIndex) currentIndex--;
    renderPlaylist();
  }
});

/* save/load & clear */
savePl.addEventListener('click', ()=>{
  const out = JSON.stringify(playlist.map(p=> ({ title:p.title, src:p.src, origin:p.origin }) ));
  const a = document.createElement('a'); a.href = URL.createObjectURL(new Blob([out],{type:'application/json'}));
  a.download = 'playlist.json'; document.body.appendChild(a); a.click(); a.remove();
});
loadPl.addEventListener('click', ()=> plImport.click());
plImport.addEventListener('change', (e)=>{
  const f = e.target.files && e.target.files[0]; if (!f) return;
  const reader = new FileReader();
  reader.onload = ()=> {
    try {
      const data = JSON.parse(reader.result);
      data.forEach(it=> playlist.push(it));
      if (currentIndex===-1 && playlist.length>0) setSourceByIndex(0,false);
      renderPlaylist();
    } catch(err){ alert('Invalid playlist file'); }
  };
  reader.readAsText(f);
  plImport.value='';
});
clearPl.addEventListener('click', ()=>{
  playlist.forEach(it=>{ if (it.origin==='file' && it.src && it.src.startsWith('blob:')) URL.revokeObjectURL(it.src); });
  playlist = []; currentIndex = -1;
  video.removeAttribute('src'); video.load();
  titleEl.textContent='No video loaded'; subtitleEl.textContent='Drag & drop a file, open a URL or add to playlist';
  removeYouTubeIframe();
  renderPlaylist();
});

/* video ended */
video.addEventListener('ended', ()=>{
  if (isShuffle) setSourceByIndex(Math.floor(Math.random()*playlist.length));
  else {
    if (currentIndex < playlist.length - 1) setSourceByIndex(currentIndex + 1);
    else if (video.loop) video.play();
  }
});

/* initial render */
renderPlaylist();

/* Error handling */
video.addEventListener('error', (e)=> {
  console.error('Video error', e);
  alert('Could not play this video (CORS or unsupported format). Try downloading or using another source.');
});

window.addEventListener('unload', ()=> {
  try { if (ytPlayer) ytPlayer.destroy(); } catch(e){}
});
</script>
</body>
</html>

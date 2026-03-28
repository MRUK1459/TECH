<!DOCTYPE html>
<html lang="si">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ජෛව රසායනය - Large YouTube UI</title>
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;500&display=swap" rel="stylesheet">
    <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
    <style>
        body { background: #000; margin: 0; padding: 0; font-family: 'Roboto', sans-serif; color: white; overflow-x: hidden; }

        /* Icons Fix & Large Size */
        .material-Icons {
            font-family: 'Material Icons';
            font-size: 32px; /* අයිකන විශාල කරන ලදී */
            display: inline-block;
            line-height: 1;
            text-transform: none;
            letter-spacing: normal;
            word-wrap: normal;
            white-space: nowrap;
            direction: ltr;
            -webkit-font-smoothing: antialiased;
            font-feature-settings: 'liga';
            opacity: 0;
            transition: opacity 0.3s ease;
        }

        .icons-loaded .material-Icons { opacity: 1; }

        /* Login Screen */
        #loginOverlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: #0b0b0b; z-index: 1000; display: flex; align-items: center; justify-content: center; }
        .login-box { background: #111; padding: 30px; border-radius: 15px; width: 85%; max-width: 350px; text-align: center; border: 1px solid #222; }
        .login-box h2 { color: #ff0000; margin-bottom: 25px; }
        .login-box input { width: 100%; padding: 14px; margin: 10px 0; background: #1a1a1a; border: 1px solid #333; color: white; border-radius: 8px; box-sizing: border-box; }
        .login-box button { width: 100%; padding: 14px; background: #ff0000; color: white; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; }

        /* Player Styles */
        #mainContent { display: none; padding: 10px; }
        .video-box { position: relative; max-width: 900px; margin: 10px auto; background: #000; border-radius: 12px; overflow: hidden; aspect-ratio: 16 / 9; }
        #player { width: 100%; height: 100%; pointer-events: none; }
        .click-layer { position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 15; cursor: pointer; }

        /* Large Control Bar */
        .custom-controls {
            position: absolute; bottom: 0; left: 0; right: 0;
            background: linear-gradient(transparent, rgba(0,0,0,0.9));
            display: flex; flex-direction: column; 
            padding: 15px 20px; /* බාර් එකේ ඉඩ වැඩි කරන ලදී */
            transition: 0.4s; z-index: 20;
        }
        .custom-controls.hidden { opacity: 0; transform: translateY(25px); pointer-events: none; }

        /* Seek Bar - Thicker for YouTube feel */
        .seek-bar-container { width: 100%; display: flex; align-items: center; margin-bottom: 12px; cursor: pointer; position: relative; height: 10px; }
        .seek-bar-slider { width: 100%; height: 5px; background: rgba(255,255,255,0.3); border-radius: 3px; position: relative; }
        .seek-bar-progress { height: 100%; background: #ff0000; width: 0%; border-radius: 3px; }
        
        /* Larger Knob */
        .seek-bar-knob {
            width: 16px; height: 16px; background: #ff0000;
            border-radius: 50%; position: absolute;
            top: 50%; transform: translate(-50%, -50%);
            left: 0%; transition: opacity 0.2s;
        }

        .time-row { display: flex; align-items: center; justify-content: space-between; }
        .controls-row { display: flex; align-items: center; gap: 20px; }

        .time-display { font-size: 14px; color: #fff; font-weight: 500; }
        .now-playing { font-size: 15px; color: #fff; font-weight: 500; margin-left: 10px; opacity: 0.9; }

        .icon-btn { background: none; border: none; color: #fff; cursor: pointer; display: flex; align-items: center; justify-content: center; padding: 5px; }
        .icon-btn:active { transform: scale(0.8); }

        /* Speed Select Large */
        .select-style { background: #222; color: white; border: 1px solid #444; padding: 6px 10px; border-radius: 5px; font-size: 14px; cursor: pointer; }

        /* Playlist */
        .playlist-container { max-width: 900px; margin: 20px auto; background: #111; border-radius: 12px; padding: 15px; border: 1px solid #222; }
        .playlist-item { display: flex; align-items: center; padding: 15px; margin-bottom: 8px; background: #1a1a1a; border-radius: 8px; cursor: pointer; }
        .playlist-item.active { border: 1px solid #ff0000; background: #ff000010; }
    </style>
</head>
<body>

    <div id="loginOverlay">
        <div class="login-box">
            <h2>ජෛව රසායනය - Login</h2>
            <input type="text" id="username" placeholder="පරිශීලක නාමය">
            <input type="password" id="password" placeholder="මුරපදය">
            <button onclick="checkLogin()">ඇතුල් වන්න</button>
            <p id="error" style="color:red; display:none; margin-top:10px;">වැරදියි!</p>
        </div>
    </div>

    <div id="mainContent">
        <div class="video-box" id="videoContainer">
            <div id="player"></div>
            <div class="click-layer" onclick="toggleControls()" ondblclick="skipForward(event)"></div>
            
            <div class="custom-controls" id="controlBar">
                <div class="seek-bar-container" id="seekBarContainer" onclick="handleSeek(event)">
                    <div class="seek-bar-slider">
                        <div class="seek-bar-progress" id="seekBarProgress"></div>
                        <div class="seek-bar-knob" id="seekBarKnob"></div>
                    </div>
                </div>

                <div class="time-row">
                    <div class="controls-row">
                        <button class="icon-btn" onclick="togglePlay(event)" id="playPauseBtn">
                            <i class="material-Icons">play_arrow</i>
                        </button>
                        <div class="time-display">
                            <span id="currentTime">0:00</span> / <span id="duration">0:00</span>
                        </div>
                        <div class="now-playing" id="nowPlaying">පූරණය වේ...</div>
                    </div>

                    <div class="controls-row">
                        <select class="select-style" id="speed" onchange="setSpeed(event)">
                            <option value="0.5">0.5x</option>
                            <option value="1" selected>Normal</option>
                            <option value="1.5">1.5x</option>
                            <option value="2">2x</option>
                        </select>
                        <button class="icon-btn" onclick="toggleFullScreen(event)" id="fullscreenBtn">
                            <i class="material-Icons">fullscreen</i>
                        </button>
                    </div>
                </div>
            </div>
        </div>
        <div class="playlist-container" id="playlist"></div>
    </div>

<script>
    const VALID_USER = "Ulindu kawishka";
    const VALID_PASS = "200704938Uk";

    document.addEventListener("DOMContentLoaded", function() {
        if (localStorage.getItem("app_authenticated") === "true") showPlayer();
        if (document.fonts) {
            document.fonts.load("12px Material Icons").then(() => document.body.classList.add("icons-loaded"));
        }
    });

    function checkLogin() {
        if (document.getElementById("username").value === VALID_USER && document.getElementById("password").value === VALID_PASS) {
            localStorage.setItem("app_authenticated", "true");
            showPlayer();
        } else {
            document.getElementById("error").style.display = "block";
        }
    }

    function showPlayer() {
        document.getElementById("loginOverlay").style.display = "none";
        document.getElementById("mainContent").style.display = "block";
        loadYouTubeAPI();
    }

    const videoData = [
        { id: "lMR_RtHbGcw", title: "ජෛව රසායනය ( Day 01 )" },
        { id: "SOi5karTuQU", title: "ජෛව රසායනය ( Day 02 )[Part 01]" },
        { id: "2iCOyz9fWME", title: "ජෛව රසායනය ( Day 02 )[Part 02]" },
        { id: "IcMmdsa3yw8", title: "ජෛව රසායනය - Day 03" },
        { id: "pp49TELoPs0", title: "ජෛව රසායනය - Day 04" },
        { id: "AdfjiYBNDTw", title: "ජෛව රසායනය - Day 04(part 2)" },
        { id: "XH3uO16T7A0", title: "ජෛව රසායනය - Day 05" },
        { id: "hpdxRUjHJhU", title: "ජෛව රසායනය - Day 05(part2)" },
        { id: "G36isU4cPhU", title: "ජෛව රසායනය - Day 06" },
        { id: "ilScRAwcJmI", title: "ජෛව රසායනය - Day 07 Part 1" },
        { id: "9wKjkB0OUCA", title: "ජෛව රසායනය - Day 07 Part 2" }
    ];

    function loadYouTubeAPI() {
        var tag = document.createElement('script');
        tag.src = "https://www.youtube.com/iframe_api";
        var firstScriptTag = document.getElementsByTagName('script')[0];
        firstScriptTag.parentNode.insertBefore(tag, firstScriptTag);
    }

    var player, hideTimeout, currentVidIndex = 0;
    var controlBar = document.getElementById("controlBar");
    var playPauseBtn = document.getElementById("playPauseBtn");

    function onYouTubeIframeAPIReady() {
        player = new YT.Player('player', {
            videoId: videoData[0].id,
            playerVars: { 'autoplay': 0, 'controls': 0, 'modestbranding': 1, 'enablejsapi': 1, 'rel': 0, 'playsinline': 1 },
            events: { 'onReady': onPlayerReady, 'onStateChange': onPlayerStateChange }
        });
    }

    function skipForward(e) {
        e.stopPropagation();
        if (player) player.seekTo(player.getCurrentTime() + 10, true);
    }

    function onPlayerReady() {
        renderPlaylist();
        document.getElementById("nowPlaying").innerText = videoData[0].title;
        setInterval(updateUI, 500);
    }

    function onPlayerStateChange(event) {
        if (event.data === YT.PlayerState.PLAYING) {
            playPauseBtn.innerHTML = '<i class="material-Icons">pause</i>';
            resetTimer();
        } else {
            playPauseBtn.innerHTML = '<i class="material-Icons">play_arrow</i>';
            showControls();
        }
        if (event.data === YT.PlayerState.ENDED && currentVidIndex < videoData.length - 1) loadVideo(currentVidIndex + 1);
    }

    function renderPlaylist() {
        const listDiv = document.getElementById("playlist");
        listDiv.innerHTML = '<div style="font-size:18px; color:#ff0000; font-weight:bold; margin-bottom:15px; border-bottom:1px solid #333; padding-bottom:10px;">පාඩම් මාලාව</div>';
        videoData.forEach((vid, index) => {
            const item = document.createElement("div");
            item.className = "playlist-item" + (index === 0 ? " active" : "");
            item.onclick = () => loadVideo(index);
            item.innerHTML = `<span style="font-weight:bold; margin-right:15px;">${index+1}</span><span>${vid.title}</span>`;
            listDiv.appendChild(item);
        });
    }

    function loadVideo(index) {
        currentVidIndex = index;
        player.loadVideoById(videoData[index].id);
        document.getElementById("nowPlaying").innerText = videoData[index].title;
        document.querySelectorAll(".playlist-item").forEach((el, i) => el.classList.toggle("active", i === index));
        showControls();
    }

    function toggleControls() { controlBar.classList.contains("hidden") ? showControls() : hideControls(); }
    function showControls() { controlBar.classList.remove("hidden"); resetTimer(); }
    function hideControls() { if (player.getPlayerState() === 1) controlBar.classList.add("hidden"); }
    function resetTimer() { clearTimeout(hideTimeout); if (player.getPlayerState() === 1) hideTimeout = setTimeout(hideControls, 4000); }

    function togglePlay(e) { e.stopPropagation(); (player.getPlayerState() === 1) ? player.pauseVideo() : player.playVideo(); }
    function setSpeed(e) { e.stopPropagation(); player.setPlaybackRate(parseFloat(document.getElementById("speed").value)); }
    
    function toggleFullScreen(e) {
        e.stopPropagation();
        var container = document.getElementById("videoContainer");
        if (!document.fullscreenElement) {
            container.requestFullscreen();
            document.getElementById("fullscreenBtn").innerHTML = '<i class="material-Icons">fullscreen_exit</i>';
        } else {
            document.exitFullscreen();
            document.getElementById("fullscreenBtn").innerHTML = '<i class="material-Icons">fullscreen</i>';
        }
    }

    function handleSeek(e) {
        e.stopPropagation();
        var rect = document.getElementById("seekBarContainer").getBoundingClientRect();
        var pos = (e.clientX - rect.left) / rect.width;
        player.seekTo(player.getDuration() * pos, true);
    }

    function updateUI() {
        if (player && player.getCurrentTime) {
            var curr = player.getCurrentTime(), dur = player.getDuration();
            document.getElementById("currentTime").innerText = formatTime(curr);
            document.getElementById("duration").innerText = formatTime(dur);
            if (dur > 0) {
                var p = (curr / dur) * 100;
                document.getElementById("seekBarProgress").style.width = p + "%";
                document.getElementById("seekBarKnob").style.left = p + "%";
            }
        }
    }

    function formatTime(t) {
        t = Math.round(t);
        var m = Math.floor(t / 60), s = t % 60;
        return m + ":" + (s < 10 ? "0" : "") + s;
    }
</script>
</body>
</html>

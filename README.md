<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<title>D@niels Rätsel.de – Multiplayer Quiz & Präsentation</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
body{font-family:Arial; margin:0; padding:0; background:linear-gradient(120deg,#1e3c72,#2a5298); color:white; text-align:center; overflow-x:hidden;}
h1{margin:20px 0;}
.box{background:rgba(0,0,0,0.6); padding:20px; margin:20px auto; width:90%; max-width:400px; border-radius:15px;}
button{padding:15px; margin:10px; border:none; border-radius:10px; font-weight:bold; font-size:16px; cursor:pointer; width:80%; transition:0.3s;}
button:hover{opacity:0.8;}
.hidden{display:none;}
ul{list-style:none; padding-left:0; text-align:left;}
.answer{margin:10px 0; padding:10px; border-radius:10px; background:#3498db; cursor:pointer;}
.answer:hover{background:#2980b9;}
img.presentationImage{width:80%; border-radius:10px; margin-top:10px;}
.infoText{font-size:14px; color:#f1c40f; margin-top:10px;}
#winnerStickman{font-size:50px; margin-top:20px;}
.balloon{position:absolute; width:40px; height:60px; border-radius:50%; animation:fly 5s linear infinite; opacity:0.9;}
@keyframes fly {0%{transform:translateY(100vh);}100%{transform:translateY(-50vh);}}
</style>
</head>
<body>

<h1>D@niels Rätsel.de – Multiplayer Quiz & Präsentation</h1>

<!-- Login -->
<div class="box" id="login">
<h2>Einloggen</h2>
<input id="playerInput" placeholder="Master-Code oder Benutzername"><br><br>
<button onclick="login()">Einloggen</button>
<p id="loginMsg"></p>
<div id="qr"></div>
</div>

<!-- Master Menü -->
<div class="box hidden" id="masterMenu">
<h2>Master-Menü</h2>
<p>Teilnehmer live:</p>
<ul id="playerList"></ul>
<button onclick="startPresentation()">Präsentation starten</button>
</div>

<!-- Präsentation -->
<div class="box hidden" id="presentationBox">
<h2 id="slideTitle"></h2>
<img id="slideImage" class="presentationImage" src="">
<p id="slideText"></p>
<p id="slideSource"></p>
<button id="nextSlideBtn" onclick="nextSlide()">Next</button>
<button id="finishPresentationBtn" class="hidden" onclick="finishPresentation()">Fertig</button>
</div>

<!-- Master Quiz -->
<div class="box hidden" id="masterQuiz">
<h2 id="masterQuestion"></h2>
<div id="masterAnswers"></div>
<p id="masterInfo" class="infoText"></p>
<h3 id="masterPoints">Punkte: 0</h3>
</div>

<!-- Spieler View -->
<div class="box hidden" id="playerView">
<h2 id="playerQuestion">Warte auf den Master...</h2>
<div id="playerAnswers"></div>
<p id="playerInfo" class="infoText"></p>
<p>Dein Benutzername: <span id="yourName"></span></p>
<h3 id="playerPoints">Punkte: 0</h3>
</div>

<!-- Gewinner-Anzeige -->
<div class="box hidden" id="winnerBox">
<h2>🎉 Quiz beendet! 🎉</h2>
<h3>Gewinner: <span id="winnerName"></span></h3>
<h3>Punkte: <span id="winnerPoints"></span></h3>
<div id="winnerStickman"></div>
</div>

<audio id="applause" src="https://www.soundjay.com/human/applause-01.mp3" preload="auto"></audio>

<script>
// Quizfragen mit Info und minimalen Schreibfehlern
let fragen=[
  {frage:"Wer legte die Grundlagen für den Computer?", 
     antworten:["Alan Turing","Bill Gates","Steve Jobs","Mark Zuckerberg"], 
        korrekt:0,
           info:"Alan Turing wurde 1912 geboren, starb 1954. Er inspirierte die Computerentwicklung in Großbritannien."},
             {frage:"Was ist die CPU?", 
                antworten:["Das Gehirn des Computers","Ein Speichergerät","Ein Monitor","Ein Router"], 
                   korrekt:0,
                      info:"Die CPU (Central Processing Unit) führt Berechnungen aus und steuert die Abläufe des Computers."},
                        {frage:"Welches Gerät verbindet dich mit dem Internet?", 
                           antworten:["Router","Maus","Drucker","Lautsprecher"], 
                              korrekt:0,
                                 info:"Router verbinden Computer mit Netzwerken. Ohne sie wäre Internet surfen unmöglich."},
                                   {frage:"Welche Sprache läuft direkt im Browser?", 
                                      antworten:["Python","C++","JavaScript","Java"], 
                                         korrekt:2,
                                            info:"JavaScript ist die Programmiersprache, die direkt im Browser ausgeführt wird."},
                                              {frage:"Was speichert nur kurzzeitig Daten, aber sehr schnell?", 
                                                 antworten:["RAM","Festplatte","CPU","Grafikkarte"], 
                                                    korrekt:0,
                                                       info:"RAM speichert Daten vorübergehend, um schnelle Zugriffe zu ermöglichen."},
                                                         {frage:"Wer schreibt Programme und träumt von Bits?", 
                                                            antworten:["Computerentwickler","Elektriker","Astronaut","Lehrer"], 
                                                               korrekt:0,
                                                                  info:"Computerentwickler erstellen Software, testen Programme und träumen oft von Bits."}
                                                                  ];

                                                                  // Präsentation
                                                                  let presentationSlides=[
                                                                    {title:"Alan Turing – der Pionier", text:"Alan Turing entwickelte die Grundlagen der Computertechnik.", image:"https://upload.wikimedia.org/wikipedia/commons/a/a1/Alan_Turing_Aged_16.jpg", source:"Quelle: Wikipedia"},
                                                                      {title:"CPU – das Gehirn des Computers", text:"Die CPU führt Berechnungen aus und steuert Abläufe.", image:"https://upload.wikimedia.org/wikipedia/commons/4/4f/CPU_Intel_Core_i7.jpg", source:"Quelle: Wikipedia"},
                                                                        {title:"RAM – der Arbeitsspeicher", text:"RAM speichert Daten vorübergehend und ermöglicht schnelle Zugriffe.", image:"https://upload.wikimedia.org/wikipedia/commons/2/2f/DDR3_RAM.jpg", source:"Quelle: Wikipedia"},
                                                                          {title:"Router – Verbindung zum Internet", text:"Router verbinden Computer mit Netzwerken und Internet.", image:"https://upload.wikimedia.org/wikipedia/commons/6/69/TP-Link_Router.jpg", source:"Quelle: Wikipedia"},
                                                                            {title:"Computerentwickler – die Programmierer", text:"Entwickler erstellen Programme, träumen von Bits, trinken Kaffee und erschaffen Software.", image:"https://upload.wikimedia.org/wikipedia/commons/3/35/Software_engineer.jpg", source:"Quelle: Wikipedia"}
                                                                            ];

                                                                            let master=false;
                                                                            let player=false;
                                                                            let username="";
                                                                            let players={};
                                                                            let bc = new BroadcastChannel('dani_quiz');
                                                                            let currentQuestion=0;
                                                                            let currentSlide=0;

                                                                            // Login
                                                                            function login(){
                                                                              let input=document.getElementById("playerInput").value.trim();
                                                                                if(input==="b-daniel519"){
                                                                                    master=true;
                                                                                        document.getElementById("login").classList.add("hidden");
                                                                                            document.getElementById("masterMenu").classList.remove("hidden");
                                                                                                let url=window.location.href;
                                                                                                    document.getElementById("qr").innerHTML="<p>QR-Code für Spieler:</p><img src='https://api.qrserver.com/v1/create-qr-code/?data="+encodeURIComponent(url)+"&size=150x150'>";
                                                                                                      } else if(input.length>0){
                                                                                                          player=true;
                                                                                                              username=input;
                                                                                                                  document.getElementById("yourName").innerText=username;
                                                                                                                      document.getElementById("login").classList.add("hidden");
                                                                                                                          document.getElementById("playerView").classList.remove("hidden");
                                                                                                                              bc.postMessage({type:"join", name:username});
                                                                                                                                } else document.getElementById("loginMsg").innerText="Bitte Master-Code oder Benutzernamen eingeben!";
                                                                                                                                }

                                                                                                                                // Teilnehmerliste
                                                                                                                                function updatePlayerList(){
                                                                                                                                  if(!master) return;
                                                                                                                                    let ul=document.getElementById("playerList");
                                                                                                                                      ul.innerHTML="";
                                                                                                                                        for(let p in players){
                                                                                                                                            let li=document.createElement("li");
                                                                                                                                                li.innerText=p+" ("+players[p]+" Punkte)";
                                                                                                                                                    ul.appendChild(li);
                                                                                                                                                      }
                                                                                                                                                      }

                                                                                                                                                      // BroadcastChannel
                                                                                                                                                      bc.onmessage=function(ev){
                                                                                                                                                        let data=ev.data;
                                                                                                                                                          if(data.type==="join"){
                                                                                                                                                              if(!(data.name in players)) players[data.name]=0;
                                                                                                                                                                  updatePlayerList();
                                                                                                                                                                    }
                                                                                                                                                                      if(player && data.type==="question"){
                                                                                                                                                                          showPlayerQuestion(data.index, data.frage, data.answers, data.info);
                                                                                                                                                                            }
                                                                                                                                                                            }

                                                                                                                                                                            // Präsentation starten
                                                                                                                                                                            function startPresentation(){
                                                                                                                                                                              document.getElementById("masterMenu").classList.add("hidden");
                                                                                                                                                                                document.getElementById("presentationBox").classList.remove("hidden");
                                                                                                                                                                                  currentSlide=0;
                                                                                                                                                                                    showSlide(currentSlide);
                                                                                                                                                                                    }

                                                                                                                                                                                    function showSlide(index){
                                                                                                                                                                                      let slide=presentationSlides[index];
                                                                                                                                                                                        document.getElementById("slideTitle").innerText=slide.title;
                                                                                                                                                                                          document.getElementById("slideText").innerText=slide.text;
                                                                                                                                                                                            document.getElementById("slideImage").src=slide.image;
                                                                                                                                                                                              document.getElementById("slideSource").innerText=slide.source;
                                                                                                                                                                                                document.getElementById("nextSlideBtn").classList.remove("hidden");
                                                                                                                                                                                                  document.getElementById("finishPresentationBtn").classList.add("hidden");
                                                                                                                                                                                                    if(index===presentationSlides.length-1){
                                                                                                                                                                                                        document.getElementById("nextSlideBtn").classList.add("hidden");
                                                                                                                                                                                                            document.getElementById("finishPresentationBtn").classList.remove("hidden");
                                                                                                                                                                                                              }
                                                                                                                                                                                                              }

                                                                                                                                                                                                              function nextSlide(){ currentSlide++; showSlide(currentSlide);}
                                                                                                                                                                                                              function finishPresentation(){document.getElementById("presentationBox").classList.add("hidden"); startQuiz();}

                                                                                                                                                                                                              // Quiz starten
                                                                                                                                                                                                              function startQuiz(){ currentQuestion=0; document.getElementById("masterQuiz").classList.remove("hidden"); sendQuestion(); }

                                                                                                                                                                                                              function shuffleArray(array){return array.sort(()=>Math.random()-0.5);}

                                                                                                                                                                                                              function sendQuestion(){
                                                                                                                                                                                                                let q=fragen[currentQuestion];
                                                                                                                                                                                                                  let shuffledAnswers=shuffleArray(q.antworten.map((a,i)=>({text:a,index:i})));
                                                                                                                                                                                                                    bc.postMessage({type:"question", index:currentQuestion, frage:q.frage, answers:shuffledAnswers, info:q.info});
                                                                                                                                                                                                                      showMasterQuestion(q, shuffledAnswers);
                                                                                                                                                                                                                      }

                                                                                                                                                                                                                      // Master Frage anzeigen
                                                                                                                                                                                                                      function showMasterQuestion(q, shuffledAnswers){
                                                                                                                                                                                                                        document.getElementById("masterQuestion").innerText=q.frage;
                                                                                                                                                                                                                          document.getElementById("masterInfo").innerText=q.info;
                                                                                                                                                                                                                            let container=document.getElementById("masterAnswers");
                                                                                                                                                                                                                              container.innerHTML="";
                                                                                                                                                                                                                                shuffledAnswers.forEach(aObj=>{
                                                                                                                                                                                                                                    let btn=document.createElement("button");
                                                                                                                                                                                                                                        btn.innerText=aObj.text;
                                                                                                                                                                                                                                            btn.className="answer";
                                                                                                                                                                                                                                                if(master && aObj.index===q.korrekt) btn.style.border="3px solid #00ff00";
                                                                                                                                                                                                                                                    btn.onclick=function(){
                                                                                                                                                                                                                                                          for(let p in players){
                                                                                                                                                                                                                                                                  if(aObj.index===q.korrekt) players[p]+=10; // 10 Punkte pro richtig
                                                                                                                                                                                                                                                                        }
                                                                                                                                                                                                                                                                              updatePlayerList();
                                                                                                                                                                                                                                                                                    nextQuestion();
                                                                                                                                                                                                                                                                                        }
                                                                                                                                                                                                                                                                                            container.appendChild(btn);
                                                                                                                                                                                                                                                                                              });
                                                                                                                                                                                                                                                                                              }

                                                                                                                                                                                                                                                                                              // Spielerfrage
                                                                                                                                                                                                                                                                                              function showPlayerQuestion(index, frage, answers, info){
                                                                                                                                                                                                                                                                                                document.getElementById("playerQuestion").innerText=frage;
                                                                                                                                                                                                                                                                                                  document.getElementById("playerInfo").innerText=info;
                                                                                                                                                                                                                                                                                                    let container=document.getElementById("playerAnswers");
                                                                                                                                                                                                                                                                                                      container.innerHTML="";
                                                                                                                                                                                                                                                                                                        answers.forEach(aObj=>{
                                                                                                                                                                                                                                                                                                            let btn=document.createElement("button");
                                                                                                                                                                                                                                                                                                                btn.innerText=aObj.text;
                                                                                                                                                                                                                                                                                                                    btn.className="answer";
                                                                                                                                                                                                                                                                                                                        btn.onclick=function(){
                                                                                                                                                                                                                                                                                                                              if(aObj.index===fragen[index].korrekt) players[username]+=10;
                                                                                                                                                                                                                                                                                                                                    document.getElementById("playerPoints").innerText="Punkte: "+players[username];
                                                                                                                                                                                                                                                                                                                                        }
                                                                                                                                                                                                                                                                                                                                            container.appendChild(btn);
                                                                                                                                                                                                                                                                                                                                              });
                                                                                                                                                                                                                                                                                                                                              }

                                                                                                                                                                                                                                                                                                                                              // Nächste Frage Master
                                                                                                                                                                                                                                                                                                                                              function nextQuestion(){
                                                                                                                                                                                                                                                                                                                                                currentQuestion++;
                                                                                                                                                                                                                                                                                                                                                  if(currentQuestion<fragen.length) sendQuestion();
                                                                                                                                                                                                                                                                                                                                                    else finishQuiz();
                                                                                                                                                                                                                                                                                                                                                    }

                                                                                                                                                                                                                                                                                                                                                    // Quiz beenden
                                                                                                                                                                                                                                                                                                                                                    function finishQuiz(){
                                                                                                                                                                                                                                                                                                                                                      document.getElementById("masterQuiz").classList.add("hidden");
                                                                                                                                                                                                                                                                                                                                                        document.getElementById("winnerBox").classList.remove("hidden");
                                                                                                                                                                                                                                                                                                                                                          let maxPoints=0, winner="";
                                                                                                                                                                                                                                                                                                                                                            for(let p in players){ if(players[p]>maxPoints){ maxPoints=players[p]; winner=p; } }
                                                                                                                                                                                                                                                                                                                                                              document.getElementById("winnerName").innerText=winner;
                                                                                                                                                                                                                                                                                                                                                                document.getElementById("winnerPoints").innerText=maxPoints;

                                                                                                                                                                                                                                                                                                                                                                  let audio=document.getElementById("applause"); audio.play();

                                                                                                                                                                                                                                                                                                                                                                    // Strichmännchen + Emoji
                                                                                                                                                                                                                                                                                                                                                                      let emojiHeads=["❤️","⭐","😎","🤖","👑"];
                                                                                                                                                                                                                                                                                                                                                                        let head=emojiHeads[Math.floor(Math.random()*emojiHeads.length)];
                                                                                                                                                                                                                                                                                                                                                                          document.getElementById("winnerStickman").innerHTML=head+"<br>  O<br> /|\\<br> / \\";

                                                                                                                                                                                                                                                                                                                                                                            // Luftballons
                                                                                                                                                                                                                                                                                                                                                                              for(let i=0;i<15;i++){
                                                                                                                                                                                                                                                                                                                                                                                  let b=document.createElement("div");
                                                                                                                                                                                                                                                                                                                                                                                      b.className="balloon";
                                                                                                                                                                                                                                                                                                                                                                                          b.style.backgroundColor=["#ff0000","#00ff00","#0000ff","#ffff00","#ff00ff"][Math.floor(Math.random()*5)];
                                                                                                                                                                                                                                                                                                                                                                                              b.style.left=Math.random()*90+"%";
                                                                                                                                                                                                                                                                                                                                                                                                  b.style.animationDuration=(3+Math.random()*3)+"s";
                                                                                                                                                                                                                                                                                                                                                                                                      document.body.appendChild(b);
                                                                                                                                                                                                                                                                                                                                                                                                          setTimeout(()=>b.remove(),5000);
                                                                                                                                                                                                                                                                                                                                                                                                            }
                                                                                                                                                                                                                                                                                                                                                                                                            }
                                                                                                                                                                                                                                                                                                                                                                                                            </script>
                                                                                                                                                                                                                                                                                                                                                                                                            </body>
                                                                                                                                                                                                                                                                                                                                                                                                            </html>
# no-fap-tracker
<!DOCTYPE html>
<html lang="bn-BD">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>No-Fap Tracker Mini App</title>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>
<style>
  body { font-family: Arial, sans-serif; background:#f4f4f9; padding:20px; }
  h1 { text-align:center; color:#ff0050; }
  #leaderboard { margin:20px 0; }
  .user { display:flex; align-items:center; margin-bottom:10px; padding:10px; background:white; border-radius:5px; box-shadow:0 2px 5px rgba(0,0,0,0.1); }
  .user img { margin-right:10px; border-radius:50%; }
  .user .name { font-weight:bold; margin-right:10px; }
  .user .badge { margin-right:10px; }
  .user .streak { color:#555; }
  button { padding:10px 20px; margin:5px; border:none; border-radius:5px; cursor:pointer; background:#007bff; color:white; }
  button:hover { background:#0056b3; }
</style>
</head>
<body>

<h1>No-Fap Tracker</h1>

<p id="streak-msg"></p>

<div id="leaderboard"></div>

<button onclick="logToday()">Log Today</button>
<button onclick="shareApp()">Share App</button>
<button onclick="joinChannel()">Join Our Channel</button>

<script>
  // ------------------- Firebase Config -------------------
  const firebaseConfig = {
    apiKey: "AIzaSyCwZ-O1oYNg9-6Lx9jvXCkV7xuedqSXSN4",
    authDomain: "no-fap-tracker1.firebaseapp.com",
    projectId: "no-fap-tracker1",
    storageBucket: "no-fap-tracker1.firebasestorage.app",
    messagingSenderId: "981425818415",
    appId: "1:981425818415:web:2d8989fc5123a6e554afda"
  };
  firebase.initializeApp(firebaseConfig);
  const db = firebase.firestore();

  // ------------------- Telegram User Example -------------------
  const tgUser = {
    id: "123456789",      // Telegram user ID (প্রতিটি ইউজারের জন্য unique)
    first_name: "Rahim"   // Telegram user name
  };

  const today = new Date().toISOString().split("T")[0];

  // ------------------- Badge Logic -------------------
  function getBadge(streak) {
    if(streak >= 100) return "🏆 Platinum";
    if(streak >= 30) return "🥇 Gold";
    if(streak >= 7) return "🔵 Blue";
    if(streak >= 1) return "⚪ Grey";
    return "";
  }

  // ------------------- Firestore User Ref -------------------
  const userRef = db.collection("streaks").doc(tgUser.id);

  // ------------------- Daily Log Function -------------------
  async function logToday() {
    const docSnap = await userRef.get();
    if(docSnap.exists) {
      const data = docSnap.data();
      if(data.lastDate !== today) {
        await userRef.update({
          streak: data.streak + 1,
          lastDate: today
        });
        document.getElementById("streak-msg").innerText = `Congrats! Streak updated to ${data.streak + 1} days.`;
      } else {
        document.getElementById("streak-msg").innerText = `You've already logged today! Current streak: ${data.streak} days.`;
      }
    } else {
      await userRef.set({
        name: tgUser.first_name,
        streak: 1,
        lastDate: today
      });
      document.getElementById("streak-msg").innerText = `Welcome! Streak started: 1 day.`;
    }
  }

  // ------------------- Leaderboard -------------------
  function showLeaderboard() {
    const leaderboardDiv = document.getElementById("leaderboard");
    leaderboardDiv.innerHTML = "";
    db.collection("streaks").orderBy("streak", "desc").limit(10)
      .onSnapshot(snapshot => {
        leaderboardDiv.innerHTML = "";
        snapshot.forEach(doc => {
          const data = doc.data();
          const badge = getBadge(data.streak);
          const div = document.createElement("div");
          div.classList.add("user");
          div.innerHTML = `
            <img src="https://via.placeholder.com/40" alt="${data.name}">
            <span class="name">${data.name}</span>
            <span class="badge">${badge}</span>
            <span class="streak">${data.streak} days</span>
          `;
          leaderboardDiv.appendChild(div);
        });
      });
  }

  showLeaderboard();

  // ------------------- Share App -------------------
  function shareApp() {
    const shareData = {
      title: "No-Fap Tracker",
      text: "Join me in tracking our streaks!",
      url: "https://t.me/cmvbangla"
    };
    navigator.share(shareData).catch(err => console.log(err));
  }

  // ------------------- Join Channel -------------------
  function joinChannel() {
    window.open("https://t.me/cmvbangla", "_blank");
  }
</script>

</body>
</html>

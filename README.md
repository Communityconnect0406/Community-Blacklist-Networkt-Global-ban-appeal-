


<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Appeal Review</title>
  <style>
    body {
      margin: 0;
      background: #0f0f12;
      font-family: "Inter", sans-serif;
      color: #e5e5e5;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      padding: 20px;
    }

    .card {
      width: 100%;
      max-width: 650px;
      background: rgba(255, 255, 255, 0.04);
      border-radius: 18px;
      padding: 28px;
      backdrop-filter: blur(18px);
      box-shadow: 0 0 40px rgba(0,0,0,0.4);
      border: 1px solid rgba(255,255,255,0.08);
    }

    h1 {
      margin: 0 0 10px 0;
      font-size: 1.7rem;
      font-weight: 600;
      letter-spacing: 0.5px;
    }

    .desc {
      font-size: 0.9rem;
      opacity: 0.7;
      margin-bottom: 22px;
    }

    label {
      font-size: 0.85rem;
      opacity: 0.85;
      margin-bottom: 6px;
      display: block;
    }

    input, textarea {
      width: 100%;
      padding: 10px 12px;
      border-radius: 10px;
      border: 1px solid rgba(255,255,255,0.12);
      background: rgba(255,255,255,0.06);
      color: #fff;
      font-size: 0.9rem;
      outline: none;
      transition: 0.15s;
    }

    input:focus, textarea:focus {
      border-color: #6366f1;
      background: rgba(255,255,255,0.12);
    }

    textarea {
      min-height: 90px;
      resize: vertical;
    }

    .btn {
      margin-top: 20px;
      width: 100%;
      padding: 12px;
      border-radius: 10px;
      border: none;
      background: linear-gradient(135deg, #4f46e5, #6366f1);
      color: white;
      font-size: 1rem;
      font-weight: 600;
      cursor: pointer;
      transition: 0.15s;
    }

    .btn:hover {
      transform: translateY(-1px);
      box-shadow: 0 8px 20px rgba(99,102,241,0.4);
    }

    .status {
      margin-top: 16px;
      font-size: 0.85rem;
      opacity: 0.8;
    }

    .score {
      margin-top: 10px;
      font-size: 0.9rem;
      font-weight: 600;
    }

    .accepted {
      color: #4ade80;
    }

    .declined {
      color: #f87171;
    }
  </style>
</head>
<body>

  <div class="card">
    <h1>Ban Appeal</h1>
    <div class="desc">Your appeal will be automatically reviewed and scored by the system.</div>

    <form id="appealForm">
      <label>Discord Username</label>
      <input id="username" required />

      <label style="margin-top: 14px;">Discord ID</label>
      <input id="userId" required />

      <label style="margin-top: 14px;">Reason you were banned</label>
      <textarea id="banReason" required></textarea>

      <label style="margin-top: 14px;">What you did wrong</label>
      <textarea id="whatWrong" required></textarea>

      <label style="margin-top: 14px;">What you will do differently</label>
      <textarea id="whatDifferent" required></textarea>

      <label style="margin-top: 14px;">Why you believe your appeal should be accepted</label>
      <textarea id="whyAccept" required></textarea>

      <label style="margin-top: 14px;">Why staff might decline your appeal</label>
      <textarea id="whyDecline" required></textarea>

      <button class="btn" type="submit">Submit Appeal</button>

      <div id="scoreDisplay" class="score"></div>
      <div id="statusMessage" class="status"></div>
    </form>
  </div>

  <script>
    const WEBHOOK_URL = "https://discord.com/api/webhooks/1487941701489262672/K1KkkDeWYR38ubxGI9wOwBBHy-ZsuVCShvgcdIF9-445CglPqJJmwLkYsYcuHaNOKzoP";

    function scoreAppeal(banReason, whatWrong, whatDifferent) {
      let score = 0;
      const clean = s => (s || "").trim();
      banReason = clean(banReason);
      whatWrong = clean(whatWrong);
      whatDifferent = clean(whatDifferent);

      const totalLength = banReason.length + whatWrong.length + whatDifferent.length;

      if (totalLength > 150) score += 8;
      if (totalLength > 350) score += 8;
      if (totalLength > 700) score += 9;

      const lower = (banReason + whatWrong + whatDifferent).toLowerCase();
      const admits = ["i did", "my fault", "i was wrong", "i take responsibility", "i admit"];
      const blames = ["not my fault", "false ban", "admin fault", "staff fault"];

      admits.forEach(k => { if (lower.includes(k)) score += 5; });
      blames.forEach(k => { if (lower.includes(k)) score -= 6; });

      if (whatWrong.length > 80) score += 8;
      if (whatWrong.length > 200) score += 6;

      const futureWords = ["i will", "i won't", "in the future", "next time", "from now on"];
      futureWords.forEach(k => { if (whatDifferent.toLowerCase().includes(k)) score += 4; });

      if (whatDifferent.length > 80) score += 6;
      if (whatDifferent.length > 200) score += 6;

      let tone = 5;
      const badTone = ["fuck", "idiot", "kys", "trash staff"];
      badTone.forEach(k => { if (lower.includes(k)) tone -= 3; });

      const polite = ["please", "thank you", "sorry"];
      polite.forEach(k => { if (lower.includes(k)) tone += 2; });

      score += Math.max(0, Math.min(10, tone));

      return Math.max(1, Math.min(100, Math.round(score)));
    }

    async function sendToWebhook(payload) {
      return await fetch(WEBHOOK_URL, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(payload)
      });
    }

    document.getElementById("appealForm").addEventListener("submit", async (e) => {
      e.preventDefault();

      const username = document.getElementById("username").value.trim();
      const userId = document.getElementById("userId").value.trim();
      const banReason = document.getElementById("banReason").value.trim();
      const whatWrong = document.getElementById("whatWrong").value.trim();
      const whatDifferent = document.getElementById("whatDifferent").value.trim();
      const whyAccept = document.getElementById("whyAccept").value.trim();
      const whyDecline = document.getElementById("whyDecline").value.trim();

      const score = scoreAppeal(banReason, whatWrong, whatDifferent);
      const accepted = score >= 75;
      const decision = accepted ? "Accepted" : "Declined";

      document.getElementById("scoreDisplay").innerHTML =
        `Score: ${score}/100 — <span class="${accepted ? "accepted" : "declined"}">${decision}</span>`;

      const payload = {
        username: "Appeal System",
        embeds: [
          {
            title: "Ban Appeal Submission",
            color: accepted ? 0x22c55e : 0xef4444,
            fields: [
              { name: "Discord Username", value: username },
              { name: "Discord ID", value: userId },
              { name: "Ban Reason", value: banReason },
              { name: "What They Did Wrong", value: whatWrong },
              { name: "What They'll Do Differently", value: whatDifferent },
              { name: "Why They Want Acceptance", value: whyAccept },
              { name: "Why It Might Be Declined", value: whyDecline },
              { name: "Score", value: `${score}/100` },
              { name: "Decision", value: decision }
            ],
            timestamp: new Date().toISOString()
          }
        ]
      };

      document.getElementById("statusMessage").textContent = "Submitting appeal…";

      try {
        const res = await sendToWebhook(payload);
        if (res.ok) {
          document.getElementById("statusMessage").textContent = "Appeal submitted successfully.";
        } else {
          document.getElementById("statusMessage").textContent =
            "Webhook blocked by browser. Use a backend proxy.";
        }
      } catch {
        document.getElementById("statusMessage").textContent =
          "Unable to send (likely CORS). Use a backend proxy.";
      }
    });
  </script>

</body>
</html>

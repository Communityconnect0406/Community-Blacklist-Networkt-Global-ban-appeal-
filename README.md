



<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Global Ban Appeal</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #0f172a;
      color: #e5e7eb;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      margin: 0;
    }
    .container {
      background: rgba(15, 23, 42, 0.9);
      border-radius: 16px;
      padding: 24px 28px;
      max-width: 500px;
      width: 100%;
      box-shadow: 0 18px 45px rgba(0, 0, 0, 0.6);
      border: 1px solid rgba(148, 163, 184, 0.4);
      backdrop-filter: blur(18px);
    }
    h1 {
      margin-top: 0;
      font-size: 1.4rem;
      text-align: center;
      color: #f9fafb;
    }
    label {
      display: block;
      margin-top: 12px;
      font-size: 0.9rem;
      color: #cbd5f5;
    }
    input, textarea {
      width: 100%;
      margin-top: 6px;
      padding: 8px 10px;
      border-radius: 10px;
      border: 1px solid rgba(148, 163, 184, 0.6);
      background: rgba(15, 23, 42, 0.8);
      color: #e5e7eb;
      font-size: 0.9rem;
      outline: none;
    }
    textarea {
      resize: vertical;
      min-height: 80px;
    }
    button {
      margin-top: 18px;
      width: 100%;
      padding: 10px 0;
      border-radius: 999px;
      border: none;
      background: linear-gradient(135deg, #38bdf8, #6366f1);
      color: #f9fafb;
      font-weight: 600;
      font-size: 0.95rem;
      cursor: pointer;
    }
    .message {
      margin-top: 14px;
      font-size: 0.9rem;
      text-align: center;
    }
    .success { color: #4ade80; }
    .error { color: #f97373; }
  </style>
</head>
<body>
  <div class="container">
    <h1>Global Ban Appeal</h1>
    <form id="appealForm">
      <label for="discordUser">Discord User:</label>
      <input type="text" id="discordUser" required>

      <label for="discordId">Discord ID:</label>
      <input type="text" id="discordId" required>

      <label for="reasonBan">Reason for Global Ban:</label>
      <textarea id="reasonBan" required></textarea>

      <label for="doDifferently">What are you going to do differently?</label>
      <textarea id="doDifferently" required></textarea>

      <button type="submit">Submit Appeal</button>
      <div id="responseMessage" class="message"></div>
    </form>
  </div>

  <script>
    const webhookUrl = "https://discord.com/api/webhooks/1487941701489262672/K1KkkDeWYR38ubxGI9wOwBBHy-ZsuVCShvgcdIF9-445CglPqJJmwLkYsYcuHaNOKzoP";

    // NEW: Severe ban keywords
    const severeKeywords = [
      "dox", "doxxing", "doxing",
      "threat", "death threat",
      "raid", "raiding",
      "child", "minor", "cp",
      "gore", "nsfw", "sexual",
      "terror", "terroristic",
      "malware", "hacking", "hack",
      "ip grabber", "token grabber"
    ];

    function evaluateAppeal(reason, improvement) {
      let accountability = 0;
      let maturity = 0;
      let honesty = 0;
      let risk = 15;
      const flags = [];

      // SEVERE BAN CHECK
      let severe = false;
      severeKeywords.forEach(k => {
        if (reason.toLowerCase().includes(k)) severe = true;
      });

      const denialKeywords = [
        "not my fault", "false", "unfair", "didn't do anything",
        "idk", "dont know", "don't know", "false ban"
      ];

      const apologyKeywords = [
        "sorry", "apologize", "apology", "my fault", "i regret",
        "take responsibility", "responsibility", "i understand",
        "fully understand", "i take full responsibility"
      ];

      const improvementKeywords = [
        "respect", "respect members", "respect staff",
        "won't happen again", "improve", "change", "changed",
        "differently", "better", "second chance", "2nd chance"
      ];

      if (improvement.length > 120) accountability += 10;
      if (improvement.length > 200) accountability += 10;
      if (improvement.length < 50) {
        accountability -= 10;
        flags.push("⚠️ Very short improvement explanation");
      }

      apologyKeywords.forEach(k => {
        if (improvement.toLowerCase().includes(k)) {
          accountability += 6;
          honesty += 6;
        }
      });

      improvementKeywords.forEach(k => {
        if (improvement.toLowerCase().includes(k)) maturity += 6;
      });

      denialKeywords.forEach(k => {
        if (reason.toLowerCase().includes(k)) {
          accountability -= 10;
          honesty -= 10;
          flags.push("🚫 Denial of wrongdoing detected");
        }
      });

      if (reason.length < 40) {
        maturity -= 5;
        flags.push("⚠️ Very short ban reason explanation");
      }

      if (reason.toLowerCase().includes("troll")) {
        risk += 15;
        flags.push("🚨 Possible troll behavior");
      }

      accountability = Math.max(0, Math.min(25, accountability));
      maturity = Math.max(0, Math.min(25, maturity + 10));
      honesty = Math.max(0, Math.min(25, honesty));
      risk = Math.max(0, Math.min(25, risk));

      const total = accountability + maturity + honesty + (25 - risk);

      let confidence =
        (accountability * 1.4) +
        (honesty * 1.6) +
        (maturity * 1.2) -
        (risk * 1.3);

      confidence = Math.max(0, Math.min(100, Math.round(confidence)));

      let recommendation = "DECLINE";
      let evaluationReason = "User shows insufficient accountability or maturity.";

      if (total >= 65 && confidence >= 55) {
        recommendation = "ACCEPT";
        evaluationReason = "User demonstrates strong accountability and improvement.";
      }

      // HARD OVERRIDE FOR SEVERE BANS
      if (severe) {
        recommendation = "DO NOT ACCEPT";
        evaluationReason = "⚫ **DO NOT ACCEPT THIS APPEAL FOR SECURITY REASONS**";
        flags.push("⛔ Severe ban reason detected");
      }

      return {
        accountability,
        maturity,
        honesty,
        risk,
        total,
        confidence,
        recommendation,
        evaluationReason,
        flags,
        severe
      };
    }

    document.getElementById("appealForm").addEventListener("submit", async (e) => {
      e.preventDefault();

      const discordUser = document.getElementById("discordUser").value.trim();
      const discordId = document.getElementById("discordId").value.trim();
      const reasonBan = document.getElementById("reasonBan").value.trim();
      const doDifferently = document.getElementById("doDifferently").value.trim();
      const responseMessage = document.getElementById("responseMessage");

      const evalResult = evaluateAppeal(reasonBan, doDifferently);

      const flagsText = evalResult.flags.length > 0 ? evalResult.flags.join("\n• ") : "None detected";

      const payload = {
        content: "<@&1487941139649790063> New global ban appeal submitted.",
        embeds: [
          {
            title: "📌 Global Ban Appeal Submitted",
            color: evalResult.severe ? 0x000000 : (evalResult.recommendation === "ACCEPT" ? 0x2ecc71 : 0xe74c3c),
            fields: [
              {
                name: "👤 User Information",
                value: `• **Username:** ${discordUser}\n• **Discord ID:** ${discordId}`
              },
              {
                name: "📘 Reason for Ban",
                value: reasonBan
              },
              {
                name: "🛠 What They Will Do Differently",
                value: doDifferently
              },
              {
                name: "📊 System Evaluation",
                value:
                  `• **Accountability:** ${evalResult.accountability}/25\n` +
                  `• **Maturity:** ${evalResult.maturity}/25\n` +
                  `• **Honesty:** ${evalResult.honesty}/25\n` +
                  `• **Risk Score:** ${evalResult.risk}/25\n` +
                  `• **Overall Appeal Score:** ${evalResult.total}/100`
              },
              {
                name: "🔍 Genuine Appeal Confidence",
                value: `${evalResult.confidence}%`
              },
              {
                name: "🧩 Flags Detected",
                value: `• ${flagsText}`
              },
              {
                name: "🧠 System Recommendation",
                value: evalResult.evaluationReason
              }
            ],
            timestamp: new Date().toISOString()
          }
        ]
      };

      try {
        const res = await fetch(webhookUrl, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(payload)
        });

        if (res.ok) {
          responseMessage.textContent =
            "Thank you for submitting an appeal. You will get a response shortly from one of our staff members. Please make sure to keep your DMs open!";
          responseMessage.classList.add("success");
          e.target.reset();
        } else {
          responseMessage.textContent = "There was an error submitting your appeal.";
          responseMessage.classList.add("error");
        }
      } catch {
        responseMessage.textContent = "There was an error submitting your appeal.";
        responseMessage.classList.add("error");
      }
    });
  </script>
</body>
</html>

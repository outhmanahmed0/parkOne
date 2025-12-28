<script>
  import { onMount } from "svelte";
  import "https://cdn.marmot-cloud.com/npm/hylid-bridge/2.10.0/index.js";

  let authCode = $state("");
  let token = $state("");
  let scannedCode = $state("");
  let baghdadTime = $state("");

  onMount(() => {
    const updateBaghdadTime = () => {
      const now = new Date();
      const baghdadTz = new Intl.DateTimeFormat('en-US', {
        timeZone: 'Asia/Baghdad',
        hour: '2-digit',
        minute: '2-digit',
        second: '2-digit',
        hour12: false
      }).format(now);
      baghdadTime = baghdadTz;
    };
    
    updateBaghdadTime();
    const interval = setInterval(updateBaghdadTime, 1000);

    return () => clearInterval(interval);
  });

  function auth() {
    console.log("Auth button clicked"); 

    if (typeof my === "undefined") {
      alert(
        "Error: Hylid bridge (my) is not loaded. Make sure you're running this in a mini app environment.",
      );
      return;
    }

    my.getAuthCode({
      scopes: ["auth_base", "USER_ID"],
      success: (res) => {
        authCode = res.authCode;
        console.log("Auth code received:", authCode);

        fetch("https://its.mouamle.space/api/auth-with-superQi", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            token: authCode,
          }),
        })
          .then((res) => res.json())
          .then((data) => {
            token = data.token;
            console.log("Token received:", token);
            my.alert({
              content: "Login successful",
            });
          })
          .catch((err) => {
            console.error("Auth error:", err);
            let errorDetails = "";
            if (err && typeof err === "object") {
              errorDetails = JSON.stringify(err, null, 2);
            } else {
              errorDetails = String(err);
            }
            my.alert({
              content: "Error: " + errorDetails,
            });
          });
      },
      fail: (res) => {
        console.error("Auth failed:", res.authErrorScopes);
        my.alert({
          content:
            "Authentication failed: " + JSON.stringify(res.authErrorScopes),
        });
      },
    });
  }

  function pay() {
    if (!token) {
      my.alert({
        content: "Please authenticate first",
      });
      return;
    }

    fetch("https://its.mouamle.space/api/payment", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: token,
      },
    })
      .then((res) => res.json())
      .then((data) => {
        my.tradePay({
          paymentUrl: data.url,
          success: (res) => {
            my.alert({
              content: "Payment successful",
            });
          },
        });
      })
      .catch((err) => {
        console.error("Payment error:", err);
        my.alert({
          content: "Payment failed: " + String(err),
        });
      });
  }

  function copyAuthCode() {
    if (!authCode) {
      my.alert({
        content: "No auth code to copy",
      });
      return;
    }
    navigator.clipboard.writeText(authCode);
    my.alert({
      content: "Auth code copied!",
    });
  }

  function scan() {
    console.log("Scan button clicked"); 
    if (typeof my === "undefined") {
      alert(
        "Error: Hylid bridge (my) is not loaded. Make sure you're running this in a mini app environment.",
      );
      return;
    }

    my.scan({
      type: "qr",
      success: (res) => {
        console.log("QR code scanned:", res.code);
        scannedCode = res.code; 
        my.alert({
          title: "Scanned Code",
          content: res.code,
        });
      },
      fail: (err) => {
        console.error("Scan failed:", err);
        my.alert({
          title: "Scan Failed",
          content: "Failed to scan QR code",
        });
      },
    });
  }
</script>

<div class="header">
  <div class="header-content">
    <div class="time-display">
      <span class="time-label">Iraq Time</span>
      <span class="time-value">{baghdadTime || "--:--:--"}</span>
    </div>
  </div>
</div>

<main>
  <div class="button-container">
    <button class="btn btn-auth" onclick={auth}> Auth </button>
    <button class="btn btn-scan" onclick={scan}> Scan QR Code </button>
    <button class="btn btn-pay" onclick={pay}> Pay </button>
  </div>
</main>

<style>
  :global(:root){
    --bg-1: #eae0cf;
    --bg-2: #eae0cf;
    --card: rgba(100,116,139,0.05);
    --muted: #64748b;
    --accent: #3b82f6;
    --accent-2: #8b5cf6;
    --success: #10b981;
    --radius: 12px;
  }

  :global(body) {
    margin: 0;
    padding: 0;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen,
      Ubuntu, Cantarell, sans-serif;
    background: #eae0cf;
    color: #1e293b;
    -webkit-font-smoothing:antialiased;
    -moz-osx-font-smoothing:grayscale;
    -webkit-text-size-adjust: 100%;
  }

  .header {
    position: fixed;
    top: 20px;
    left: 50%;
    transform: translateX(-50%);
    background: linear-gradient(135deg,rgba(218,197,160,0.5),rgba(218,197,160,0.4));
    padding: 12px 24px;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 16px;
    backdrop-filter: blur(8px) saturate(140%);
    border: 1px solid rgba(218,197,160,0.6);
    box-shadow: 0 4px 12px rgba(15,23,42,0.08);
    z-index: 1000;
    max-width: 520px;
    width: 90%;
  }

  .header-content { display:flex;gap:28px;align-items:center }

  .time-display, .scanned-display { display:flex;flex-direction:column;gap:4px;align-items:center }

  .time-label { font-size:0.7rem;color:#64748b;text-transform:uppercase;letter-spacing:0.08em;font-weight:600 }

  .time-value { font-size:1.1rem;color:#3b82f6;font-weight:800;font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, "Roboto Mono", monospace;letter-spacing:0.02em }

  .code-value { font-size:0.9rem;color:#1e40af;font-weight:600;font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, "Roboto Mono", monospace;max-width:150px;overflow:hidden;text-overflow:ellipsis;white-space:nowrap }

  .code-value.placeholder { color:#94a3b8;font-weight:400;font-style:italic }

  main{min-height:100vh;display:flex;align-items:center;justify-content:center;padding:100px 16px 30px}

  .button-container{display:flex;flex-direction:column;gap:14px;max-width:360px;width:100%;min-width:280px}

  .btn{color:white;padding:12px 18px;border-radius:12px;font-size:1.05rem;font-weight:700;border:none;cursor:pointer;transition:transform .16s ease,box-shadow .16s ease;box-shadow:0 4px 12px rgba(15,23,42,0.15);width:100%}
  .btn:hover{transform:translateY(-3px);box-shadow:0 12px 24px rgba(15,23,42,0.2)}
  .btn:active{transform:translateY(0)}

  .btn-auth{background:linear-gradient(90deg,#3b82f6,#2563eb)}
  .btn-scan{background:linear-gradient(90deg,#10b981,#059669)}
  .btn-pay{background:linear-gradient(90deg,#8b5cf6,#7c3aed)}

  .btn-auth:hover{filter:brightness(0.95)}
  .btn-scan:hover{filter:brightness(0.95)}
  .btn-pay:hover{filter:brightness(0.95)}

  @media (max-width:640px){
    .header {
      top: 12px;
      width: 95%;
      max-width: calc(100% - 24px);
      padding: 10px 16px;
      border-radius: 14px;
    }

    .time-label { font-size:0.65rem;letter-spacing:0.07em }
    .time-value { font-size:1rem }

    main{padding:90px 12px 20px}

    .button-container{gap:12px;max-width:100%}
    .btn{padding:11px 16px;font-size:0.95rem;border-radius:10px}
  }

  @media (max-width:480px){
    .header {
      top: 10px;
      width: 92%;
      padding: 8px 14px;
    }

    .time-label { font-size:0.6rem;letter-spacing:0.06em }
    .time-value { font-size:0.95rem }

    main{padding:80px 10px 16px;min-height:100vh}

    .button-container{gap:10px}
    .btn{padding:10px 14px;font-size:0.9rem;border-radius:10px}
  }

  @media (max-width:375px){
    .header {
      top: 8px;
      width: 90%;
      padding: 8px 12px;
    }

    .time-label { font-size:0.58rem;letter-spacing:0.05em }
    .time-value { font-size:0.9rem }

    main{padding:75px 8px 12px}

    .button-container{gap:8px}
    .btn{padding:9px 12px;font-size:0.85rem;border-radius:9px}
  }
</style>

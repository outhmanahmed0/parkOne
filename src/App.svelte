<script>
  import { onMount } from "svelte";
  import "https://cdn.marmot-cloud.com/npm/hylid-bridge/2.10.0/index.js";

  let authCode = $state("");
  let token = $state("");
  let scannedCode = $state("");
  let secondsOnPage = $state(0);

  // Start timer when component mounts
  onMount(() => {
    const interval = setInterval(() => {
      secondsOnPage++;
    }, 1000);

    return () => clearInterval(interval);
  });

  // Format seconds into HH:MM:SS
  function formatTime(seconds) {
    const hours = Math.floor(seconds / 3600);
    const minutes = Math.floor((seconds % 3600) / 60);
    const secs = seconds % 60;
    return `${String(hours).padStart(2, "0")}:${String(minutes).padStart(2, "0")}:${String(secs).padStart(2, "0")}`;
  }

  function auth() {
    console.log("Auth button clicked"); // Debug log

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
    console.log("Scan button clicked"); // Debug log

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
        scannedCode = res.code; // Save the scanned code
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
  <div class="info-item">
    <span class="label">Scanned Code:</span>
    <span class="value {scannedCode ? '' : 'placeholder'}"
      >{scannedCode || "Scan a QR code to see it here"}</span
    >
  </div>
  <div class="info-item">
    <span class="label">Time on Page:</span>
    <span class="value">{formatTime(secondsOnPage)}</span>
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
  :global(body) {
    margin: 0;
    padding: 0;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen,
      Ubuntu, Cantarell, sans-serif;
  }

  .header {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    background-color: #1a1a1a;
    padding: 1rem 2rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.5);
    z-index: 1000;
  }

  .info-item {
    display: flex;
    flex-direction: column;
    gap: 0.25rem;
  }

  .label {
    font-size: 0.75rem;
    color: #888;
    text-transform: uppercase;
    letter-spacing: 0.05em;
  }

  .value {
    font-size: 1rem;
    color: #fff;
    font-weight: 600;
    font-family: "Courier New", monospace;
  }

  .value.placeholder {
    color: #666;
    font-style: italic;
    font-weight: 400;
  }

  main {
    min-height: 100vh;
    background-color: #000000;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .button-container {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  .btn {
    color: white;
    padding: 0.75rem 1.5rem;
    border-radius: 0.5rem;
    font-size: 1.125rem;
    font-weight: 600;
    border: none;
    cursor: pointer;
    transition: all 0.3s ease;
    box-shadow:
      0 10px 15px -3px rgba(0, 0, 0, 0.1),
      0 4px 6px -2px rgba(0, 0, 0, 0.05);
  }

  .btn:hover {
    transform: translateY(-2px);
    box-shadow:
      0 20px 25px -5px rgba(0, 0, 0, 0.1),
      0 10px 10px -5px rgba(0, 0, 0, 0.04);
  }

  .btn:active {
    transform: translateY(0);
  }

  .btn-auth {
    background-color: #3b82f6;
  }

  .btn-auth:hover {
    background-color: #2563eb;
  }

  .btn-scan {
    background-color: #10b981;
  }

  .btn-scan:hover {
    background-color: #059669;
  }

  .btn-pay {
    background-color: #8b5cf6;
  }

  .btn-pay:hover {
    background-color: #7c3aed;
  }
</style>

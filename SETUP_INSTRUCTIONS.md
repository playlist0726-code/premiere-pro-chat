# 🎬 Claude Code로 자동 설정하기

## 📋 Claude Code에 이 명령어를 복사-붙여넣기하세요:

```
당신은 Premiere Pro 내부에 Claude AI 채팅 패널을 만드는 Node.js 백엔드 개발자입니다.

다음 작업을 정확히 순서대로 수행하세요:

## 1단계: 프로젝트 폴더 생성 및 초기화
- 새 폴더 `premiere-chat-plugin` 생성
- 이 폴더를 작업 디렉토리로 설정

## 2단계: package.json 파일 생성
다음 내용으로 `package.json` 파일을 생성:
{
  "name": "premiere-pro-chat",
  "version": "1.0.0",
  "type": "module",
  "description": "Claude AI Chat Panel for Adobe Premiere Pro",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "node --watch server.js",
    "build": "webpack"
  },
  "dependencies": {
    "@anthropic-ai/sdk": "^0.24.0",
    "cors": "^2.8.5",
    "express": "^4.18.2",
    "dotenv": "^16.3.1"
  },
  "devDependencies": {
    "@babel/core": "^7.22.0",
    "@babel/preset-react": "^7.22.0",
    "babel-loader": "^9.1.0",
    "webpack": "^5.88.0",
    "webpack-cli": "^5.1.0"
  }
}

## 3단계: .env 파일 생성
다음 내용으로 `.env` 파일을 생성:
CLAUDE_API_KEY=your-api-key-here
PORT=3001

## 4단계: src 폴더 생성
`src` 디렉토리 생성

## 5단계: src/index.html 파일 생성
다음 내용으로 `src/index.html` 파일을 생성:

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8" />
    <title>Claude Chat for Premiere Pro</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
            background: #1e1e1e;
            color: #fff;
            height: 100vh;
            display: flex;
        }
        
        #root {
            width: 100%;
            height: 100%;
            display: flex;
            flex-direction: column;
        }
        
        .chat-container {
            display: flex;
            flex-direction: column;
            height: 100%;
            padding: 12px;
        }
        
        .chat-header {
            padding: 12px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            border-radius: 8px;
            margin-bottom: 12px;
            text-align: center;
            font-weight: bold;
        }
        
        .chat-messages {
            flex: 1;
            overflow-y: auto;
            margin-bottom: 12px;
            padding: 12px;
            background: #2d2d2d;
            border-radius: 8px;
            border: 1px solid #444;
        }
        
        .message {
            margin: 8px 0;
            padding: 10px 12px;
            border-radius: 8px;
            word-wrap: break-word;
            font-size: 13px;
            line-height: 1.4;
        }
        
        .message.user {
            background: #0078d4;
            margin-left: 20px;
            text-align: right;
        }
        
        .message.assistant {
            background: #3d3d3d;
            margin-right: 20px;
            border-left: 3px solid #667eea;
        }
        
        .message.loading {
            color: #888;
            font-style: italic;
        }
        
        .chat-input-area {
            display: flex;
            gap: 8px;
            padding: 12px;
            background: #2d2d2d;
            border-radius: 8px;
            border: 1px solid #444;
        }
        
        .chat-input-area input {
            flex: 1;
            padding: 10px 12px;
            background: #1e1e1e;
            color: #fff;
            border: 1px solid #444;
            border-radius: 6px;
            font-size: 13px;
        }
        
        .chat-input-area input:focus {
            outline: none;
            border-color: #667eea;
            box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
        }
        
        .chat-input-area button {
            padding: 10px 16px;
            background: #667eea;
            color: #fff;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            font-weight: 600;
            font-size: 13px;
            transition: background 0.2s;
        }
        
        .chat-input-area button:hover:not(:disabled) {
            background: #764ba2;
        }
        
        .chat-input-area button:disabled {
            opacity: 0.5;
            cursor: not-allowed;
        }
        
        .error {
            color: #ff6b6b;
            padding: 8px;
            background: rgba(255, 107, 107, 0.1);
            border-radius: 4px;
            margin: 8px 0;
        }
    </style>
</head>
<body>
    <div id="root"></div>
    <script src="index.js"></script>
</body>
</html>

## 6단계: src/index.js 파일 생성
다음 내용으로 `src/index.js` 파일을 생성:

const API_URL = "http://localhost:3001";

let messages = [
    { role: "assistant", content: "안녕! 나는 Claude 어시스턴트야. 프리미어 프로 작업을 도와줄 수 있어. 무엇을 할까?" }
];

function renderUI() {
    const root = document.getElementById("root");
    root.innerHTML = \`
        <div class="chat-container">
            <div class="chat-header">🎬 Claude Chat for Premiere Pro</div>
            <div class="chat-messages" id="messages"></div>
            <div class="chat-input-area">
                <input type="text" id="input" placeholder="메시지 입력..." />
                <button id="send-btn">전송</button>
            </div>
        </div>
    \`;
    
    renderMessages();
    setupEventListeners();
}

function renderMessages() {
    const messagesDiv = document.getElementById("messages");
    messagesDiv.innerHTML = messages.map((msg, idx) => \`
        <div class="message \${msg.role}">
            \${msg.content}
        </div>
    \`).join("");
    messagesDiv.scrollTop = messagesDiv.scrollHeight;
}

function setupEventListeners() {
    const input = document.getElementById("input");
    const sendBtn = document.getElementById("send-btn");
    
    sendBtn.addEventListener("click", sendMessage);
    input.addEventListener("keypress", (e) => {
        if (e.key === "Enter") sendMessage();
    });
}

async function sendMessage() {
    const input = document.getElementById("input");
    const text = input.value.trim();
    
    if (!text) return;
    
    messages.push({ role: "user", content: text });
    input.value = "";
    renderMessages();
    
    const sendBtn = document.getElementById("send-btn");
    sendBtn.disabled = true;
    sendBtn.textContent = "...";
    
    try {
        const response = await fetch(\`\${API_URL}/api/chat\`, {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ message: text, history: messages })
        });
        
        if (!response.ok) {
            throw new Error(\`서버 오류: \${response.status}\`);
        }
        
        const data = await response.json();
        messages.push({ role: "assistant", content: data.reply });
        renderMessages();
        
    } catch (error) {
        messages.push({ 
            role: "assistant", 
            content: \`❌ 오류: \${error.message}\\n\\n백엔드 서버가 실행 중인지 확인하세요.\` 
        });
        renderMessages();
    }
    
    sendBtn.disabled = false;
    sendBtn.textContent = "전송";
}

renderUI();

## 7단계: server.js 파일 생성
다음 내용으로 `server.js` 파일을 생성:

import Anthropic from "@anthropic-ai/sdk";
import cors from "cors";
import express from "express";
import dotenv from "dotenv";

dotenv.config();

const app = express();
app.use(cors());
app.use(express.json());

const client = new Anthropic({
    apiKey: process.env.CLAUDE_API_KEY
});

app.post("/api/chat", async (req, res) => {
    try {
        const { message, history } = req.body;
        
        if (!message) {
            return res.status(400).json({ error: "메시지가 없습니다" });
        }
        
        const messages = history.map(m => ({
            role: m.role,
            content: m.content
        }));
        
        const response = await client.messages.create({
            model: "claude-3-5-sonnet-20241022",
            max_tokens: 1024,
            system: \`당신은 Adobe Premiere Pro 전문가 어시스턴트입니다.
사용자의 비디오 편집 질문에 답변하고, 친절하고 간결하게 설명해주세요.
Premiere Pro의 기능, 단축키, 워크플로우 등을 도와줄 수 있습니다.\`,
            messages: messages
        });
        
        const reply = response.content[0].text;
        
        res.json({
            reply: reply,
            success: true
        });
        
    } catch (error) {
        console.error("Error:", error);
        res.status(500).json({
            error: error.message,
            success: false
        });
    }
});

app.get("/health", (req, res) => {
    res.json({ status: "ok" });
});

const PORT = process.env.PORT || 3001;
app.listen(PORT, () => {
    console.log(\`🚀 Claude Chat 서버 실행: http://localhost:\${PORT}\`);
    console.log(\`📌 Premiere Pro 패널에서 이 서버와 연결됩니다\`);
    console.log(\`⚠️  Claude API 키가 .env 파일에 설정되어 있는지 확인하세요!\`);
});

## 8단계: plugin.xml 파일 생성
다음 내용으로 `plugin.xml` 파일을 생성:

<?xml version="1.0" encoding="UTF-8"?>
<ExtensionManifest xmlns="http://ns.adobe.com/ExtensionManifest/1.0">
  <ExtensionBundleVersion>1.0</ExtensionBundleVersion>
  <ExtensionBundleId>com.example.premierechat</ExtensionBundleId>
  <Extension Id="com.example.premierechat.panel">
    <Type>Panel</Type>
    <ScriptPath>./src/index.html</ScriptPath>
    <UI>
      <Type>Panel</Type>
      <Geometry>
        <Size>
          <Width>400</Width>
          <Height>600</Height>
        </Size>
      </Geometry>
    </UI>
  </Extension>
</ExtensionManifest>

## 9단계: .gitignore 파일 생성
다음 내용으로 `.gitignore` 파일을 생성:

node_modules/
.env
dist/
*.log
.DS_Store

## 10단계: npm 패키지 설치
터미널에서 다음 명령어 실행:
npm install

## 11단계: README.md 파일 생성
다음 내용으로 `README.md` 파일을 생성:

# 🎬 Claude Chat Panel for Premiere Pro

Premiere Pro 내부에 Claude AI 채팅 기능을 추가하는 플러그인입니다.

## 설치 방법

1. npm 패키지 설치
\`\`\`bash
npm install
\`\`\`

2. Claude API 키 설정
\`\`\`bash
export CLAUDE_API_KEY="your-api-key-from-console.anthropic.com"
\`\`\`

3. 서버 시작
\`\`\`bash
npm start
\`\`\`

4. Premiere Pro에서 Window > Extensions > Claude Chat 열기

## 기능
- Premiere Pro 내부 채팅 인터페이스
- Claude AI를 통한 실시간 답변
- 프리미어 프로 편집 팁 및 조언
- 단축키 및 워크플로우 상담

## 트러블슈팅
- 플러그인이 보이지 않으면 Premiere Pro 재시작
- 서버 연결 오류 시 http://localhost:3001/health 확인
- API 키 오류 시 .env 파일 재확인

## 최종 완료
모든 파일이 생성되었습니다! 이제:
1. npm install 실행
2. CLAUDE_API_KEY 환경 변수 설정
3. npm start 로 서버 시작
4. Premiere Pro 재시작 후 플러그인 활성화

준비 완료! 🚀
```

---

## ✅ **Claude Code에 전달하는 방법:**

1. **Claude Code 열기**
2. **위의 전체 텍스트 복사** (```로 시작하는 부분부터 끝까지)
3. Claude Code 채팅창에 **붙여넣기**
4. **Enter 누르기**

Claude Code가 **자동으로:**
- ✅ 폴더 생성
- ✅ 모든 파일 생성
- ✅ 코드 작성
- ✅ npm install 실행
- ✅ 설정 완료

---

## 🎉 완료 후 실행:

```bash
# 터미널 1
export CLAUDE_API_KEY="your-api-key-here"
npm start

# 터미널 2 (선택사항)
npm run dev
```

**이제 Premiere Pro를 재시작하고 Window > Extensions 에서 플러그인을 찾으면 됩니다!** 🚀

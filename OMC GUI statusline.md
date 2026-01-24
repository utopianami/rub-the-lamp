# OMC GUI Statusline 설치해줘

아래 단계대로 설치해줘:

## 1. 스크립트 파일 생성

`~/.claude/hooks/gui-status.mjs` 파일 생성:

```javascript
#!/usr/bin/env node
import { existsSync, readFileSync, readdirSync, statSync } from 'fs';
import { homedir } from 'os';
import { join } from 'path';
import { execSync } from 'child_process';
import https from 'https';

const CHARS_PER_TOKEN = 4;
const CONTEXT_LIMIT = 200000;

function getAccessToken() {
  if (process.platform === 'darwin') {
    try {
      const result = execSync('/usr/bin/security find-generic-password -s "Claude Code-credentials" -w 2>/dev/null', { encoding: 'utf-8', timeout: 2000 }).trim();
      if (result) { const parsed = JSON.parse(result); const creds = parsed.claudeAiOauth || parsed; if (creds.accessToken) return creds.accessToken; }
    } catch {}
  }
  try {
    const credPath = join(homedir(), '.claude/.credentials.json');
    if (existsSync(credPath)) { const content = readFileSync(credPath, 'utf-8'); const parsed = JSON.parse(content); const creds = parsed.claudeAiOauth || parsed; if (creds.accessToken) return creds.accessToken; }
  } catch {}
  return null;
}

function fetchRateLimits(accessToken) {
  return new Promise((resolve) => {
    const req = https.request({ hostname: 'api.anthropic.com', path: '/api/oauth/usage', method: 'GET', headers: { 'Authorization': `Bearer ${accessToken}`, 'anthropic-beta': 'oauth-2025-04-20', 'Content-Type': 'application/json' }, timeout: 5000 }, (res) => {
      let data = ''; res.on('data', (chunk) => { data += chunk; }); res.on('end', () => { if (res.statusCode === 200) { try { resolve(JSON.parse(data)); } catch { resolve(null); } } else { resolve(null); } });
    });
    req.on('error', () => resolve(null)); req.on('timeout', () => { req.destroy(); resolve(null); }); req.end();
  });
}

function formatTimeRemaining(resetsAt) {
  if (!resetsAt) return '';
  const diffMs = new Date(resetsAt) - new Date();
  if (diffMs <= 0) return '0m';
  const diffMins = Math.floor(diffMs / 60000), diffHours = Math.floor(diffMins / 60), diffDays = Math.floor(diffHours / 24);
  if (diffDays > 0) return `${diffDays}d${diffHours % 24}h`;
  if (diffHours > 0) return `${diffHours}h${diffMins % 60}m`;
  return `${diffMins}m`;
}

function findCurrentTranscript() {
  const projectsDir = join(homedir(), '.claude/projects');
  if (!existsSync(projectsDir)) return null;
  const cwd = process.cwd(), cwdHash = '-' + cwd.replace(/\//g, '-').substring(1);
  const projectDir = join(projectsDir, cwdHash);
  if (existsSync(projectDir)) return findLatestTranscript(projectDir);
  try { for (const d of readdirSync(projectsDir)) { const dirPath = '/' + d.substring(1).replace(/-/g, '/'); if (cwd.startsWith(dirPath) || dirPath.startsWith(cwd)) return findLatestTranscript(join(projectsDir, d)); } } catch {}
  return null;
}

function findLatestTranscript(projectDir) {
  try { const files = readdirSync(projectDir).filter(f => f.endsWith('.jsonl')).map(f => ({ path: join(projectDir, f), mtime: statSync(join(projectDir, f)).mtime })).sort((a, b) => b.mtime - a.mtime); return files.length > 0 ? files[0].path : null; } catch { return null; }
}

function parseTranscript(transcriptPath) {
  const result = { model: null, sessionStart: null, agents: { running: 0, total: 0 }, todos: { completed: 0, total: 0 }, tokenEstimate: 0, thinking: false, thinkingLastSeen: null };
  if (!transcriptPath || !existsSync(transcriptPath)) return result;
  try {
    const lines = readFileSync(transcriptPath, 'utf-8').split('\n').filter(l => l.trim());
    let textLength = 0; const agentMap = new Map(); let latestTodos = [];
    for (const line of lines) {
      try {
        const entry = JSON.parse(line);
        if (!result.model) { if (entry.model) result.model = entry.model; else if (entry.message?.model) result.model = entry.message.model; }
        if (!result.sessionStart && entry.timestamp) result.sessionStart = new Date(entry.timestamp);
        const msgContent = entry.message?.content;
        if (Array.isArray(msgContent)) {
          for (const block of msgContent) {
            if (block.type === 'thinking' || block.type === 'reasoning') { result.thinking = true; result.thinkingLastSeen = new Date(entry.timestamp); }
            if (block.type === 'text' && block.text) textLength += block.text.length;
            if (block.type === 'tool_use' && block.name === 'Task') agentMap.set(block.id, { status: 'running' });
            if (block.type === 'tool_result' && agentMap.has(block.tool_use_id)) { if (!(typeof block.content === 'string' && block.content.includes('Async agent launched'))) agentMap.get(block.tool_use_id).status = 'completed'; }
            if (block.type === 'tool_use' && block.name === 'TodoWrite' && block.input?.todos) latestTodos = block.input.todos;
          }
        }
      } catch {}
    }
    result.tokenEstimate = Math.ceil(textLength / CHARS_PER_TOKEN);
    result.agents.total = agentMap.size; result.agents.running = Array.from(agentMap.values()).filter(a => a.status === 'running').length;
    result.todos.total = latestTodos.length; result.todos.completed = latestTodos.filter(t => t.status === 'completed').length;
  } catch {}
  return result;
}

function formatModel(m) {
  if (!m) return '?';
  if (m.includes('opus-4-5')) return 'opus 4.5'; if (m.includes('opus-4')) return 'opus 4'; if (m.includes('sonnet-4')) return 'sonnet 4';
  if (m.includes('3-5-sonnet')) return 'sonnet 3.5'; if (m.includes('3-5-haiku')) return 'haiku 3.5';
  if (m.includes('haiku')) return 'haiku'; if (m.includes('sonnet')) return 'sonnet'; if (m.includes('opus')) return 'opus';
  return m.substring(0, 10);
}

async function main() {
  const parts = ['[OMC]'];
  const transcript = parseTranscript(findCurrentTranscript());
  parts.push(formatModel(transcript.model));
  if (transcript.thinking && transcript.thinkingLastSeen && Date.now() - transcript.thinkingLastSeen.getTime() < 30000) parts.push('thinking');
  const token = getAccessToken();
  if (token) { const limits = await fetchRateLimits(token); if (limits) { parts.push(`5h:${Math.round(limits.five_hour?.utilization ?? 0)}%${formatTimeRemaining(limits.five_hour?.resets_at) ? `(${formatTimeRemaining(limits.five_hour?.resets_at)})` : ''}`); parts.push(`wk:${Math.round(limits.seven_day?.utilization ?? 0)}%${formatTimeRemaining(limits.seven_day?.resets_at) ? `(${formatTimeRemaining(limits.seven_day?.resets_at)})` : ''}`); } }
  const diffMs = transcript.sessionStart ? Date.now() - transcript.sessionStart.getTime() : 0, diffMins = Math.floor(diffMs / 60000), diffHours = Math.floor(diffMins / 60);
  parts.push(`session:${diffHours > 0 ? `${diffHours}h${diffMins % 60}m` : `${diffMins}m`}`);
  parts.push(`ctx:${Math.min(Math.round((transcript.tokenEstimate / CONTEXT_LIMIT) * 100), 100)}%`);
  if (transcript.agents.running > 0) parts.push(`agents:${transcript.agents.running}`);
  if (transcript.todos.total > 0) parts.push(`todos:${transcript.todos.completed}/${transcript.todos.total}`);
  console.log(parts.join(' | '));
}
main().catch(() => console.log('[OMC] | error'));
```

## 2. 실행 권한 부여

```bash
chmod +x ~/.claude/hooks/gui-status.mjs
```

## 3. settings.json에 훅 추가

`~/.claude/settings.json`에 hooks 섹션 추가:

```json
"hooks": {
  "UserPromptSubmit": [
    {
      "hooks": [
        {
          "type": "command",
          "command": "node ~/.claude/hooks/gui-status.mjs"
        }
      ]
    }
  ]
}
```

## 4. 테스트

```bash
node ~/.claude/hooks/gui-status.mjs
```

## 5. CLAUDE.md에 출력 규칙 추가

`~/.claude/CLAUDE.md` 파일 상단에 다음 내용 추가:

```markdown
## PART 0: STATUSLINE OUTPUT (ALWAYS)

**매 응답 시작 시**, system-reminder에 `[OMC]`로 시작하는 statusline이 있으면 응답 첫 줄에 코드블록으로 출력하라.

```
[OMC] | opus 4.5 | 5h:8%(3h26m) | wk:11%(4d22h) | session:3m | ctx:0%
```

이 규칙은 **모든 응답에 적용**된다. 예외 없음.
```

> **왜 필요한가?** VSCode Extension에서 hook의 stdout은 사용자 UI에 표시되지 않고, AI 컨텍스트(`<system-reminder>`)로만 전달된다. 따라서 AI가 직접 출력하도록 지시해야 한다.

## 6. Claude Code 재시작

---

## 표시 예시

```
[OMC] | opus 4.5 | thinking | 5h:7%(3h35m) | wk:11%(4d22h) | session:2h56m | ctx:6% | agents:2 | todos:3/5
```

## 표시 항목

| 항목 | 설명 |
|------|------|
| opus 4.5 | 현재 사용 중인 모델 |
| thinking | 확장 사고 모드 활성화 |
| 5h:7%(3h35m) | 5시간 사용량 (리셋까지 남은 시간) |
| wk:11%(4d22h) | 주간 사용량 (리셋까지 남은 시간) |
| session:2h56m | 세션 지속 시간 |
| ctx:6% | 컨텍스트 윈도우 사용량 |
| agents:2 | 실행 중인 서브에이전트 수 |
| todos:3/5 | 할 일 완료 현황 |

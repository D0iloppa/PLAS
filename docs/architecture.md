# PLAS Architecture Blueprint

> 언어 습득 프레임워크의 상세 시스템 설계

---

## 📋 Table of Contents

1. [시스템 개요](#시스템-개요)
2. [핵심 설계 원칙](#핵심-설계-원칙)
3. [아키텍처 다이어그램](#아키텍처-다이어그램)
4. [모듈 상세 설계](#모듈-상세-설계)
5. [MCP 통합](#mcp-통합)
6. [데이터 모델](#데이터-모델)
7. [인터페이스 정의](#인터페이스-정의)
8. [기술 스택 상세](#기술-스택-상세)

---

## 시스템 개요

### 목적
Krashen의 Input Hypothesis를 소프트웨어로 구현하여, 
사용자가 자연스럽게 언어를 습득할 수 있는 환경을 제공한다.

### 핵심 기능
1. **Input Engine**: 콘텐츠 → 이해 가능한 입력 변환
2. **Mirror Engine**: 언어적 부모 역할의 AI 대화 (Claude + MCP)
3. **Shared System**: 사용자 습득 수준 추적

---

## 핵심 설계 원칙

### 1. 도메인 중심 설계 (DDD)
```
각 엔진은 독립된 도메인으로 분리
- Input Engine: 콘텐츠 처리 도메인
- Mirror Engine: 대화 생성 도메인  
- Shared System: 사용자 모델 도메인
- MCP Layer: AI 통합 도메인
```

### 2. 낮은 결합도 (Loose Coupling)
```
Input Engine: 외부 콘텐츠 소스와 느슨한 결합
- 플러그인 아키텍처
- 인터페이스 기반 설계
- 소스 추가/변경 시 최소 수정

Mirror Engine: MCP를 통한 느슨한 결합
- Claude API 직접 의존 제거
- MCP 표준 프로토콜 준수
- 다른 LLM으로 전환 가능
```

### 3. 확장 가능성 (Scalability)
```
초기: 1명 (나)
향후: N명 확장 가능하도록
- 멀티 테넌시 고려
- 상태 비저장 설계 (Stateless)
- MCP Tools 동적 확장
```

### 4. 측정 가능성 (Measurability)
```
모든 상호작용은 습득 지표로 변환:
- 발화 복잡도
- 응답 시간
- 어휘 사용 패턴
- MCP Tool 호출 패턴 분석
```

---

## 아키텍처 다이어그램

### 전체 시스템 구조
```
┌──────────────────────────────────────────────────────────────┐
│                        User Interface                         │
│                   (React Web Application)                     │
│                                                               │
│  ┌─────────────────┐              ┌─────────────────┐       │
│  │ Content Player  │              │   Chat Window   │       │
│  │  - Video/Audio  │              │  - Text Input   │       │
│  │  - Subtitles    │              │  - Voice Input  │       │
│  └─────────────────┘              └─────────────────┘       │
└──────────────────────────────────────────────────────────────┘
                            ↓ ↑
                    [API Gateway / BFF]
                            ↓ ↑
┌──────────────────────────────────────────────────────────────┐
│                    Backbone Orchestrator                      │
│                     (Spring Boot Core)                        │
│                                                               │
│  - Request Routing                                           │
│  - Event Processing                                          │
│  - Pipeline Coordination                                     │
│  - MCP Client Integration                                    │
└──────────────────────────────────────────────────────────────┘
         ↓               ↓                    ↓
┌─────────────┐  ┌──────────────┐   ┌──────────────────┐
│Input Engine │  │Shared System │   │  Mirror Engine   │
│             │  │              │   │                  │
│  Content    │←→│ User Vocab   │   │  MCP Client      │
│  Processing │  │ Metrics DB   │   │  + Prompt Logic  │
└─────────────┘  └──────────────┘   └──────────────────┘
      ↓                  ↓                    ↓
┌─────────────┐  ┌──────────────┐   ┌──────────────────┐
│External     │  │ PostgreSQL   │   │   MCP Server     │
│Content      │  │   + Redis    │   │   (PLAS Tools)   │
│Sources      │  │              │   │        ↓         │
└─────────────┘  └──────────────┘   │   Claude API     │
                                     └──────────────────┘

[MCP Server 상세]
┌──────────────────────────────────────┐
│        PLAS MCP Server               │
│                                      │
│  Tools:                              │
│  ├─ get_user_vocabulary              │
│  ├─ analyze_acquisition_metrics      │
│  ├─ suggest_comprehensible_content   │
│  ├─ update_vocab_usage               │
│  └─ get_learning_history             │
│                                      │
│  Resources:                          │
│  ├─ user_profile                     │
│  ├─ content_library                  │
│  └─ metrics_dashboard                │
└──────────────────────────────────────┘
```

### 데이터 흐름
```
[사용자가 YouTube URL 입력]
         ↓
   [Input Engine]
    ├─ YouTube API 호출
    ├─ 자막 추출 (yt-dlp)
    ├─ 문장 분리 (spaCy)
    ├─ 난이도 분석
    └─ 어휘 마킹
         ↓
   [Shared System]
    ├─ 사용자 어휘 DB 조회
    ├─ 이해 가능 여부 판단
    └─ 메타데이터 저장
         ↓
   [User Interface]
    - 자막과 함께 영상 재생
    
[사용자가 대화 입력]
         ↓
   [Mirror Engine]
    ├─ MCP Client 초기화
    ├─ Claude에게 대화 요청
    │   (사용자 입력 + 시스템 프롬프트)
    └─ Claude가 필요 시 MCP Tools 호출
         ↓
   [MCP Server - Tool Execution]
    ├─ get_user_vocabulary
    │   → Shared System 조회
    │   → 사용자 어휘 제약 반환
    ├─ analyze_acquisition_metrics
    │   → 발화 복잡도 계산
    │   → 실시간 지표 반환
    └─ update_vocab_usage
        → 새 어휘 사용 기록
         ↓
   [Claude Response Generation]
    ├─ Tool 결과 기반 응답 생성
    ├─ 어휘 제약 자동 준수
    └─ 자연스러운 재표현
         ↓
   [Mirror Engine]
    ├─ 응답 후처리
    └─ 습득 지표 저장
         ↓
   [User Interface]
    - 응답 표시
    - 통계 업데이트
```

---

## 모듈 상세 설계

### 1. Input Engine

**책임**:
- 외부 콘텐츠를 이해 가능한 형태로 변환
- 사용자 수준에 맞는 난이도 조절

**인터페이스**:
```java
public interface InputEngine {
    ProcessedContent process(ContentSource source);
    List<Sentence> extractSentences(RawContent content);
    DifficultyLevel analyzeDifficulty(Sentence sentence, UserProfile profile);
}
```

**구현체**:
```
- YouTubeInputEngine
- LocalFileInputEngine
- StreamingInputEngine (future)
```

**외부 의존성**:
- YouTube API / yt-dlp
- Whisper API (음성 → 텍스트)
- spaCy (문장 분리)
- 난이도 분석 알고리즘 (TBD)

**특징**:
- **플러그인 구조**: 새 콘텐츠 소스 추가 시 인터페이스만 구현
- **느슨한 결합**: 특정 플랫폼에 종속되지 않음

---

### 2. Mirror Engine

**책임**:
- 사용자 발화의 의도 파악
- 자연스러운 재표현 (문법 교정 X)
- MCP를 통한 Claude 통합
- 습득 지표 수집

**인터페이스**:
```java
public interface MirrorEngine {
    Response respond(UserInput input, UserProfile profile);
    AcquisitionMetrics analyzeInput(UserInput input);
    String naturalRephrase(String userSentence, VocabConstraint constraint);
}
```

**핵심 로직 (MCP 통합)**:
```java
public Response respond(UserInput input, UserProfile profile) {
    // 1. 사용자 입력 분석 (사전 처리)
    InputAnalysis analysis = analyzeInput(input);
    
    // 2. MCP Client 초기화
    MCPClient mcpClient = new MCPClient("plas-mcp-server");
    
    // 3. Claude에게 요청 (MCP를 통해)
    // Claude가 필요 시 자동으로 MCP Tools 호출
    String systemPrompt = buildSystemPrompt(profile);
    MCPResponse mcpResponse = mcpClient.chat(
        systemPrompt,
        input.getText(),
        profile.getUserId()
    );
    
    // 4. Claude의 응답 + Tool 호출 로그 수신
    String response = mcpResponse.getContent();
    List<ToolCall> toolCalls = mcpResponse.getToolCalls();
    
    // 5. 습득 지표 계산 (Tool 호출 패턴 포함)
    AcquisitionMetrics metrics = AcquisitionMetrics.builder()
        .complexity(analysis.getWordsPerSentence())
        .hesitations(analysis.getPauseCount())
        .responseTime(analysis.getLatency())
        .vocabularyRange(analysis.getUniqueWords())
        .toolCallsCount(toolCalls.size())
        .vocabularyLookups(countToolType(toolCalls, "get_user_vocabulary"))
        .build();
    
    // 6. 지표 저장
    sharedSystem.saveMetrics(metrics);
    
    return new Response(response, metrics, toolCalls);
}

private String buildSystemPrompt(UserProfile profile) {
    return """
        You are a linguistic parent helping a language learner.
        
        Rules:
        1. Never correct grammar directly
        2. Understand their intention and rephrase naturally
        3. Use vocabulary appropriate to their level
        4. Be encouraging and supportive
        
        Available Tools:
        - get_user_vocabulary: Check what words the user knows
        - analyze_acquisition_metrics: Track their progress
        - suggest_comprehensible_content: Recommend suitable content
        
        Use these tools when needed to personalize your responses.
        """;
}
```

**외부 의존성**:
- MCP Client Library
- PLAS MCP Server
- Claude API (via MCP)

**특징**:
- **도구 기반 통합**: Claude가 필요 시 자동으로 사용자 데이터 조회
- **토큰 효율성**: 필요한 정보만 Tool로 조회 (프롬프트 주입 불필요)
- **측정 자동화**: Tool 호출 패턴도 습득 지표로 활용

---

### 3. Shared System

**책임**:
- 사용자 어휘 데이터 관리
- 습득 지표 저장/조회
- 학습 히스토리 추적
- **MCP Tools의 데이터 제공자**

**인터페이스**:
```java
public interface SharedSystem {
    UserProfile getUserProfile(String userId);
    VocabConstraint getUserVocab(String userId);
    void updateVocabUsage(String userId, String word);
    void saveMetrics(String userId, AcquisitionMetrics metrics);
    MetricsHistory getMetricsHistory(String userId, DateRange range);
    
    // MCP Tool 전용
    ComprehensibleContent suggestContent(String userId, String topic);
    AcquisitionAnalysis analyzeProgress(String userId, DateRange range);
}
```

**데이터 저장소**:
- PostgreSQL: 영구 데이터
- Redis: 세션/캐시

**특징**:
- **안정성 우선**: 자주 변경되지 않는 구조
- **중앙 집중**: 모든 모듈이 조회하는 단일 소스
- **MCP 친화적**: Tool 호출에 최적화된 API

---

### 4. Backbone Orchestrator

**책임**:
- 모듈 간 통신 조율
- 이벤트 라우팅
- 파이프라인 제어
- **MCP Client 관리**

**역할**:
```java
@RestController
public class OrchestrationController {
    
    @Autowired
    private MCPClientManager mcpClientManager;
    
    @PostMapping("/process-content")
    public ResponseEntity<?> processContent(@RequestBody ContentRequest request) {
        // 1. Input Engine 호출
        ProcessedContent content = inputEngine.process(request.getSource());
        
        // 2. Shared System에 저장
        sharedSystem.saveContent(content);
        
        // 3. 응답 반환
        return ResponseEntity.ok(content);
    }
    
    @PostMapping("/chat")
    public ResponseEntity<?> chat(@RequestBody ChatRequest request) {
        // 1. 사용자 프로필 조회
        UserProfile profile = sharedSystem.getUserProfile(request.getUserId());
        
        // 2. MCP Client 가져오기
        MCPClient mcpClient = mcpClientManager.getClient(request.getUserId());
        
        // 3. Mirror Engine 호출 (MCP 통합)
        Response response = mirrorEngine.respondViaMCP(
            request.getInput(), 
            profile,
            mcpClient
        );
        
        // 4. 응답 반환 (지표는 이미 저장됨)
        return ResponseEntity.ok(response);
    }
}
```

**특징**:
- **얇은 레이어**: 비즈니스 로직 없음, 라우팅만
- **확장 가능**: 새 엔드포인트 추가 용이
- **MCP 세션 관리**: 사용자별 MCP 연결 유지

---

## MCP 통합

### MCP Server 구현

**PLAS MCP Server** (TypeScript/Node.js)
```typescript
// mcp-server-plas/src/index.ts

import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  ListToolsRequestSchema,
  CallToolRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";

const server = new Server(
  {
    name: "plas-mcp-server",
    version: "1.0.0",
  },
  {
    capabilities: {
      tools: {},
      resources: {},
    },
  }
);

// Tool 1: 사용자 어휘 조회
server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [
    {
      name: "get_user_vocabulary",
      description: "Retrieve user's known vocabulary and proficiency level",
      inputSchema: {
        type: "object",
        properties: {
          user_id: {
            type: "string",
            description: "User ID"
          }
        },
        required: ["user_id"]
      }
    },
    {
      name: "analyze_acquisition_metrics",
      description: "Analyze user's language acquisition progress over time",
      inputSchema: {
        type: "object",
        properties: {
          user_id: { type: "string" },
          time_range: {
            type: "string",
            enum: ["week", "month", "all"],
            description: "Time period to analyze"
          }
        },
        required: ["user_id"]
      }
    },
    {
      name: "suggest_comprehensible_content",
      description: "Suggest content based on user's current level (i+1)",
      inputSchema: {
        type: "object",
        properties: {
          user_id: { type: "string" },
          topic: {
            type: "string",
            description: "Topic of interest"
          }
        },
        required: ["user_id"]
      }
    },
    {
      name: "update_vocab_usage",
      description: "Record that user used a specific word",
      inputSchema: {
        type: "object",
        properties: {
          user_id: { type: "string" },
          word: { type: "string" }
        },
        required: ["user_id", "word"]
      }
    },
    {
      name: "get_learning_history",
      description: "Get user's learning history and patterns",
      inputSchema: {
        type: "object",
        properties: {
          user_id: { type: "string" }
        },
        required: ["user_id"]
      }
    }
  ]
}));

// Tool 실행 핸들러
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  switch (name) {
    case "get_user_vocabulary": {
      const vocab = await fetchUserVocab(args.user_id);
      return {
        content: [
          {
            type: "text",
            text: JSON.stringify({
              known_words: vocab.knownWords,
              learning_words: vocab.learningWords,
              proficiency_level: vocab.level,
              vocabulary_size: vocab.knownWords.length
            })
          }
        ]
      };
    }

    case "analyze_acquisition_metrics": {
      const metrics = await fetchAcquisitionMetrics(
        args.user_id,
        args.time_range
      );
      return {
        content: [
          {
            type: "text",
            text: JSON.stringify({
              complexity_trend: metrics.complexityTrend,
              fluency_improvement: metrics.fluencyScore,
              vocabulary_growth: metrics.vocabGrowth,
              insights: generateInsights(metrics)
            })
          }
        ]
      };
    }

    case "suggest_comprehensible_content": {
      const suggestions = await getSuggestedContent(
        args.user_id,
        args.topic
      );
      return {
        content: [
          {
            type: "text",
            text: JSON.stringify({
              recommendations: suggestions.items,
              difficulty_level: suggestions.targetLevel,
              reasoning: suggestions.explanation
            })
          }
        ]
      };
    }

    case "update_vocab_usage": {
      await recordVocabUsage(args.user_id, args.word);
      return {
        content: [
          {
            type: "text",
            text: JSON.stringify({ success: true })
          }
        ]
      };
    }

    case "get_learning_history": {
      const history = await fetchLearningHistory(args.user_id);
      return {
        content: [
          {
            type: "text",
            text: JSON.stringify(history)
          }
        ]
      };
    }

    default:
      throw new Error(`Unknown tool: ${name}`);
  }
});

// 데이터 조회 함수들 (Shared System API 호출)
async function fetchUserVocab(userId: string) {
  // Spring Boot API 호출
  const response = await fetch(
    `http://localhost:8080/api/vocab/${userId}`
  );
  return response.json();
}

async function fetchAcquisitionMetrics(userId: string, timeRange: string) {
  const response = await fetch(
    `http://localhost:8080/api/metrics/${userId}?range=${timeRange}`
  );
  return response.json();
}

// ... 기타 함수들

// 서버 시작
const transport = new StdioServerTransport();
await server.connect(transport);
```

### MCP 통합 장점

**1. 토큰 효율성**
```
Before (프롬프트 주입):
System: "User knows these 500 words: apple, banana, cat, ..."
→ 토큰 낭비, 매 요청마다 반복

After (MCP Tool):
Claude: [calls get_user_vocabulary when needed]
→ 필요할 때만, 구조화된 데이터로
→ 토큰 사용 40% 감소
```

**2. 동적 컨텍스트**
```
Before:
- 대화 시작 시 사용자 정보 로드
- 중간에 변경사항 반영 안 됨

After:
- Claude가 실시간으로 최신 정보 조회
- 대화 중 사용자 수준 변화 즉시 반영
```

**3. 확장성**
```
새 기능 추가:
1. MCP Server에 Tool 추가
2. Claude가 자동으로 사용법 학습
3. Mirror Engine 코드 수정 불필요
```

---

## 데이터 모델

### User Profile
```sql
CREATE TABLE user_profiles (
    user_id VARCHAR(36) PRIMARY KEY,
    name VARCHAR(100),
    target_language VARCHAR(10),
    current_level VARCHAR(10), -- A1, A2, B1, B2, C1, C2
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);
```

### User Vocabulary
```sql
CREATE TABLE user_vocabulary (
    id BIGSERIAL PRIMARY KEY,
    user_id VARCHAR(36) REFERENCES user_profiles(user_id),
    word VARCHAR(100),
    frequency INT DEFAULT 0,
    last_seen TIMESTAMP,
    mastery_level INT, -- 0-5
    INDEX idx_user_word (user_id, word)
);
```

### Acquisition Metrics
```sql
CREATE TABLE acquisition_metrics (
    id BIGSERIAL PRIMARY KEY,
    user_id VARCHAR(36) REFERENCES user_profiles(user_id),
    timestamp TIMESTAMP,
    session_id VARCHAR(36),
    
    -- 발화 지표
    user_input TEXT,
    word_count INT,
    unique_words INT,
    complexity_score DECIMAL(3,2), -- words per sentence
    
    -- 유창성 지표
    response_time_ms INT,
    hesitation_count INT,
    self_corrections INT,
    
    -- 정확성 지표
    grammar_naturalness DECIMAL(3,2), -- 1-10
    vocabulary_level VARCHAR(10), -- A1-C2
    
    -- MCP 관련
    tool_calls_count INT, -- Tool 호출 횟수
    vocabulary_lookups INT, -- 어휘 조회 횟수
    
    INDEX idx_user_time (user_id, timestamp)
);
```

### Content Library
```sql
CREATE TABLE contents (
    id BIGSERIAL PRIMARY KEY,
    user_id VARCHAR(36) REFERENCES user_profiles(user_id),
    source_type VARCHAR(50), -- youtube, local, stream
    source_url TEXT,
    title VARCHAR(500),
    difficulty_level VARCHAR(10),
    processed_at TIMESTAMP,
    metadata JSONB
);

CREATE TABLE sentences (
    id BIGSERIAL PRIMARY KEY,
    content_id BIGINT REFERENCES contents(id),
    sequence_no INT,
    text TEXT,
    difficulty_score DECIMAL(3,2),
    new_vocab_count INT,
    timestamp_start INT, -- ms
    timestamp_end INT
);
```

---

## 인터페이스 정의

### InputEngine Interface
```java
package com.plas.input;

public interface InputEngine {
    /**
     * 콘텐츠 소스를 처리하여 이해 가능한 형태로 변환
     */
    ProcessedContent process(ContentSource source) throws ContentProcessException;
}

public class ContentSource {
    private SourceType type; // YOUTUBE, LOCAL, STREAM
    private String url;
    private Map<String, Object> metadata;
}

public class ProcessedContent {
    private String contentId;
    private List<Sentence> sentences;
    private ContentMetadata metadata;
    private DifficultyStats stats;
}

public class Sentence {
    private int sequenceNo;
    private String text;
    private DifficultyLevel difficulty;
    private List<Vocabulary> newVocab;
    private TimeRange timestamp;
}
```

### MirrorEngine Interface
```java
package com.plas.mirror;

public interface MirrorEngine {
    /**
     * 사용자 입력에 대해 자연스러운 응답 생성 (MCP 통합)
     */
    Response respondViaMCP(UserInput input, UserProfile profile, MCPClient mcpClient) 
        throws MirrorEngineException;
    
    /**
     * 입력 분석 및 습득 지표 계산
     */
    AcquisitionMetrics analyzeInput(UserInput input);
}

public class UserInput {
    private String userId;
    private String text;
    private Instant timestamp;
    private AudioMetadata audioData; // optional
}

public class Response {
    private String text;
    private AcquisitionMetrics metrics;
    private List<ToolCall> toolCalls; // MCP Tool 호출 로그
    private List<String> suggestedExpressions; // optional
}

public class AcquisitionMetrics {
    private double complexityScore;
    private int responseTimeMs;
    private int hesitationCount;
    private double grammarNaturalness;
    private int wordCount;
    private int uniqueWords;
    private int toolCallsCount; // MCP Tool 호출 횟수
    private int vocabularyLookups; // 어휘 조회 횟수
}

public class ToolCall {
    private String toolName;
    private Map<String, Object> arguments;
    private Object result;
    private long durationMs;
}
```

### SharedSystem Interface
```java
package com.plas.shared;

public interface SharedSystem {
    UserProfile getUserProfile(String userId);
    VocabConstraint getUserVocab(String userId);
    void updateVocabUsage(String userId, String word);
    void saveMetrics(String userId, AcquisitionMetrics metrics);
    MetricsHistory getMetricsHistory(String userId, DateRange range);
    
    // MCP Tool 전용 메서드
    ComprehensibleContent suggestContent(String userId, String topic);
    AcquisitionAnalysis analyzeProgress(String userId, DateRange range);
}

public class VocabConstraint {
    private Set<String> knownWords;
    private Set<String> learningWords;
    private DifficultyLevel maxLevel;
}

public class MetricsHistory {
    private List<DataPoint> complexityTrend;
    private List<DataPoint> fluencyTrend;
    private Map<String, Integer> vocabularyGrowth;
}
```

### MCPClient Interface
```java
package com.plas.mcp;

public interface MCPClient {
    /**
     * MCP를 통해 Claude와 대화
     */
    MCPResponse chat(String systemPrompt, String userMessage, String userId);
    
    /**
     * 사용 가능한 Tool 목록 조회
     */
    List<ToolDefinition> listTools();
    
    /**
     * MCP 연결 상태 확인
     */
    boolean isConnected();
}

public class MCPResponse {
    private String content; // Claude의 응답
    private List<ToolCall> toolCalls; // 호출된 Tool 목록
    private Map<String, Object> metadata;
}
```

---

## 기술 스택 상세

### Backend

**Spring Boot 3.x**
- Role: Backbone Orchestrator
- 모듈 간 통신 관리
- REST API 제공
- MCP Client 관리

**Python 3.11+**
- Role: NLP Utilities
- spaCy, NLTK (문장 분리, 품사 태깅)
- yt-dlp (YouTube 자막 추출)

**Node.js 20+ / TypeScript**
- Role: MCP Server
- PLAS Tools 구현
- Shared System API 호출

**Package Structure**:
```
backend/
├── orchestrator/          (Spring Boot)
│   ├── controller/
│   ├── service/
│   ├── mcp/              ← MCP Client 관리
│   └── config/
├── input-engine/          (Python)
│   ├── youtube/
│   ├── local/
│   └── analyzer/
├── mirror-engine/         (Java)
│   ├── prompter/
│   ├── analyzer/
│   ├── mcp/              ← MCP 통합 로직
│   └── metrics/
└── mcp-server-plas/      (TypeScript)
    ├── tools/            ← MCP Tools 구현
    ├── api/              ← Shared System 연동
    └── index.ts
```

### AI/ML

**Claude API (via MCP)**
- Mirror Engine 대화 생성
- MCP를 통한 도구 기반 통합
- 프롬프트 + Tool 조합

**Whisper API** (향후)
- 음성 → 텍스트
- 실시간 STT

### MCP

**MCP SDK**
- @modelcontextprotocol/sdk
- Server/Client 구현

**MCP Tools**
- get_user_vocabulary
- analyze_acquisition_metrics
- suggest_comprehensible_content
- update_vocab_usage
- get_learning_history

### Storage

**PostgreSQL 15**
- 사용자 프로필
- 어휘 데이터
- 습득 지표
- 콘텐츠 라이브러리

**Redis 7**
- 세션 캐시
- 실시간 메트릭 버퍼
- MCP 세션 상태
- Rate limiting

### Frontend

**React 18**
- Vite 빌드
- TypeScript

**shadcn/ui**
- UI 컴포넌트

**TanStack Query**
- 서버 상태 관리

---

## 배포 구조
```
[Local Development]
- Docker Compose
  ├─ PostgreSQL container
  ├─ Redis container
  ├─ MCP Server container
  └─ Spring Boot (localhost:8080)
- React Dev Server (localhost:5173)

[Production (Future)]
- Cloud Run / ECS
  ├─ Spring Boot API
  └─ MCP Server
- Managed PostgreSQL (RDS/Cloud SQL)
- Redis (ElastiCache/MemoryStore)
- CDN (Static assets)
```

---

## 성능 목표

| 지표 | 목표 | 측정 방법 |
|------|------|-----------|
| Mirror Engine 응답 시간 (MCP) | < 1.5초 | API latency (Tool 포함) |
| MCP Tool 호출 시간 | < 100ms | Tool execution time |
| Input Engine 처리 시간 | < 30초/10분 영상 | 배치 처리 |
| DB 쿼리 시간 | < 50ms | Slow query log |
| 동시 사용자 | 1명 (MVP) → 100명 (Phase 2) | Load testing |

---

## 보안 고려사항

1. **API Key 관리**
   - 환경 변수로 분리
   - `.env` 파일 gitignore
   - AWS Secrets Manager (향후)

2. **사용자 데이터**
   - 대화 로그는 개인 정보
   - 암호화 저장 (AES-256)
   - 삭제 정책 (30일 후 자동 삭제 옵션)

3. **MCP 보안**
   - MCP Server 인증/인가
   - Tool 호출 권한 제어
   - Rate limiting per user

4. **외부 API**
   - Rate limiting
   - Retry with exponential backoff
   - Circuit breaker pattern

---

## 모니터링 및 로깅
```
[Logging]
- 구조화된 로깅 (JSON)
- Logback (Spring Boot)
- 로그 레벨: DEBUG (개발), INFO (운영)
- MCP Tool 호출 로그

[Metrics]
- Micrometer + Prometheus (향후)
- 주요 지표:
  - API response time
  - MCP Tool call latency
  - Claude API latency
  - DB connection pool
  - Tool call frequency by type
  
[Alerting]
- MCP Server 연결 끊김
- Claude API 실패 시 알림
- DB 연결 끊김 알림
- Tool 호출 실패율 임계값
```

---

## 확장 계획

### Phase 1: MVP (현재)
- 1명 사용 (나)
- YouTube + Claude (MCP)
- 웹 인터페이스만
- 5개 MCP Tools

### Phase 2: 검증 (3-6개월 후)
- 베타 사용자 10명
- 음성 입력 추가
- 모바일 반응형
- 10개 MCP Tools

### Phase 3: 확장 (1년 후)
- 멀티 테넌시
- 다국어 지원 (한→영, 영→일 등)
- 사내 교육 콘텐츠 연동
- MCP Tools Marketplace (커뮤니티 기여)

---

## MCP 커뮤니티 기여 계획

1. **PLAS MCP Server 오픈소스 공개**
   - GitHub: https://github.com/D0iloppa/plas-mcp-server
   - npm 패키지 배포

2. **Anthropic MCP Repository PR**
   - Example: Language Acquisition MCP Server
   - Documentation 기여

3. **블로그 시리즈**
   - "Building Language Learning System with MCP"
   - "MCP Performance: Before and After"

4. **커뮤니티 참여**
   - MCP Discord 활동
   - HackerNews "Show HN" 포스트

---

_Last updated: 2025-11-18_
_Version: 2.0 (MCP Integration)_
# 1. 핵심 결론

보안도메인의 엣지 LLM 배포는 “LLM을 엣지 서버에 올리는 것”이 아니라, 민감 데이터가 생기는 위치 가까이에서 1차 추론·필터링·관제를 수행하고 중앙 클라우드에는 지식·정책·업데이트·거버넌스를 맡기는 구조로 이동하고 있습니다.

> 민감 데이터는 엣지/온프레미스에 남기고, LLM은 RAG·에이전트·AI Gateway와 결합해 SOC 분석, 취약점 triage, LLM 앱 보호, 영상 보안 판단을 보조합니다.
> 

보안 로그, 패킷, EDR/XDR 이벤트, 영상, 취약점 정보는 그대로 외부 SaaS LLM에 보내기 어렵습니다. 그래서 최근 사례들은 중앙 cloud-only 구조보다 private, edge, hybrid LLM 구조를 선호합니다.

- 엣지/온프레미스: 민감 원천 데이터 수집, 전처리, 요약, 1차 판단
- 중앙 클라우드: 모델·정책 업데이트, 공통 지식베이스, 감사·거버넌스
- 운영 원칙: LLM은 판단 보조와 근거 제시에 집중하고, 차단·격리 같은 고위험 조치는 human-in-the-loop로 통제

# 2. 최근 동향: Cloud-only에서 Private / Edge / Hybrid로

최근 흐름은 세 갈래로 정리됩니다.

- 첫째, 민감 데이터 보호를 위해 LLM 추론 위치가 데이터 발생 지점 가까이로 내려옵니다.
- 둘째, 보안을 위한 LLM과 LLM을 보호하는 보안이 동시에 커집니다.
- 셋째, CDN/WAF/ADC 기업들이 AI Gateway와 AI Firewall을 엣지 보안 계층으로 확장합니다.

## 2.1. 민감 데이터는 밖으로 보내지 않고, 지식과 정책만 중앙에서 관리

보안도메인에서 LLM을 도입할 때 제일 큰 제약은 데이터 이동입니다. 패킷, 로그, 사고 대응 기록, 취약점 스캔 결과, 영상은 조직 내부 맥락과 개인정보를 포함합니다.

그래서 실무적으로는 로컬 GPU/NPU, MEC, 프라이빗 클라우드, 온프레미스 어플라이언스에서 SLM/LLM을 구동하고, 중앙은 업데이트와 governance에 집중하는 설계가 자연스럽습니다.

## 2.2. “보안을 위한 LLM”과 “LLM을 보호하는 보안”의 동시 성장

보안을 위한 LLM은 SOC 분석, 위협 인텔리전스, 취약점 분석, 로그 요약, 대응 절차 추천에 쓰입니다. 

반대로 LLM을 보호하는 보안은 prompt injection, 민감정보 노출, 모델·데이터 공급망, vector DB/embedding 취약점, 과도한 agent 권한을 통제합니다.

> OWASP LLM Top 10 2025
: Prompt Injection, Sensitive Information Disclosure, Supply Chain, Data/Model Poisoning, Excessive Agency, System Prompt Leakage, Vector and Embedding Weaknesses 등을 주요 위험으로 제시
> 

## 2.3. AI Gateway / AI Firewall이 새 엣지 보안 계층으로 부상

Cloudflare, Akamai, F5는 기존 WAF/API Gateway/CDN 위치를 LLM 트래픽 제어 지점으로 확장하고 있습니다. 모델이 외부 SaaS든 내부 vLLM/NIM이든 앞단에서 prompt, response, DLP, rate limit, 정책, 감사 로그를 통합 관리하는 패턴입니다.

# 3. 대표 사례 정리

발표에서는 제품명을 나열하기보다 “어떤 배포 패턴을 보여주는가”로 사례를 묶는 편이 이해가 쉽습니다. 아래 사례들은 취약점 분석, SOC/위협탐지, LLM 앱 보호, 분산 보안, 물리보안/VLM으로 나누어 볼 수 있습니다.

## 3.1. NVIDIA NIM Agent Blueprint for Container Security

NVIDIA의 Container Security Blueprint는 컨테이너 CVE 분석을 LLM/RAG/agent 기반으로 자동화하는 사례입니다. NIM microservices, Morpheus, cuVS, RAPIDS 등을 활용해 스캐너 결과와 CVE 정보를 연결하고, LLM이 실제 위험도와 대응 방향을 설명합니다.

- 핵심 입력: 컨테이너 스캔 결과, SBOM, CVE DB, 소스코드·문서, 런타임 맥락
- 핵심 출력: 영향도 설명, exploit 가능성 판단, 패치 우선순위, 대응 조치 초안
- 스터디 포인트: Trivy/Grype/Syft/OSV/NVD/Git repo를 RAG로 묶어 로컬 LLM이 CVE triage를 수행하는 PoC로 재현하기 좋습니다.

## 3.2. NVIDIA Morpheus + CrowdStrike

CrowdStrike와 NVIDIA의 협업은 EDR/XDR 기반 보안 데이터를 가속 컴퓨팅과 custom LLM application으로 분석하려는 흐름을 보여줍니다. 완전한 엣지 배포라기보다는 대용량 보안 스트리밍 데이터를 필터링·분류하고 SOC 판단을 보조하는 AI pipeline 사례로 보는 것이 적절합니다.

- 엣지 해석: Edge sensor/collector → streaming analysis → LLM triage → central SOC 전송

## 3.3. 국내 사례: IGLOO AiR + Rebellions NPU

국내 사례로는 이글루코퍼레이션 AiR와 Rebellions NPU 조합이 중요합니다. 보안 위협 판단, 판단 근거 제시, 예측 기반 분석을 보안 어시스턴트 형태로 제공하면서, 보안이 보장된 온프레미스 환경과 전성비를 고려했다는 점이 엣지 클라우드 관점과 잘 맞습니다.

- 의미: 외부 cloud 전송이 어려운 SOC 데이터를 온프레미스/프라이빗 엣지 박스에서 처리
- 인프라 포인트: GPU뿐 아니라 NPU 기반 appliance도 보안 LLM 배포 옵션이 될 수 있음

## 3.4. Cloudflare / Akamai / F5: AI Gateway와 AI Firewall

이 세 사례는 “LLM 자체를 엣지에 올리는 것”보다 “LLM 트래픽 보호 계층을 엣지에 두는 것”에 가깝습니다. 기존 WAF가 HTTP request를 검사하듯, AI Firewall은 prompt와 response를 검사합니다.

- Cloudflare: Firewall for AI와 AI Gateway로 prompt injection, PII disclosure, unbound consumption, 관측성, 비용·정책 제어를 제공
- Akamai: 모델 비종속 Firewall for AI를 edge/API 방식으로 제공하고 prompt injection, 유해 출력, 민감정보 노출 등을 방어
- F5: AI Gateway를 Kubernetes 기반으로 배포하고 public cloud, on-prem, edge 환경에서 보안·관측성을 제공하는 방향

## 3.5. Cisco Hypershield

Cisco Hypershield는 LLM 배포 제품이라기보다는 분산형 AI-native 보안 아키텍처 사례입니다. 서버, 애플리케이션, 네트워크, 퍼블릭/프라이빗 클라우드 전반에서 워크로드 가까이 정책 집행과 관측을 수행한다는 점에서 엣지 보안 모델의 참고 사례로 볼 수 있습니다.

## 3.6. NVIDIA Metropolis / VLM Edge / Ambarella

보안도메인을 물리보안과 안전관리까지 넓히면 엣지 VLM 사례가 더 많아집니다. NVIDIA Metropolis와 Video Search and Summarization Blueprint는 영상에서 맥락을 이해하고 검색·요약·알림을 생성하는 visual AI agent 흐름을 보여줍니다. Ambarella의 edge GenAI SoC도 보안 카메라, 온프레미스 AI box, 스마트시티 보안을 겨냥합니다.

# 4. 배포 아키텍처 패턴

## 4.1. 패턴 A: Edge SOC Copilot

- 보안 로그가 외부로 나가면 안 되는 환경에 맞는 구조
- 엣지 노드에서 Suricata, Zeek, Sysmon, auditd, EDR 이벤트를 수집하고, MITRE ATT&CK, 내부 runbook, CVE, 자산 정보를 RAG로 연결합니다.

```
Edge Collector / SIEM Agent → Local Parser → Vector DB / RAG → Local LLM or SLM → Human Analyst → SOAR
```

- LLM 역할: alert 요약, 위험 근거 설명, IOC 연결, 다음 조치 추천
- 통제 원칙: 자동 차단보다 analyst approval 이후 SOAR 실행

## 4.2. 패턴 B: Container CVE Agent

- NVIDIA Blueprint에 가장 가까운 패턴
- 스캐너가 뱉은 CVE 목록만으로는 실제 위험도를 판단하기 어렵기 때문에, LLM이 “실제로 런타임 경로에 있는가”, “exploit 조건이 맞는가”, “우선 패치해야 하는가”를 설명

```
Scanner → SBOM → CVE DB → Source Repo → RAG → LLM Agent → Patch Recommendation
```

## 4.3. 패턴 C: AI Gateway / AI Firewall

- 모델 위치와 무관하게 LLM 요청·응답 앞단을 통제하는 패턴
- 내부 vLLM, NIM, 외부 OpenAI/Anthropic/Gemini API를 모두 같은 정책면으로 묶을 수 있습니다.

```
User / App → AI Gateway at Edge → Policy / DLP / Prompt Filter → LLM → Output Filter → App
```

- 핵심 기능: prompt injection 탐지, PII masking, output filtering, token/rate limit, model routing, audit log, user policy

## 4.4. 패턴 D: Edge VLM 물리보안

기존 영상분석이 단일 이벤트 판별에 가까웠다면, VLM/LLM 결합은 “왜 위험한지”, “무엇을 해야 하는지”, “이전 이벤트와 어떤 관련이 있는지”까지 설명합니다.

```
Camera / Sensor → Edge VLM → Event Summarizer LLM → Local Alert → Central SOC
```

# 5. 구현 시 핵심 기술 요소

엣지 LLM 보안 PoC는 모델 하나를 띄우는 데서 끝나지 않습니다. 모델 경량화, RAG, human-in-the-loop, LLM 보안 통제, 관측성까지 같이 설계해야 합니다.

## 5.1. 모델 경량화와 가속기 선택

엣지에서는 대형 모델을 그대로 올리기 어렵습니다. SLM 선택, 4bit/8bit quantization, pruning, distillation, LoRA, speculative decoding, KV cache 최적화가 중요합니다. GPU 외에도 NPU나 전용 SoC가 전력·상면 제약에서 장점이 있습니다.

## 5.2. RAG 중심 설계

보안 지식은 계속 바뀝니다. CVE, 내부 자산, 탐지 룰, 대응 절차, threat intelligence를 모델 weight에 모두 넣기보다 RAG와 문서 저장소에 두는 것이 현실적입니다. 모델은 요약과 판단 보조, 지식은 검색 가능한 외부 저장소에 둡니다.

## 5.3. Human-in-the-loop와 최소권한 agent

보안 대응은 오탐 비용이 큽니다. 초기 설계에서는 LLM이 요약, 근거 제시, 대응 스크립트 초안까지 맡고 실제 차단·격리·삭제는 사람이 승인하도록 두는 편이 안전합니다. agent 권한은 read-only first, 최소권한, 감사 로그, tool call policy로 제한해야 합니다.

## 5.4. Prompt Injection =  앱 설계 문제

NCSC가 지적하듯 prompt injection은 단순 SQL injection처럼 입력값 escaping만으로 해결하기 어렵습니다. LLM은 명령과 데이터를 완벽히 분리하기 어려운 “confusable deputy”에 가깝기 때문에, 권한 제한, 출력 검증, tool 호출 통제, 민감정보 마스킹, 정책 기반 gateway가 필요합니다.

# 6. 스터디 진행방향

> → 데이터센터 → 각 기업 → 사용자 PC로 이어지는 Edge Security LLM Architecture 설계 프로젝트
> 

### 목표

- 데이터센터의 중앙 보안 관제와 기업별 엣지 보안 노드, 사용자 PC의 endpoint telemetry를 계층적으로 연결하는 보안솔루션 아키텍처를 설계한다
- 민감 원본 데이터는 기업 내부와 사용자 단말 가까이에 남기고, 엣지 LLM이 1차 분석·요약·정책 판단을 수행한 뒤 중앙 데이터센터 SOC에는 표준화된 alert summary와 대응 근거만 전달한다.

### 구성 예시

```
Data Center / Central SOC
  - Global Threat Intelligence, Model/Policy Registry, Governance
  - Cross-tenant correlation, dashboard, audit log
        ↓
Enterprise Edge Security Node
  - SIEM/XDR connector, RAG, local vLLM or SLM, AI Gateway
  - DLP, prompt/output policy, MITRE ATT&CK mapping
        ↓
User PC / Endpoint Agent
  - EDR event, process/network/file telemetry, local pre-filter
  - suspicious event summary, privacy-preserving upload
        ↑
Feedback Loop
  - analyst approval, policy update, detection rule tuning
```

### 기대 결과

- 아키텍처 산출물: 데이터센터, 기업 엣지 노드, 사용자 PC endpoint agent를 연결한 end-to-end 보안솔루션 구조도와 데이터 흐름을 제시합니다.
- 보안 기능 산출물: 이벤트 요약, MITRE ATT&CK 매핑, 위험도 판단, 대응 가이드 생성, DLP/AI Gateway 정책 통제를 포함합니다.
- 엣지 클라우드 관점: latency, privacy, bandwidth, local autonomy, 중앙 governance 사이의 trade-off를 설계 근거로 설명할 수 있습니다.
- 진로적 강점: Cloud/Edge architecture, SOC/XDR, endpoint security, LLM/RAG, AI Gateway를 하나의 포트폴리오 주제로 연결해 보안 아키텍트·클라우드 보안·AI 보안 직무 역량을 보여줄 수 있습니다.

# #. 참고 링크

- NVIDIA NIM Agent Blueprint for Container Security — https://blogs.nvidia.com/blog/nim-agent-blueprint-container-security/
- NVIDIA Developer: AWS CI pipeline security patching blueprint — https://developer.nvidia.com/blog/automate-early-security-patching-in-ci-pipelines-on-aws-using-nvidia-ai-blueprints/
- NVIDIA Morpheus Cybersecurity — https://developer.nvidia.com/morpheus-cybersecurity
- CrowdStrike + NVIDIA generative AI cybersecurity collaboration — https://ir.crowdstrike.com/news-releases/news-release-details/crowdstrike-collaborates-nvidia-advance-cybersecurity-generative
- Rebellions: IGLOO AiR 사례 — https://kr.rebellions.ai/soc%EC%9D%98-%EB%B3%B4%EC%95%88-%EC%9C%84%ED%98%91-%ED%83%90%EC%A7%80%EC%99%80-%EB%8C%80%EC%9D%91%EC%97%90-llm-%EA%B8%B0%EB%B0%98-ai-%EC%A0%91%EB%AA%A9/
- Cloudflare Firewall for AI — https://blog.cloudflare.com/block-unsafe-llm-prompts-with-firewall-for-ai/
- Akamai Firewall for AI — https://www.akamai.com/newsroom/press-release/akamai-firewall-for-ai-enables-secure-ai-applications-with-advanced-threat-protection
- F5 AI Gateway — https://www.f5.com/company/blog/put-control-and-security-where-your-ai-applications-are
- Cisco Hypershield — https://blogs.cisco.com/security/cisco-hypershield-a-new-era-of-distributed-ai-native-security
- NVIDIA VLM/Visual AI agents for edge — https://developer.nvidia.com/blog/develop-generative-ai-powered-visual-ai-agents-for-the-edge/
- Ambarella edge GenAI at CES 2025 — https://www.ambarella.com/blog/advancing-genai-at-the-edge-during-ces-2025/
- OWASP Top 10 for LLM Applications — https://owasp.org/www-project-top-10-for-large-language-model-applications/
- NCSC: Prompt injection is not SQL injection — https://www.ncsc.gov.uk/blog-post/prompt-injection-is-not-sql-injection

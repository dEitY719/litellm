# 개발용 LiteLLM 설치

## 설치 및 실행
`docker compose up -d` 로 실행

tinyllama1가 Ollama 모델 서버로 실행되고, postgres가 DB 서버로 실행되고 litellm이 설정된 모델 목록을 불러와서 프록시 API 서버로 동작하게 됩니다.

✅ 이후 필수 초기 작업 (최초 1회만)
Litellm이 모델 프록시를 올바르게 처리하려면 각 Ollama 인스턴스에 모델 다운로드 및 로드 명령을 실행해야 합니다:

docker exec -it tinyllama1 ollama run tinyllama

이 작업은 한 번만 하면 됩니다 (모델 캐시됨)
시간이 오래 걸리지 않습니다 (~50MB)

## 테스트
마스터 키 테스트
```bash
curl -X GET "http://localhost:4444/models" \
  -H "Authorization: Bearer sk-4444" \
  -H "Content-Type: application/json"
```

모델 쿼리 테스트
```bash
curl http://localhost:4444/v1/chat/completions \
  -H "Authorization: Bearer sk-4444" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "tinyllama1",
    "messages": [{"role": "user", "content": "안녕!"}]
  }'
```

## 모델 추가하는 방법
GUI로 접속하면 gemini 등 외부 api 아주 쉽게 등록 가능.

### 외부 LLM(Gemini, OpenAI, DeepSeek 등) 추가 절차
1. **API 키 주입**: `docker-compose.yml`의 `litellm` 서비스 `environment` 블록에 각 공급자의 키를 넣습니다.
   ```yaml
   environment:
     GEMINI_API_KEY: "your-gemini-api-key"
     OPENAI_API_KEY: "your-openai-api-key"
     DEEPSEEK_API_KEY: "your-deepseek-api-key"
   ```
   이미 실행 중이라면 `docker compose up -d` 전에 키 값을 갱신하세요.
2. **모델 매핑 등록**: `litellm_settings.yml`에 원하는 모델을 추가합니다. 예시는 다음과 같습니다.
   ```yaml
   model_list:
     - model_name: gemini-pro
       litellm_params:
         model: gemini/gemini-pro
     - model_name: openai-gpt4o
       litellm_params:
         model: openai/gpt-4o-mini
     - model_name: deepseek-chat
       litellm_params:
         model: deepseek/deepseek-chat
   ```
   `model_name` 값이 곧 프록시에서 호출할 ID이므로 명확한 규칙(예: 공급자-모델명)을 유지합니다.
3. **서비스 재시작**: 설정 반영을 위해 `docker compose restart litellm` 또는 전체 스택을 재기동합니다.
4. **동작 확인**: `curl http://localhost:4444/models -H "Authorization: Bearer sk-4444"`로 모델이 노출되는지 확인하고, 이어서 `/v1/chat/completions` 요청 시 `"model": "openai-gpt4o"`와 같이 새 모델 이름을 지정합니다.

GUI(LiteLLM Dashboard)에서도 같은 과정을 UI로 처리할 수 있으며, `STORE_MODEL_IN_DB=True` 덕분에 추가한 모델은 Postgres에 저장되어 재부팅 후에도 유지됩니다.

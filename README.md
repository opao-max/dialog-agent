# Dialog Agent

A task-oriented dialogue agent that orchestrates a pipeline of microservices — intent detection, NLU, semantic slot filling, function calling, and rejection handling — with MCP (Model Context Protocol) tool servers and streaming LLM responses over WebSocket.

## Highlights

- **Microservice orchestration**: intent recognition → NLU → slot parsing → function calling → response arbitration → streaming chat
- **Function calling**: built-in domain managers (weather / music / maps) plus dynamic slot processing
- **MCP integration**: MCP clients and servers (AMP, music) for tool execution
- **Model training pipeline**: BERT-based intent / rejection classifiers with LoRA-friendly training loop
- **Streaming over WebSocket**: Flask-SocketIO server with threaded async mode
- **Redis-backed session state**: conversation context and slot persistence

## Architecture

```
                    ┌────────────────────────────────────────────┐
                    │              SocketIO Server               │
                    │              start.py (Flask)              │
                    └───────┬──────────┬──────────┬──────────────┘
                            │          │          │
              ┌─────────────▼──┐  ┌────▼──────┐  ┌▼─────────────────┐
              │ Intent Server  │  │  NLU      │  │ Reject Server    │
              │ (BERT cls)     │  │ Server    │  │ (BERT cls)       │
              └─────────────┬──┘  └────┬──────┘  └┬─────────────────┘
                            └──────────┼──────────┘
                                       ▼
                    ┌────────────────────────────────────────────┐
                    │        Semantic Slot & Function Call       │
                    │     function_call/ (dm, slot_process)      │
                    └──────────────────────┬─────────────────────┘
                                           ▼
                    ┌────────────────────────────────────────────┐
                    │          MCP Tool Servers (AMP/Music)      │
                    └──────────────────────┬─────────────────────┘
                                           ▼
                    ┌────────────────────────────────────────────┐
                    │     LLM Streaming (rewrite → chat → nlg)   │
                    └────────────────────────────────────────────┘
```

## Quickstart

### 1. Install dependencies

```bash
pip install -r requirements.txt
```

### 2. Configure

```bash
export API_KEY="Bearer <your-api-key>"
export BASE_URL="https://ark.cn-beijing.volces.com/api/v3/chat/completions"
```

### 3. Start dependent services

The system expects intent / NLU / reject microservices (see `config/config.ini` for endpoints), then:

```bash
bash server.sh
```

or run directly:

```bash
python start.py
```

### 4. Run a dialogue test

```bash
python test.py
```

## Project Layout

```
├── client/          # microservice clients (NLU / reject / arbitration / streaming)
├── config/          # class map, slot-intent config, endpoint config
├── function_call/   # domain managers and semantic slot processing
├── mcp_core/        # MCP client and tool servers
├── train/           # BERT intent / rejection classifier training
├── utils/           # logger, Redis session helpers
├── prompts.py       # LLM prompt templates
├── dialog.py        # dialogue state machine
├── start.py         # SocketIO server entry
└── test/            # benchmark scripts and sample data
```

## Model Training

```bash
python train/run.py          # train intent classifier
python train/train_eval.py   # evaluate
python train/intent_infer.py # inference
```

## License

[MIT](LICENSE)

## Notes

- Intent and reject classifiers are trained separately and exposed as independent microservices; the orchestrator calls them over HTTP.
- The MCP tool servers are optional - the dialogue loop degrades gracefully when they are unreachable.


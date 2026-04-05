# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test

```bash
# Compile
mvn validate compile

# Package (specify platform: windows-x86, windows-x86_64, macosx-arm64, macosx-x86_64, linux-arm64, linux-x86, linux-x86_64)
mvn -Dplatform=linux-x86_64 clean package

# Run single test
mvn test -Dtest=ClassName

# Run all tests
mvn test
```

## Architecture

**Spring Boot 3.0.5 application** (Java 17) - Agent for Sonic Cloud Real Machine Platform, handling device connectivity and test execution for Android/iOS devices.

### Module Structure

- **`bridge/`** - Device communication layer
  - `android/` - ADB bridge via ddmlib, device status listeners, thread pools
  - `ios/` - iOS device bridge using sibTool, WebDriverAgent coordination

- **`tests/`** - Test execution engine
  - `AndroidTests.java` / `IOSTests.java` - Main test orchestrators
  - `RunStepThread.java` - Base thread for step execution
  - `handlers/` - Step type handlers (If/Else/While/Assert/Touch/Permission)
  - `android/` & `ios/` - Platform-specific threads (battery, perf, record, screen streaming)
  - `minicap/` & `scrcpy/` - Android screen streaming (MiniCapUtil, ScrcpyServerUtil)

- **`tools/`** - Utilities
  - `EnvCheckTool.java` - Environment validation
  - `AgentManagerTool.java` - Agent lifecycle management
  - `ProcessCommandTool.java` - External process execution
  - `BytesTool.java`, `PortTool.java`, `PHCTool.java`, `SGMTool.java` - Low-level utilities
  - `file/` - Download/upload operations

- **`common/`** - Shared infrastructure
  - `maps/` - Thread-safe maps for device/session state management
  - `enums/` - AndroidKey, ConditionEnum, PlatformType, StepType, ResultDetailStatus
  - `config/` - WebSocket, RestTemplate, resource configuration

- **`mini/`** - Pre-built binaries (minicap, Android SDK tools)
- **`plugins/`** - Third-party APKs and JARs (scrcpy, uiautomator2, go-mitmproxy, WDA)

### Key Patterns

- **Thread Maps**: Extensive use of static maps (`AndroidDeviceManagerMap`, `WebSocketSessionMap`, `HandlerMap`) for cross-component state
- **Step Handlers**: `StepHandlers` routes step types to specific handlers (`IfHandler`, `WhileHandler`, `AssertUtil`, `TextHandler`)
- **Aspect-based validation**: `IteratorAspect` validates iteration annotations via AOP
- **WebSocket communication**: `WebSocketConfig` + `WsEndpointConfigure` for server communication

### Configuration

- Main config: `config/application-sonic-agent.yml`
- Key settings: agent host/port/key, server host/port, WDA bundle ID and project path
- Logging: `logs/sonic-agent.log` with 3-day retention

### Test Structure

Tests located in `src/test/java/`:
- `aspect/` - AOP tests (MockPocoParamTest, TestAop)
- `bridge/ios/` - CompareVersionUtilTest
- `handlers/` - AndroidTouchHandlerTest, TextHandlerTest
- `tools/` - BytesToolTest, PHCToolTest
- `android/` - MockStepDebugTest (device mocking)

# 🚀 OpenCode Support Implementation Roadmap

**Goal:** Extend claude-monitor to support OpenCode, enabling monitoring of ANY LLM provider (OpenAI, Anthropic, Google, etc.)

**Timeline:** 2-3 days  
**Status:** 🟡 In Progress  
**Branch:** `feat/opencode-support`

---

## 📊 Progress Overview

- [x] **Phase 0:** Investigation & Design (COMPLETED)
- [ ] **Phase 1:** Core OpenCode Parser (Day 1)
- [ ] **Phase 2:** Multi-Provider Support (Day 2)
- [ ] **Phase 3:** Testing & Documentation (Day 3)
- [ ] **Phase 4:** PR & Release

**Overall Progress:** 2/4 Phases Complete (50%)

---

## 🔍 Phase 0: Investigation & Design ✅

**Status:** COMPLETED  
**Duration:** 2 hours

### Completed Tasks

- [x] Investigate OpenCode storage structure
- [x] Analyze message file format
- [x] Identify token and cost data location
- [x] Design adapter architecture
- [x] Create implementation plan
- [x] Document findings

### Key Findings

**OpenCode Storage Location:** `~/.local/share/opencode/storage/`

**Message File Structure:**
```json
{
  "modelID": "gpt-5-codex",
  "providerID": "openai",
  "tokens": {
    "input": 18300,
    "output": 568,
    "reasoning": 448,
    "cache": {"write": 0, "read": 0}
  },
  "cost": 0,
  "time": {"created": 1759527626911, "completed": 1759527634819}
}
```

**Conclusion:** ✅ All required data is present and accessible

---

## 🏗️ Phase 1: Core OpenCode Parser (Day 1)

**Status:** 🟡 Pending  
**Estimated Duration:** 6-8 hours  
**Goal:** Basic working OpenCode support

### Task List

#### 1.1 Create OpenCode Reader Module
- [ ] Create `src/claude_monitor/data/opencode_reader.py`
- [ ] Implement `load_opencode_sessions()` function
- [ ] Implement `_parse_opencode_message()` helper
- [ ] Add timestamp parsing from OpenCode format
- [ ] Handle session grouping by session ID
- [ ] Filter messages by `hours_back` parameter

**Estimated Time:** 3 hours

#### 1.2 Add Platform Detection
- [ ] Modify `src/claude_monitor/data/reader.py`
- [ ] Implement `detect_platform()` function
- [ ] Add platform routing logic
- [ ] Extract Claude Code logic to separate function
- [ ] Add data path resolution per platform

**Estimated Time:** 1.5 hours

#### 1.3 Update Settings
- [ ] Modify `src/claude_monitor/core/settings.py`
- [ ] Add `platform` field with options: auto, claude-code, opencode
- [ ] Add `data_path` field for custom paths
- [ ] Update field validators
- [ ] Add platform auto-detection on startup

**Estimated Time:** 1 hour

#### 1.4 Manual Testing
- [ ] Test with real OpenCode data
- [ ] Verify message parsing accuracy
- [ ] Check timestamp conversion
- [ ] Validate session grouping
- [ ] Test platform auto-detection
- [ ] Test explicit `--platform opencode` flag

**Estimated Time:** 1.5 hours

### Phase 1 Deliverable

✅ Basic OpenCode support working with:
- Message parsing from `~/.local/share/opencode/storage/`
- Platform auto-detection
- CLI flag: `claude-monitor --platform opencode`
- Session grouping and time filtering

---

## 💰 Phase 2: Multi-Provider Support (Day 2)

**Status:** 🟡 Pending  
**Estimated Duration:** 6-8 hours  
**Goal:** Full multi-provider pricing and token support

### Task List

#### 2.1 Extend Pricing Calculator
- [ ] Modify `src/claude_monitor/core/pricing.py`
- [ ] Add `PROVIDER_PRICING` dictionary with:
  - [ ] Anthropic (Claude) pricing
  - [ ] OpenAI (GPT-4, GPT-4o, o1) pricing
  - [ ] Google (Gemini) pricing
- [ ] Update `calculate_cost()` to accept `provider` parameter
- [ ] Add reasoning token support for o1 models
- [ ] Implement provider/model lookup logic

**Estimated Time:** 2.5 hours

#### 2.2 Model Name Normalization
- [ ] Update `src/claude_monitor/core/models.py`
- [ ] Add provider prefix to model names (e.g., "openai/gpt-4o")
- [ ] Create model display name mapping
- [ ] Ensure UI shows friendly model names

**Estimated Time:** 1 hour

#### 2.3 Token Type Handling
- [ ] Add reasoning token field to `UsageEntry` model
- [ ] Update OpenCode parser to extract reasoning tokens
- [ ] Handle provider-specific token types
- [ ] Ensure backward compatibility with Claude Code data

**Estimated Time:** 1.5 hours

#### 2.4 Cost Calculation Updates
- [ ] Update OpenCode parser to use provider-aware pricing
- [ ] Calculate costs when message.cost = 0
- [ ] Add fallback pricing for unknown models
- [ ] Log warnings for missing pricing data

**Estimated Time:** 1 hour

#### 2.5 Integration Testing
- [ ] Test with OpenAI messages
- [ ] Test with Anthropic messages
- [ ] Test with Google Gemini messages
- [ ] Verify cost calculations
- [ ] Test mixed-provider sessions

**Estimated Time:** 2 hours

### Phase 2 Deliverable

✅ Multi-provider support with:
- Accurate cost calculation for OpenAI, Anthropic, Google
- Reasoning token tracking (o1 models)
- Provider-aware pricing
- Graceful handling of unknown models

---

## 🧪 Phase 3: Testing & Documentation (Day 3)

**Status:** 🟡 Pending  
**Estimated Duration:** 6-8 hours  
**Goal:** Production-ready with tests and docs

### Task List

#### 3.1 Unit Tests
- [ ] Create `src/tests/test_opencode_reader.py`
- [ ] Test message parsing:
  - [ ] OpenAI messages
  - [ ] Anthropic messages
  - [ ] Google messages
  - [ ] Messages with reasoning tokens
  - [ ] Messages with cache tokens
- [ ] Test time filtering
- [ ] Test session grouping
- [ ] Test platform detection
- [ ] Aim for >80% code coverage

**Estimated Time:** 2.5 hours

#### 3.2 Integration Tests
- [ ] Create test fixtures in `tests/fixtures/opencode/`
- [ ] Mock OpenCode storage structure
- [ ] Test end-to-end flow:
  - [ ] Platform detection → Parser → UsageEntry
  - [ ] Multi-provider cost calculation
  - [ ] UI display with OpenCode data
- [ ] Test CLI with both platforms

**Estimated Time:** 1.5 hours

#### 3.3 Multi-Provider Pricing Tests
- [ ] Create `src/tests/test_multi_provider_pricing.py`
- [ ] Test cost calculations for each provider
- [ ] Test reasoning token costs
- [ ] Test cache token costs
- [ ] Verify pricing accuracy against official rates

**Estimated Time:** 1 hour

#### 3.4 Documentation
- [ ] Update `README.md`:
  - [ ] Add OpenCode installation instructions
  - [ ] Add usage examples with `--platform`
  - [ ] Document supported providers
  - [ ] Add cost calculation notes
- [ ] Update `TROUBLESHOOTING.md`:
  - [ ] Add OpenCode-specific issues
  - [ ] Document storage location differences
  - [ ] Add platform detection debugging
- [ ] Create migration guide for OpenCode users
- [ ] Document provider pricing sources

**Estimated Time:** 2 hours

#### 3.5 Final Testing
- [ ] Run full test suite
- [ ] Test on both platforms (Claude Code + OpenCode)
- [ ] Test with real-world data
- [ ] Verify no breaking changes to Claude Code support
- [ ] Performance testing (1000+ messages)
- [ ] Edge case testing

**Estimated Time:** 1.5 hours

### Phase 3 Deliverable

✅ Production-ready feature with:
- Comprehensive test coverage (>80%)
- Full documentation
- Migration guide
- No breaking changes

---

## 🚢 Phase 4: PR & Release

**Status:** 🟡 Pending  
**Estimated Duration:** 2-4 hours  
**Goal:** Merge to main and release

### Task List

#### 4.1 Code Review & Cleanup
- [ ] Self-review all changes
- [ ] Run linting and formatting
- [ ] Check for code duplication
- [ ] Optimize performance
- [ ] Add missing docstrings
- [ ] Remove debug logging

**Estimated Time:** 1 hour

#### 4.2 Prepare PR
- [ ] Write comprehensive PR description
- [ ] Document breaking changes (if any)
- [ ] Add before/after examples
- [ ] List all new features
- [ ] Include testing evidence
- [ ] Add screenshots/demos

**Estimated Time:** 0.5 hours

#### 4.3 Submit PR
- [ ] Push to `feat/opencode-support` branch
- [ ] Create PR to main repository
- [ ] Request review from maintainers
- [ ] Address review comments
- [ ] Update based on feedback

**Estimated Time:** 1 hour

#### 4.4 Post-Merge Tasks
- [ ] Update version number
- [ ] Create release notes
- [ ] Publish to PyPI
- [ ] Announce on Discord/GitHub Discussions
- [ ] Write blog post (optional)
- [ ] Create demo video (optional)

**Estimated Time:** 1.5 hours

### Phase 4 Deliverable

✅ Merged PR with:
- Clean, reviewed code
- Passing CI/CD
- Published release
- Community announcement

---

## 📋 Technical Implementation Details

### File Changes Summary

| File | Type | Lines | Status |
|------|------|-------|--------|
| `data/opencode_reader.py` | NEW | ~150 | ⬜ Pending |
| `data/reader.py` | MODIFY | ~50 | ⬜ Pending |
| `core/pricing.py` | MODIFY | ~100 | ⬜ Pending |
| `core/settings.py` | MODIFY | ~30 | ⬜ Pending |
| `core/models.py` | MODIFY | ~20 | ⬜ Pending |
| `tests/test_opencode_reader.py` | NEW | ~200 | ⬜ Pending |
| `tests/fixtures/opencode/*` | NEW | ~50 | ⬜ Pending |
| `README.md` | MODIFY | ~50 | ⬜ Pending |
| `TROUBLESHOOTING.md` | MODIFY | ~30 | ⬜ Pending |

**Total New Code:** ~580 lines  
**Total Modified:** ~180 lines  
**Impact:** <3% of codebase (~20,000 lines)

---

## 🎯 Success Criteria

### Functional Requirements
- [ ] Auto-detect OpenCode vs Claude Code installation
- [ ] Parse OpenCode message files correctly
- [ ] Support providers: Anthropic, OpenAI, Google
- [ ] Calculate costs accurately per provider
- [ ] Handle reasoning tokens (o1 models)
- [ ] Maintain backward compatibility with Claude Code
- [ ] Filter by time range (hours_back)
- [ ] Display correct model names in UI

### Non-Functional Requirements
- [ ] No breaking changes to existing functionality
- [ ] Performance: Parse 1000+ messages in <2 seconds
- [ ] Test coverage: >80% for new code
- [ ] Documentation: Complete usage guide
- [ ] Error handling: Graceful degradation

---

## 🚧 Known Edge Cases

### To Handle
1. **Missing Cost Data:** Calculate from tokens + pricing when `cost = 0`
2. **Unknown Models:** Use fallback pricing, log warning
3. **Reasoning Tokens:** Only present in o1 models, handle gracefully
4. **Empty Sessions:** Skip sessions with no messages
5. **Timestamp Format:** OpenCode uses milliseconds since epoch

### Deferred to Phase 2
1. Real-time monitoring (file watching)
2. Project-specific filtering
3. Custom provider pricing from config file
4. Export/import functionality

---

## 📊 Metrics & KPIs

### Development Metrics
- **Total Estimated Effort:** 16-22 hours
- **Target Completion:** 3 days
- **Code Quality:** >80% test coverage
- **Performance:** <2s for 1000 messages

### Impact Metrics
- **User Base:** Expand to OpenCode users (26.4k GitHub stars)
- **Provider Support:** 3+ LLM providers
- **Breaking Changes:** 0 (backward compatible)

---

## 🔗 Related Links

- **OpenCode Repository:** https://github.com/sst/opencode
- **OpenCode Docs:** https://opencode.ai/docs
- **Claude Monitor:** https://github.com/Maciek-roboblog/Claude-Code-Usage-Monitor
- **Implementation Article:** [Gist with architecture details]

---

## 🎁 Benefits

### For Users
1. ✅ Universal monitoring across ALL LLMs
2. ✅ Cost transparency for multiple providers
3. ✅ Platform freedom (Claude Code OR OpenCode)
4. ✅ Future-proof for new providers

### For Project
1. ✅ Larger user base (OpenCode community)
2. ✅ No longer Claude-specific
3. ✅ Strategic positioning as universal AI coding monitor
4. ✅ Community contribution opportunity

---

## ❓ Questions & Decisions

### Resolved
- ✅ **Pricing Source:** Use static pricing tables (simpler, faster)
- ✅ **Session Grouping:** Group by session ID, filter globally
- ✅ **Platform Detection:** Auto-detect with manual override

### Open Questions
- ⏳ **Live Pricing API:** Consider for future enhancement?
- ⏳ **Project Filtering:** Add `--project` flag in Phase 2?
- ⏳ **Real-time Updates:** File watching for live monitoring?

---

## 🏁 Current Sprint Status

**Active Phase:** Phase 0 → Phase 1  
**Next Milestone:** Create OpenCode reader module  
**Blockers:** None  
**Risk Level:** Low ✅

---

**Last Updated:** 2025-10-03  
**Document Owner:** @av1155  
**Status:** 🟢 Active Development

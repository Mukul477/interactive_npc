# Stateful, Adaptive NPCs: Research Review and Design Guidance

## 1. Purpose and scope

This review supports the proposed NPC system: speech input is transcribed, player intent and affect are inferred, an NPC cognitive/persona engine updates persistent memory and relationship state, and a local language model produces a response that remains consistent with the NPC's personality and backstory.

The central design recommendation is **not** to treat the language model as the NPC itself. Instead, use a hybrid architecture:

1. Deterministic state and rules enforce identity, world facts, safety, and relationship updates.
2. Retrieval and summarization select the memories relevant to the current interaction.
3. The language model plans and verbalizes a response within explicit constraints.
4. A validation layer checks persona, factual grounding, emotional plausibility, and output format before the response reaches the game.

This separation directly addresses persona drift, unreliable long-term memory, and inconsistent emotional behavior.

## 2. Most relevant research

### 2.1 Generative Agents: Interactive Simulacra of Human Behavior

**Park et al., UIST 2023.** [Paper and project material](https://arxiv.org/abs/2304.03442)

Generative Agents introduced a practical architecture for believable social agents in a simulated town. Each agent maintains an experience stream, retrieves memories using relevance/recency/importance, and periodically reflects to form higher-level observations. Those memories and reflections guide planning and action.

**Why it matters for this project**

- Provides the strongest direct precedent for persistent, socially believable NPC behavior.
- Suggests separating raw events from distilled semantic memories.
- Gives a useful retrieval score:
  `retrieval_score = relevance + recency + importance`.
- Shows that planning and reflection should be separate from immediate dialogue generation.

**Adaptation**

Store every interaction as an immutable event, then asynchronously create or update semantic memories such as “the player kept a promise.” Retrieve only the top-ranked memories relevant to the current topic, relationship, and emotional context.

### 2.2 Reflexion: Language Agents with Verbal Reinforcement Learning

**Shinn et al., 2023.** [Paper](https://arxiv.org/abs/2303.11366)

Reflexion uses a verbal reflection buffer: after an outcome, an agent records what went well or poorly and uses that text in later decisions. It is not a complete NPC memory system, but it demonstrates how an agent can improve behavior without changing model weights.

**Why it matters**

- Supports episodic “lessons learned” without fine-tuning.
- Provides a mechanism for an NPC to remember failed attempts, broken promises, or successful negotiations.
- Encourages storing reflections separately from objective facts.

**Caution**

Reflections are model-generated interpretations and may be wrong. They should be marked as low-confidence hypotheses and confirmed or corrected by later evidence rather than treated as canonical world facts.

### 2.3 Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks

**Lewis et al., NeurIPS 2020.** [Paper](https://arxiv.org/abs/2005.11401)

RAG combines parametric language-model knowledge with a retrievable external memory. The key principle is that changing or growing knowledge should be represented in the retrieval store rather than hidden inside model weights.

**Why it matters**

- Fits PostgreSQL-backed NPC memories and world knowledge.
- Allows memory updates without retraining a local model.
- Makes retrieved evidence inspectable and easier to debug.

**Adaptation**

Use PostgreSQL for canonical records and `pgvector` (or a separate vector index if required) for semantic retrieval. Every retrieved memory should retain its source interaction, timestamp, confidence, and visibility scope.

### 2.4 Persona-Chat: Designing Dialogue Agents with Personas

**Zhang et al., ACL 2018.** [Paper](https://arxiv.org/abs/1801.07243) | [ACL Anthology](https://aclanthology.org/P18-2023/)

Persona-Chat established a benchmark for conditioning dialogue on explicit persona statements. It demonstrates both the value and difficulty of grounding responses in stable character attributes.

**Why it matters**

- Supports representing backstory and personality as explicit, testable persona facts.
- Provides a starting point for persona-grounded evaluation.
- Highlights that persona consistency is distinct from generic response fluency.

**Adaptation**

Represent persona as versioned facts with priority and provenance:

- immutable identity and backstory;
- stable traits and values;
- current goals and obligations;
- temporary mood;
- relationship-specific beliefs.

The prompt should label these categories separately so a temporary mood cannot silently rewrite the backstory.

### 2.5 EmpatheticDialogues: A Large-Scale Dataset for Empathetic Response Generation

**Rashkin et al., ACL 2019.** [Paper](https://arxiv.org/abs/1811.00207) | [ACL Anthology](https://aclanthology.org/P19-1534/)

EmpatheticDialogues studies responses to situations associated with emotions and provides emotional situation labels. It is useful for understanding emotion recognition and emotionally appropriate response generation, although it is not a full game-agent architecture.

**Why it matters**

- Supports the proposed tone/emotion analysis stage.
- Encourages distinguishing the player's inferred emotion from the NPC's internal mood.
- Provides ideas for evaluating emotional appropriateness separately from factual correctness.

**Adaptation**

Use emotion inference as one input to appraisal, not as a direct command. For example, detecting player anger should update the event interpretation and candidate relationship effects; it should not automatically force the NPC to apologize or become angry.

### 2.6 MemoryBank: Enhancing Large Language Models with Long-Term Memory

**Zhong et al., 2023.** [Paper](https://arxiv.org/abs/2305.10250)

MemoryBank explores long-term memory for conversational agents, including memory formation, retrieval, and update over extended interaction. It is an early but useful reference for storing user-specific experiences and evolving conversational context.

**Why it matters**

- Directly relevant to persistent player-specific memories.
- Motivates memory consolidation and forgetting rather than storing an unbounded transcript.
- Suggests that memory should influence both content and the relationship model.

**Caution**

This is an arXiv preprint. Use its architectural ideas, but validate retrieval and memory-update policies experimentally in the target game.

### 2.7 MemGPT: Towards LLMs as Operating Systems

**Packer et al., 2023.** [Paper](https://arxiv.org/abs/2310.08560)

MemGPT treats the language model as managing a hierarchy of memory: a limited working context plus larger archival storage. The model decides what to page in and out.

**Why it matters**

- Offers a useful mental model for local-model context limits.
- Supports separating immediate conversation context from long-term archives.
- Suggests explicit memory-management operations instead of blindly appending the entire history to every prompt.

**Adaptation**

Do not give the model unrestricted write access to canonical relationship values. Let the model propose memory operations in a structured schema; a deterministic cognitive engine validates and applies them.

### 2.8 Character-LLM: A Trainable Agent for Role-Playing

**Shao et al., 2023.** [Paper](https://arxiv.org/abs/2310.10158)

Character-LLM studies language-model agents that role-play characters based on biographies and experience. It is relevant to personality-conditioned generation and evaluating whether generated conversations reflect a character's profile.

**Why it matters**

- Provides a direct connection between character biography and role-play behavior.
- Reinforces the importance of evaluating character consistency, not just linguistic quality.
- Suggests using character-specific experiences as conditioning information.

### 2.9 Voyager: An Open-Ended Embodied Agent with Large Language Models

**Wang et al., 2023.** [Paper](https://arxiv.org/abs/2305.16291)

Voyager demonstrates an LLM-driven embodied agent with an automatic curriculum, skill library, and iterative feedback. It targets Minecraft rather than social NPC dialogue, but its modular skill representation is valuable for NPC action planning.

**Why it matters**

- Dialogue should be connected to executable actions and goals, not generated in isolation.
- A reusable skill/action library can constrain model outputs and make behavior reliable.
- Feedback from the game world should update the agent's state.

**Adaptation**

Represent candidate NPC actions as typed game commands such as `offer_quest`, `refuse_request`, `move_to_location`, or `change_relationship`. The model selects or fills these actions; the game validates and executes them.

### 2.10 Personalized Non-Player Characters: A Framework for Character-Consistent Dialogue Generation

**2025.** [MDPI article](https://www.mdpi.com/2673-2688/6/5/93)

This recent NPC-focused work combines persona information, interaction history, and structured relationship information for character-consistent dialogue. It is especially relevant because it addresses NPC personalization rather than only general-purpose chat.

**Why it matters**

- Supports modeling relationships as structured data rather than only prose memories.
- Provides a recent comparison point for a project evaluation.
- Motivates knowledge-graph-like links between NPCs, players, events, and beliefs.

**Caution**

Check the paper's exact datasets, baselines, and evaluation protocol before claiming superiority. Use it as related work and an implementation prompt, not as proof that a particular architecture is universally best.

### 2.11 LLM-Driven NPCs: Cross-Platform Dialogue System for Games and Social Platforms

**2025 preprint.** [Paper](https://arxiv.org/abs/2504.13928)

This work explores NPC dialogue across a game and a social platform, including synchronized interaction history and affect/favorability signals.

**Why it matters**

- Relevant to persistent identity across multiple interfaces.
- Demonstrates the value of an external state layer shared by all clients.
- Provides a modern reference for evaluating cross-session NPC continuity.

**Caution**

It is a recent preprint and should be independently reproduced before adopting its claims or metrics.

## 3. Supporting engineering and evaluation material

### PostgreSQL and vector retrieval

- [PostgreSQL documentation](https://www.postgresql.org/docs/current/)
- [pgvector](https://github.com/pgvector/pgvector)

Use relational tables for authoritative state, indexes for time and player/NPC scope, and vector embeddings only as a retrieval aid. A vector similarity match must never override a newer explicit fact or an invariant persona rule.

### Speech and emotion pipeline references

- [Whisper: Robust Speech Recognition via Large-Scale Weak Supervision](https://arxiv.org/abs/2212.04356)
- [Wav2Vec 2.0](https://arxiv.org/abs/2006.1143)
- [Speech Emotion Recognition: A Survey](https://arxiv.org/abs/2205.07317)

Speech-to-text and emotion recognition should expose confidence scores. Low-confidence emotion predictions should have a smaller effect on mood or relationships than explicit player actions and verified game events.

### Agent evaluation

- [SotA: A Survey of LLM-Based Agent Evaluation](https://arxiv.org/abs/2404.02875)
- [AgentBench: Evaluating LLMs as Agents](https://arxiv.org/abs/2308.03688)
- [G-Eval: NLG Evaluation using GPT-4 with Better Human Alignment](https://arxiv.org/abs/2303.16634)

For this project, combine automated checks with human playtests. Generic “conversation quality” scores are insufficient because an NPC can sound fluent while violating its backstory or relationship history.

## 4. Proposed architecture derived from the literature

```text
Microphone
   |
Speech-to-text + confidence
   |
Intent / tone / emotion / entities
   |
Interaction event (immutable)
   |
NPC cognitive engine
   |-- appraisal: goal impact, norm violation, threat, benefit
   |-- relationship update: trust, affinity, respect, fear, debt
   |-- mood update: short-term affect with decay
   |-- memory write proposal: event, fact, reflection, confidence
   |-- goal and action selection
   |
Memory retrieval + context assembly
   |-- recent turns
   |-- relevant episodic memories
   |-- consolidated semantic memories
   |-- persona and backstory constraints
   |-- current relationship and mood
   |
Local language model
   |
Structured response proposal
   |-- speech text
   |-- emotion display
   |-- game action(s)
   |-- memory updates
   |
Validator / policy / game-state check
   |
Text-to-speech + animation + game action
```

### Recommended state boundaries

| State | Examples | Update policy |
| --- | --- | --- |
| Immutable identity | name, origin, core backstory | Never changed by dialogue |
| Stable persona | values, traits, speaking style | Changed only by authored updates |
| Current goals | protect a location, find an item | Updated by quests and world events |
| Mood | calm, afraid, irritated | Decays toward baseline over time |
| Relationship | trust, affinity, respect, fear, obligation | Updated by appraised events |
| Episodic memory | “player warned me about the ambush” | Append, score, retrieve, consolidate |
| Semantic belief | “player is reliable” | Derived from multiple episodes; confidence tracked |
| Procedural knowledge | available actions and social policies | Authored or learned under validation |

## 5. Suggested PostgreSQL model

The exact schema can evolve, but the research suggests keeping facts, events, memories, and derived state distinct:

```text
npcs
players
npc_persona_facts
npc_relationships
interaction_events
npc_memories
npc_reflections
npc_goals
npc_state_snapshots
```

Recommended fields for `npc_memories` include:

- `memory_type`: episodic, semantic, reflection, procedural;
- `content` and optional embedding;
- `source_event_id`;
- `importance`, `confidence`, `created_at`, `last_retrieved_at`;
- `valid_from`, `valid_until` or a supersession link;
- `player_id` and visibility scope.

Store state transitions as auditable events or snapshots. This makes it possible to reproduce a conversation, investigate an unexpected response, and compare different update policies.

## 6. Research gaps that create innovation opportunities

1. **Verified persona transitions:** allow relationship adaptation while preventing model-generated text from rewriting immutable identity or core values.
2. **Multi-timescale emotion:** combine fast mood changes, slower relationship changes, and very slow personality development.
3. **Memory provenance and contradiction handling:** track where each belief came from and resolve conflicts using recency, confidence, direct evidence, and authored canon.
4. **Action-grounded dialogue:** require the model to emit structured, executable actions rather than free-form text only.
5. **Local-model optimization:** compare prompt compression, retrieval quality, quantization, and speculative generation under a fixed latency budget.
6. **Player-specific evaluation:** measure continuity after days of interaction, not just turn-level response quality.
7. **Uncertainty-aware affect:** prevent a low-confidence tone classifier from causing a large relationship change.
8. **Social memory graphs:** represent promises, debts, allies, witnesses, and shared events as linked entities rather than isolated text chunks.

## 7. Evaluation plan

Track both system metrics and human-perceived believability:

| Dimension | Example metric |
| --- | --- |
| Persona consistency | blinded raters detect fewer contradictions; automated fact checks pass |
| Memory accuracy | precision/recall of recalled facts; no fabricated memories |
| Relationship continuity | state changes match labeled interaction outcomes |
| Emotional appropriateness | response fits player affect and NPC appraisal |
| Behavioral adaptation | NPC choices change after relevant player actions |
| Action validity | percentage of generated actions accepted by the game |
| Latency | p50/p95 speech-to-response time |
| Safety and robustness | invalid schema rate, prompt-injection resistance, refusal correctness |
| Player experience | trust, perceived agency, immersion, and repetitiveness in playtests |

Build a replayable test set containing neutral, kind, hostile, deceptive, ambiguous, and contradictory interactions. Test both a fresh NPC and an NPC with a long history. Report ablations for memory retrieval, reflection, relationship state, and validation so improvements can be attributed to specific components.

## 8. Recommended implementation order

1. Implement deterministic interaction events, persona facts, relationship state, and state snapshots.
2. Add a text-only dialogue loop with structured model output and validation.
3. Add episodic memory retrieval and source-linked semantic consolidation.
4. Add intent, tone, and emotion inference with confidence-aware updates.
5. Add goals, executable game actions, and action-result feedback.
6. Add speech-to-text/text-to-speech and latency instrumentation.
7. Run ablation studies and longitudinal playtests before adding model fine-tuning.

## 9. Reference list

1. Park, J. S. et al. “Generative Agents: Interactive Simulacra of Human Behavior.” UIST 2023. https://arxiv.org/abs/2304.03442
2. Shinn, N. et al. “Reflexion: Language Agents with Verbal Reinforcement Learning.” 2023. https://arxiv.org/abs/2303.11366
3. Lewis, P. et al. “Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks.” NeurIPS 2020. https://arxiv.org/abs/2005.11401
4. Zhang, S. et al. “Personalizing Dialogue Agents: I have a dog, do you have pets too?” ACL 2018. https://aclanthology.org/P18-2023/
5. Rashkin, H. et al. “Towards Empathetic Open-domain Conversation Models: A New Benchmark and Dataset.” ACL 2019. https://aclanthology.org/P19-1534/
6. Zhong, W. et al. “MemoryBank: Enhancing Large Language Models with Long-Term Memory.” 2023. https://arxiv.org/abs/2305.10250
7. Packer, C. et al. “MemGPT: Towards LLMs as Operating Systems.” 2023. https://arxiv.org/abs/2310.08560
8. Shao, Y. et al. “Character-LLM: A Trainable Agent for Role-Playing.” 2023. https://arxiv.org/abs/2310.10158
9. Wang, G. et al. “Voyager: An Open-Ended Embodied Agent with Large Language Models.” 2023. https://arxiv.org/abs/2305.16291
10. “Personalized Non-Player Characters: A Framework for Character-Consistent Dialogue Generation.” 2025. https://www.mdpi.com/2673-2688/6/5/93
11. “LLM-Driven NPCs: Cross-Platform Dialogue System for Games and Social Platforms.” 2025. https://arxiv.org/abs/2504.13928
12. Radford, A. et al. “Robust Speech Recognition via Large-Scale Weak Supervision.” 2022. https://arxiv.org/abs/2212.04356
13. Liu, X. et al. “AgentBench: Evaluating LLMs as Agents.” 2023. https://arxiv.org/abs/2308.03688

Links were checked against the linked publisher, ACL Anthology, project, or arXiv landing pages on 2026-10-05. Preprints and industry material are labeled so their evidence is not confused with peer-reviewed findings.
